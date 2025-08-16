# podman-quadlet-docs

WIP (work-in-progress)....

Podman documentation of how to write quadlets.
Description of the most useful quadlet and systemd directives.

# Introduction

To create systemd services that run podman, write quadlet files.

For a details about all quadlet directives, see the man page
[podman-systemd.unit](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)


## Comparison to Compose files

Quadlet files allow for
better systemd integration than compose files.
The quadlet file syntax allows systemd directives to be used.
Compose files do not support _socket activation_.

To convert a _compose_ file to quadlets, use the software tool [podlet](https://github.com/containers/podlet).
One _compose_ file is typically converted into multiple quadlet files.

# Converting quadlet files into systemd system service unit files

`podman-system-generator` and `podman-user-generator` are systemd generators that convert
quadlet files in to systemd services unit files.

For details about systemd generators, see the man page [systemd.generator](https://www.freedesktop.org/software/systemd/man/latest/systemd.generator.html).

## Converting quadlet files into systemd system unit files

The program `podman-system-generator` converts quadlet files in to systemd system services.

To create _systemd system services_, do following steps

1. Save your quadlet files in the directory `/etc/containers/systemd`
2. Reload the systemd system manager
   ```
   sudo systemctl daemon-reload
   ```
   The systemd system manager now executes programs in the directory
   `/usr/lib/systemd/system-generators/`. The program
   `/usr/lib/systemd/system-generators/podman-system-generator` converts
   quadlet files into systemd system service units. The generated files are written to
   the directory `/run/systemd/generator`

Note, files under `/run/` are deleted when the system is rebooted.

## Converting quadlet files into systemd user service unit files

The program `podman-user-generator` converts quadlet files in to systemd system services.

To create _systemd user services_, do following steps

1. Create directory
   ```
   mkdir $HOME/.config/container/systemd
   ```
2. Save your quadlet files in the directory `$HOME/.config/container/systemd`
3. Reload the systemd user manager
   ```
   systemctl --user daemon-reload
   ```
   The systemd system manager now executes programs in the directory
   `/usr/lib/systemd/user-generators/`. The program
   `/usr/lib/systemd/user-generators/podman-user-generator` converts
   quadlet files into systemd user service units. The generated files
   are written to the directory `/run/user/${uid}/systemd/generator`
   where $uid is the UID of the user (that is `uid=$(id -u)`).
   The path can also written as `${XDG_RUNTIME_DIR}/systemd/generator`.

Note, files under `/run/` are deleted when the system is rebooted.

## Debugging quadlets

TODO, write more.

```
/usr/lib/systemd/system-generators/podman-system-generator {--user} --dryrun
```

or use

```
systemd-analyze {--user} --generators=true verify example.service
```

# quadlet directives

Note it is possible to mix systemd directives and quadlet directives.



## `Network=`

The directive `Network=` can be specified under the `[Container]` section in a quadlet container unit (file path suffix _.container_)

See.

## `NetworkName=`

The directive `NetworkName=` can be specified under the `[Network]` section in a quadlet network unit (file path suffix _.network_)

Set the name of the custom network.

TODO: how does network name work with templated instances?

The default network name given by Podman is `network-$filename`

## `Pod=`

The directive `Pod=` can be specified under the `[Container]` section in a quadlet container unit (file path suffix _.container_)

It's not possible to map UID/GID individually on each contianer when using a Pod.

