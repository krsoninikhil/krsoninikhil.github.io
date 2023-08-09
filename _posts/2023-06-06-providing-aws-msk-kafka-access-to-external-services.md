Setting up private Kafka on AWS is quite straight from the console. Which works well if all your services that depends on Kafka are on same VPC. But if you want to push events from a third party hosted application like [Rudderstack events to Kafka as destination](https://www.rudderstack.com/docs/destinations/streaming-destinations/kafka/), which requires you to provide SASL auth details, it becomes a bit complicated. This article aims to document the process. 

If don’t already have a MSK cluster created, you can use following Terraform script using [this module](https://github.com/krsoninikhil/terraform-modules/tree/main/msk) to create one

```bash
# main.tf
module "external_msk" {
  source = "github.com/krsoninikhil/terraform-modules//msk"

  name          = "external-cluster"
  make_public   = var.msk_make_public
  vpc_id        = var.vpc_id
  no_of_nodes   = 2
  instance_type = "kafka.t3.small"
  scram_users   = [{ username = "admin", password = "password" }]
}

# terraform.tfvars
msk_make_public = false
```

[SASL](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) is authentication framework and AWS MSK supports multiple such mechanism — [SCRAM](https://en.wikipedia.org/wiki/Salted_Challenge_Response_Authentication_Mechanism) (username password based authentication) using AWS Secret Manager for managing credentials, [IAM Authentication](https://docs.aws.amazon.com/msk/latest/developerguide/iam-access-control.html) (using IAM users and policy instead of Kafka ACLs) and [TLS based auth](https://docs.aws.amazon.com/msk/latest/developerguide/msk-authentication.html). You can allow public access to your MSK cluster by following steps from [this doc](https://docs.aws.amazon.com/msk/latest/developerguide/public-access.html):

- Turn off the plaintext communication, public cluster needs to have TLS enabled
- Turn on the SCRAM and IAM based auth and turn off the unauthenticated access
- Create secrets with username and password follwing [this](https://docs.aws.amazon.com/msk/latest/developerguide/msk-password.html) doc and attach to your cluster
- Ensure relevant ports are open for connections - 9198 for IAM auth and 9196 for SCRAM
- Turn on the public access

If you used above Terraform script, just change the `msk_make_public` variable to `true` and apply the config again. Note that MSK does not allow public access during creation, so this needs to modified by applying it again after the variable change.

```text
💡 Gotcha: While creating credentials in Secret Manager for your cluster, you must use “Plaintext” and not the key/value editor, even though it generates the same text. I’m still not why key/value method does not work properly, if you do, please let me know.
```

That’s where the doc ends, so it should be done here? Let’s try to connect the brokers via [kcat](https://github.com/edenhill/kcat)

```bash
$ kcat -L -b "broker1:port,broker2:port" -t "test_topic" -p 1 -X security.protocol=SASL_SSL -X sasl.mechanism="SCRAM-SHA-512" -X sasl.username=<user-from-secret> -X sasl.password=<password-from-secret>

Delivery failed for message: Broker: Topic authorization failed
```

Failing authorization because the SCRAM user we just added will not have appropriate ACL or permissions setup to access or create the topic. For MSK with public access, the `allow.everyone.if.no.acl.found` property must be set to `false`, so it must need ACLs. [This doc](https://catalog.workshops.aws/msk-labs/en-US/securityencryption/saslscram/authorization) explains how to create ACLs using kafka client scripts that comes with kafka installation. If kafka is not installed, you can use a docker container for kafka.

```bash
$ docker run --rm confluentinc/cp-kafka:latest /bin/sh \
  /bin/kafka-acls --bootstrap-server "broker:port"  --command-config config.properties --add --allow-principal User:<kms-user> --operation Read --topic test_topic

Adding ACLs for resource `ResourcePattern(resourceType=TOPIC, name=superbio_ui_events_staging, patternType=LITERAL)`:
    (principal=User:gossupkafkaprod, host=*, operation=READ, permissionType=ALLOW)

Error while executing ACL command: org.apache.kafka.common.errors.TimeoutException: Timed out waiting for a node assignment. Call: createAcls
... 
```

If you think about this, this was not supposed to work anyway, it’s trying to give access to the self. The problem is we made the MSK public before creating the ACL, this could have worked if executed from same VPC as MSK before making it public. Now, we can’t use SCRAM auth for creating ACLs. It can done however by using IAM auth since MSK ingores the ACLs if authenticating through IAM. So we can use IAM auth to create topic publish and read ACLs and then use SCRAM with external services like Rudderstack destination.

So, create an IAM user with appropriate policy as mentioned in [this AWS doc](https://docs.aws.amazon.com/msk/latest/developerguide/create-client-iam-role.html) and run [following Go code](https://gist.github.com/krsoninikhil/3ed02b79d7614dac450dcffc44fa979e) with the above created IAM user creds to create a topic with read write ACL on it for the given SCRAM user. Since the TLS in transit is enabled in public clusters, Go will need the CA certificate to trust, it’s not required in Java world as it already has a trust store. You can get the certificate using this command, you can use the last certificate in the listed chain.

```bash
$ openssl s_client -showcerts -connect broker1:port
```

<script src="https://gist.github.com/krsoninikhil/3ed02b79d7614dac450dcffc44fa979e.js"></script>

Topic creation and permissions can be verified using `kcat`

```bash
$ kcat -L -b "broker1:port,broker2:port" -t "test_topic" -p 1 -X security.protocol=SASL_SSL -X sasl.mechanism="SCRAM-SHA-512" -X sasl.username=<user-from-secret> -X sasl.password=<password-from-secret>

Metadata for all topics ...
2 brokers:
...
```

### Summary

- Create MSK cluster and then make it publicly accessible. You can use above Terraform config to create if not already created and then update it to allow public access.
- Create IAM user which appropriate policy to allow creating topic and publish
- Use above Go code to create topic and ACL for the provided SCRAM user
- Now, you can use this user with external services

### Discussion

- [Reddit post](https://www.reddit.com/r/devops/comments/142qfc0/providing_aws_msk_kafka_access_to_external/?utm_source=share&utm_medium=web2x&context=3)

--
[Nikhil](https://twitter.com/krsoninikhil)