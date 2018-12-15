I needed to install Mongodb for a project I was working on, as usual I
did:

```bash
$ sudo pacman -S mongodb
```

Now starting the `mongodb.service` with `systemctl` was failing
without any specific error logs:

```bash
$ journalctl -u mongodb.service -b
-- Logs begin at Thu 2017-06-01 13:13:02 IST, end at Sun 2018-01-07 12:14:08 IST. --
Jan 07 12:05:56 TheBlackPearl systemd[1]: Started High-performance, schema-free document-oriented da
Jan 07 12:05:56 TheBlackPearl systemd[1]: mongodb.service: Main process exited, code=exited, status=
Jan 07 12:05:56 TheBlackPearl systemd[1]: mongodb.service: Unit entered failed state.
Jan 07 12:05:56 TheBlackPearl systemd[1]: mongodb.service: Failed with result 'exit-code'.
Jan 07 12:09:46 TheBlackPearl systemd[1]: Started High-performance, schema-free document-oriented da
Jan 07 12:09:46 TheBlackPearl systemd[1]: mongodb.service: Main process exited, code=exited, status=
Jan 07 12:09:46 TheBlackPearl systemd[1]: mongodb.service: Unit entered failed state.
Jan 07 12:09:46 TheBlackPearl systemd[1]: mongodb.service: Failed with result 'exit-code'.
lines 1-9/9 (END)
```

And running `mongo` and `mongodb` was throwing following error:

```bash
mongo: error while loading shared libraries: libboost_program_options.so.1.65.1: cannot open shared object file: No such file or directory
```

I seached this a bit and then installed `boost` package.

```bash
$ sudo pacman -S boost
```

And now running `mongo` was throwing different error:

```bash
mongo: error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory
```

I tries installing `mongo-tools` package which also installed
`openssl-1.0` along with it. And running `mongo` was still throwing
same error. So I installed `openssl` which got me
`openssl-1.1`. That's where almost everything broke. Running `sudo`:

```bash
$ sudo pacman
sudo: error in /etc/sudo.conf, line 0 while loading plugin "sudoers_policy"
sudo: unable to load /usr/lib/sudo/sudoers.so: libssl.so.1.0.0: cannot open shared object file: No such file or directory
sudo: fatal error, unable to load plugins
```

Since `sudo` was not an option now, I switched to `root` by `su` only
to realize that even `pacman` is broken:

```bash
$ pacman -Ss openssl
pacman: error while loading shared libraries: libcrypto.so.1.0.0: cannot open shared object file: No such file or directory
```

Now, this was scary. How do fix broken packages without my package
manager?

Searching for this I found [grism's
answer](https://bbs.archlinux.org/viewtopic.php?pid=1706378#p1706378)
to similar problem. Turns out current version of `sudo` and `pacman`
depends on `openssl-1.0` and I upgraded it to `1.1`. To downgrade it,
I didn't need to download the `1.0` version package from web as
installing `mongodb-tools` also installed `openssl-1.0`, `pkg.tar.xz`
of which can be found at `/var/cache/pacman/pkg`. As per the above
link, I extracted it in `/tmp` and made a symlink for `libssl-1.0.0`
and `libcrypto-1.0.0` at `/var/lib/`:

```bash
# cd /tmp && tar xf /var/cache/pacman/pkg/openssl-1.0-1.0.2.n-1-x86_64.pkg.tar.xz
# ln -s /tmp/usr/lib/libcrypto.so.1.0.0 /usr/lib/libcrypto.so.1.0.0
# ln -s /tmp/usr/lib/libssl.so.1.0.0 /usr/lib/libssl.so.1.0.0
```

That is some terrible advice here, as symlinking one version to
another can break mulitple things, but other solutions were suggesting
to reinstall OS. Now, Arch Linux comes with absolute minimum built in
packages which means reinstalling it would require reinstalling all
the packages which I had installed over the span of 2-3 years since
I'm using Arch. So obiously, I took the above advice and well that
fixed my `pacman`. So first thing first, I need my `sudo` back, so I
installed `openssl-1.0` from downloaded `.pkg.tar.xz`:

```bash
# pacman -U /var/cache/pacman/pkg/openssl-1.0-1.0.2.n-1-x86_64.pkg.tar.xz
```

Exit `root` and try:

```bash
$ sudo ls
[sudo] password for nks:
```

That's better. Back to `MongoDB` problem.

Lesson learnt -- when they say upgrading a single package in Arch
Linux in not recommended, they mean it. Infact they mean, never ever
do that, instead do a full system upgrade like

```bash
$ sudo pacman -Syu
```

--
[@krsoninikhil](https://twitter.com/krsoninikhil)
