# kuttle: kubectl wrapper for sshuttle without SSH

Kuttle allows you to easily get an access into your Kubernetes network environment. SSH access is not required, since `kubectl` is used instead of `ssh`.

In comparison with [Telepresence](https://www.telepresence.io), `kuttle` only proxies Kubernetes network onto your local laptop.

# Installation

Install `sshuttle` following [official documentation](https://sshuttle.readthedocs.io/en/stable/installation.html) or use your distro's package manager:

* MacOS: `brew install sshuttle`
* Debian/Ubuntu: `apt-get install sshuttle`
* Fedora/RedHat/CentOS: `yum install sshuttle`

Download [kuttle](https://github.com/kayrus/kuttle/raw/master/kuttle):

```sh
wget https://github.com/kayrus/kuttle/raw/master/kuttle
chmod +x kuttle
```

Additionally you can place `kuttle` into `$PATH`

# How does it work?

Under the hood `sshuttle` spawns a remote python oneliner that evals `stdin` and sends a code via `stdin` that executes a server part of `sshuttle`, which *proxies* the traffic. To get a connection to the remote server `sshuttle` usually uses `ssh`. `kuttle` allows `sshuttle` to use `kubectl` without any `ssh` dependencies.

# Target Kubernetes pod requirements

Since `sshuttle` uses python interpreter, python should be installed inside target pod's container.

Prior to version 0.78.2, sshuttle used `netstat` to [list routes](https://github.com/sshuttle/sshuttle/pull/132). If your `sshuttle` version is older than 0.78.2, you have to ensure that `netstat` CLI is also installed inside pod's container.

Simple alpine container with a minimal python is enough for `kuttle`. You can use the `kubectl` command below in order to spawn ready-to-use pod as a VPN server:

```sh
kubectl run kuttle --image=alpine:latest --restart=Never -- sh -c 'apk add python --update && exec tail -f /dev/null'
sshuttle -r kuttle -e kuttle 0.0.0.0/0
```

# Examples

Route local requests to the `172.16.0.0/24` subnet via `pod-with-python` pod in your Kubernetes cluster:

```sh
sshuttle -r '--context my-context --namespace default pod-with-python' -e /path/to/kuttle 172.16.0.0/24
```

Use your Kubernetes pod as a VPN server with DNS requests being resolved by pod:

```sh
sshuttle --dns -r '--context my-context --namespace default pod-with-python' -e /path/to/kuttle 0.0.0.0/0
```

If you already have set `kubectl` defaults and placed `kuttle` in `$PATH`, just specify the pod name:

```sh
sshuttle --dns -r pod-with-python -e kuttle 0.0.0.0/0
```

# Credits

Thanks to sshuttle authors and [@databus23](https://github.com/databus23) for getting me inspired.
