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
quadlet files into systemd system services unit files.

For details about systemd generators, see the man page [systemd.generator](https://www.freedesktop.org/software/systemd/man/latest/systemd.generator.html).

## Converting quadlet files into systemd system service unit files

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

The program `podman-user-generator` converts quadlet files into systemd user services unit files.

To create _systemd user services_, do following steps

1. Create directory
   ```
   mkdir -p $HOME/.config/container/systemd
   ```
2. Save your quadlet files in the directory `$HOME/.config/container/systemd`
3. Reload the systemd user manager
   ```
   systemctl --user daemon-reload
   ```
   The systemd user manager now executes programs in the directory
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

It is possible to mix systemd directives and quadlet directives.

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

## `Notify=`

Use `Notify=true` if the container image has support for [_sd_notify_](https://www.freedesktop.org/software/systemd/man/latest/sd_notify.html#).
When such a container is started (for example `systemctl start demo.service`), a container process will notify the service manager when it is ready.
The command `systemctl start demo.service` then returns.

#### example: use helper tool `systemd-notify` in a container

The container unit is configured with `Notify=true`.
Use the systemd tool [`systemd-notify`](https://www.freedesktop.org/software/systemd/man/latest/systemd-notify.html) to send a ready notification to the service manager.

<details>
  <summary>Click me</summary>

1. Log in to an unprivileged user on a Linux system
2. Create directory
   ```
   mkdir src
   ```
3. Create file _src/Containerfile_ containing
   ```
   FROM docker.io/library/fedora
   RUN dnf -y install systemd
   ```
4. Build container image
   ```
   podman build -t demo src/
   ```

TODO: write the missing steps ....

</details>

# Show logs

| command | container output | podman output | systemd output |
| --      | --               | --            | --             |
| journalctl --user -xeu nginx.service | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| journalctl --user -xeu nginx.service CONTAINER_NAME=nginx | :heavy_check_mark: | | |

The journalctl filter `CONTAINER_NAME=` value should match the
[`ContainerName=`](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#containername)
value under the `[Container]` section in the quadlet container unit.

#### example: Use systemctl flag `--verbose` to show log output during service startup

<details>
  <summary>Click me</summary>

Requirements: [systemd 258](https://github.com/systemd/systemd/releases/tag/v258) or later.

1. Log in as an unprivileged user to a Linux system
2. `mkdir -p ~/.config/containers/systemd`
3. Create file `~/.config/containers/systemd/demo.container` containing
   ```
   [Container]
   Image=docker.io/library/nginx
   ```
4. Reload the system user manager
   ```
   systemctl --user daemon-reload
   ```
5. Start the service
   ```
   systemctl --user --verbose start demo.service
   ```
   The following output is printed
   ```
   $ systemctl --user --verbose start demo.service
   Nov 17 20:26:33 asus systemd[1473]: Starting demo.service...
   Nov 17 20:26:33 asus podman[1859]: 2025-11-17 20:26:33.380288556 +0100 CET m=+0.042211930 container create dd9af06670dec2826656c0cdb3edb369cc9553d0c89a181f3ec213edb9dce17f (image=docker.io/library/nginx:latest, name=demo, PODMAN_SYSTEMD_UNIT=demo.service, maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>)
   Nov 17 20:26:33 asus podman[1859]: 2025-11-17 20:26:33.356803938 +0100 CET m=+0.018727311 image pull d261fd19cb63238535ab80d4e1be1d9e7f6c8b5a28a820188968dd3e6f06072d docker.io/library/nginx
   Nov 17 20:26:33 asus podman[1859]: 2025-11-17 20:26:33.468549006 +0100 CET m=+0.130472310 container init dd9af06670dec2826656c0cdb3edb369cc9553d0c89a181f3ec213edb9dce17f (image=docker.io/library/nginx:latest, name=demo, PODMAN_SYSTEMD_UNIT=demo.service, maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>)
   Nov 17 20:26:33 asus podman[1859]: 2025-11-17 20:26:33.473043781 +0100 CET m=+0.134967085 container start dd9af06670dec2826656c0cdb3edb369cc9553d0c89a181f3ec213edb9dce17f (image=docker.io/library/nginx:latest, name=demo, maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>, PODMAN_SYSTEMD_UNIT=demo.service)
   Nov 17 20:26:33 asus systemd[1473]: Started demo.service.
   ```

</details>

# Encrypted secrets

To use a secret in a quadlet service, embed the encrypted secret directly into
the container unit file with the systemd directive `SetCredentialEncrypted=`.

To encrypt the secret, use `systemd-creds encrypt`

To bind-mount the secret directory into the container, use
`Volume=%d:/secretdir`

The systemd specifier `%d` is resolved to the directory where systemd has placed the plaintext secret.

Typically, the following container unit configuration is needed

```
[Container]
Volume=%d:/secretdir
Image=myimg
[Service]
SetCredentialEncrypted=foo: \
        70rBNnmpSA6n22iJf58WXSAAAAABAAAADAAAABAAAABSArK52DYeyetCYVAAAAAAAAAAA \
        AAAAAALACMA8AAAACAAAAAAngAgV8dWm25LgWwtvKft9QuRYFUiE2aWDUdQ4E2H71yDzi \
        wAEJ2FXGyBG/80Ddn3rcGGZk1RwUqvomZbouwUhOQXPtTnfPZCu80CCA7lGKBfRxrvXUQ \
        llkgc87IrgZ91XaPbtjpaSf6kqVxZH+iq5wXC+3HsD1mZgtQ2NEjbZGt6qEj1sXNLFgMj \
        PLWXUsDfS5sK7M1QS3EJLMKSAw+tAE4ACAALAAAEEgAgAAAAAAAAAAAAAAAAAAAAAAAAA \
        AAAAAAAAAAAAAAAAAAAEAAg2n8Z2A//zDoUhxKMEWhhFuiJIQySWPF7MsoWFe/xzOEAAA \
        AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHAAAAAAAAALv1epSair4LT2T \
        x2seJRhlmoY2kVpIGsrwhvnCutOM/a4oHc3yvILod7or3
```

The `SetCredentialEncrypted=` configuration is generated by the command
```
echo -n mysecret | systemd-creds --user encrypt -p --name=foo -
```

For details, see [`SetCredentialEncrypted=`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#SetCredentialEncrypted=)
and https://systemd.io/CREDENTIALS/

#### example: use `systemd-creds` to create an encrypted secret and
then use `SetCredentialEncrypted=` to read the secret in a container

<details>
  <summary>Click me</summary>

Requirements: [systemd 258](https://github.com/systemd/systemd/releases/tag/v258) or later.

1. Create a test user
   ```
   sudo useradd test
   ```
2. Start a login shell
   ```
   sudo machinectl shell --uid=test
   ```
3. `mkdir -p ~/.config/containers/systemd`
4. Create file `~/.config/containers/systemd/demo.container` containing
   ```
   [Container]
   ContainerName=democontainer
   Image=docker.io/library/alpine:latest
   Exec=sh -c "ls -l /secretdir ; cat /secretdir/foo"
   Volume=%d:/secretdir
   [Service]
   ```
5. Create the encrypted secret and append the `SetCredentialEncrypted=` configuration to the container unit file.
   ```
   echo -n mysecret | \
      systemd-creds --user encrypt -p --name=foo - \
      >> ~/.config/containers/systemd/demo.container
   ```
   After running the command, the file _~/.config/containers/systemd/demo.container_
   contains
   ```
   [Container]
   ContainerName=democontainer
   Image=docker.io/library/alpine:latest
   Exec=sh -c "ls -l /secretdir ; cat /secretdir/foo"
   Volume=%d:/secretdir
   [Service]
   SetCredentialEncrypted=foo: \
           70rBNnmpSA6n22iJf58WXSAAAAABAAAADAAAABAAAABSArK52DYeyetCYVAAAAAAAAAAA \
           AAAAAALACMA8AAAACAAAAAAngAgV8dWm25LgWwtvKft9QuRYFUiE2aWDUdQ4E2H71yDzi \
           wAEJ2FXGyBG/80Ddn3rcGGZk1RwUqvomZbouwUhOQXPtTnfPZCu80CCA7lGKBfRxrvXUQ \
           llkgc87IrgZ91XaPbtjpaSf6kqVxZH+iq5wXC+3HsD1mZgtQ2NEjbZGt6qEj1sXNLFgMj \
           PLWXUsDfS5sK7M1QS3EJLMKSAw+tAE4ACAALAAAEEgAgAAAAAAAAAAAAAAAAAAAAAAAAA \
           AAAAAAAAAAAAAAAAAAAEAAg2n8Z2A//zDoUhxKMEWhhFuiJIQySWPF7MsoWFe/xzOEAAA \
           AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHAAAAAAAAALv1epSair4LT2T \
           x2seJRhlmoY2kVpIGsrwhvnCutOM/a4oHc3yvILod7or3
   ```
6. Reload the system manager
   ```
   systemctl --user daemon-reload
   ```
7. Start the service
   ```
   systemctl --user start demo.service
   ```
8. Run command
   ```
   journalctl --user -xe -u demo.service | grep -B2 mysecret
   ```
   The following output is printed
   ```
   Sep 21 15:57:22 localhost.localdomain democontainer[11405]: total 4
   Sep 21 15:57:22 localhost.localdomain democontainer[11405]: -r--------    1 root     root             8 Sep 21 15:57 foo
   Sep 21 15:57:22 localhost.localdomain democontainer[11405]: mysecret
   ```
   __result:__ the secret text `mysecret` is printed by the container

</details>

Side note:
[`LoadCredentialEncrypted=`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#LoadCredential=ID:PATH) can be used instead
of `SetCredentialEncrypted=`. The encrypted secret is then loaded from a file. Unfortunately, `LoadCredentialEncrypted=` only works in a container unit
if SELinux is disabled in the container. In other words, `LoadCredentialEncrypted=` needs to be combined with `SecurityLabelDisable=true`.

## Managing quadlets of another user account

To manage quadlet services of another user, use for example `sudo systemctl --machine otheruser@ --user daemon-reload`
This even works when the other user has `/sbin/nologin` as login shell.

#### example: Create and start a container unit in another user account

<details>
  <summary>Click me</summary>

1. Create system user `otheruser`
   ```
   sudo useradd \
          --add-subids-for-system \
          --create-home \
          --shell /sbin/nologin \
          --system \
	  otheruser
   ```
   Note, it will not be possible to log into `otheruser` due to the command-line arguments `--shell /sbin/nologin`
2. Create directory
   ```
   sudo -u otheruser mkdir -p /home/otheruser/.config/containers/systemd
   ```
3. Create container unit file
   ```
   sudo -u otheruser sh -c 'echo -e "[Container]\nImage=alpine\nExec=sleep inf\n" > /home/otheruser/.config/containers/systemd/demo.container'
   ```
   The created file contains
   ```
   [Container]
   Image=alpine
   Exec=sleep inf
   ```
4. Enable linger
   ```
   sudo loginctl enable-linger otheruser
   ```
5. Reload the user systemd user manager for the user _otheruser_
   ```
   sudo systemctl --machine otheruser@ --user daemon-reload
   ```
6. Start the service
   ```
   sudo systemctl --machine otheruser@ --user start demo.service
   ```
7. Verify that the service is running
   ```
   sudo systemctl --machine otheruser@ --user is-active demo.service
   ```
   The following output is printed
   ```
   active
   ```

</details>
