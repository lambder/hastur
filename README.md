![hastur](https://cloud.githubusercontent.com/assets/674812/10144792/fadcbd28-660e-11e5-8569-894e874cff14.png)

hastur is a tool for launching systemd-nspawn containers without need of manual
configuration.

It will setup networking, base root FS and even overlay FS for containers for
you.

Primary usecase for hastur is supporting testcases for distributed systems and
running local set of trusted containers.

![gif](https://cloud.githubusercontent.com/assets/674812/10140037/37f12ea2-65f5-11e5-90c7-eb18e6a9b37b.gif)

# Motivation

systemd-nspawn is useful tool, which can create and run lightweight
containers without any additional software, because it's out-of-the-box
functionality available in systemd.

But it requires a some configuration to run working container, like
managing network, downloading and extracting packages.

hastur offers all configuration hidden in it's implementation, that will
make possible to run fully-configured systemd-nspawn containers in seconds.

# Installation

# archlinux
hastur is available for Arch Linux (for now, only) through AUR:

https://aur4.archlinux.org/packages/hastur/

# go get
hastur is also go-gettable:

```
go get github.com/seletskiy/hastur
```

# Usage

## Testing water

Most simple usage is just testing hastur out-of-the-box, telling to create
ephemeral container with basic set of packages:

```
sudo hastur -S
```

After invoking that command you will end up ~~in the Ctulhu's void~~ in the
bash shell.

That test container will be deleted after you exit shell.

## Creating non-ephemeral containers

Passing flag `-k` will tell hastur to keep container after exit.

```
sudo hastur -kS
```

If you do not like that fantastical autogenerated names, you can pass flag `-n`:

```
sudo hastur -Sn my-cool-name
```

Note, that ephemeral containers are only that ones, that have autogenerated
name and is not started with flag `-k`.

## Networking

hastur will take care of your networking by creating bridge and setting up
share network. By default, hastur will generate IP addresses automatically,
and you can see container adress either in the starting message or via running
query command:

```
sudo hastur -Q
```

You can specify your own IP address by passing `-a` flag, like that one:

```
sudo hastur -S -a 10.0.0.2/8
```

## But what about software?

hastur uses package-based container configuration and will happily populate
your container with packages that you like. Flag `-p` intented for that one.

```
sudo hastur -S -p nginx
```

It will create container with pre-installed package `nginx`. Actually, hastur
uses overlays for keeping base dirs separately of container data. That base
dirs, or, if you like, images, are just prepared root filesystems, which has
pre-installed packages. You can query cached base dirs by running hastur
with `-Qi` flag.

```
sudo hastur -Qi
```

Actually, from hastur standpoint, container is just data dir, which will
be overlayed on top of root filesystem and then given network capability, so
it will not remembed what IP address container have or what set of packages it
has installed if you forget to specify `-p` flag.

Like:

```
sudo hastur -Sn test -p git -- /bin/git --version
```

Will output:

```
git version 2.5.3
```

But running next time without `-p git` will tell that there is no such command:

```
sudo hastur -Sn test -- /bin/git --version
```

Outputs:

```
No such file or directory
```

However, that `test` container will have separate FS and all files will persist
across that two runs.

# Even more

hastur can operate over several root directories and keep container instances
one from each other. Flag `-r` is considered in that case.

```
sudo hastur -r solar-system -Sn earth
sudo hastur -r solar-system -Sn moon
sudo hastur -r alpha-centauri -Sn a
sudo hastur -r alpha-centauri -Sn b
sudo hastur -r alpha-centauri -Sn c
```

Output will be different for that two commands:

```
sudo hastur -r solar-system -Q
sudo hastur -r alpha-centauri -Q
```

First one will list only `earth` and `moon` containers, but second one will
list `a`, `b` and `c` containers.
