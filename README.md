# docker-adb

This repository contains a [Dockerfile](https://www.docker.io/) for the [Android Debug Bridge](http://developer.android.com/tools/help/adb.html).

## Gotchas

* The container needs extended privileges for USB access
* The host's `/dev/bus/usb` must be mounted on the container

## Security

The container is preloaded with an RSA key for authentication, so that you won't have to accept a new key on the device every time you run the container (normally the key is generated on-demand by the adb binary). While convenient, it means that your device will be accessible over ADB to others who possess the key. You can supply your own keys by using `-v /your/key_folder:/.android` with `docker run`.

## Usage

### Pattern 1 - Shared network on the same machine (easy)

This usage pattern shares the ADB server container's network with ADB client containers.

Start the server:

```
docker run -d --privileged -v /dev/bus/usb:/dev/bus/usb --name adbd sorccu/adb
```

Then on the same machine:

```
docker run --rm -ti --net container:adbd sorccu/adb adb devices
docker run --rm -ti --net container:adbd ubuntu nc localhost 5037 <<< 000chost:devices
```

**Pros:**

* No port redirection required
* No need to look up IP addresses
* `adb forward` works without any tricks

**Cons:**

* Cannot use bridged (or any other) network on the client container
* Only works if the server and client containers run on the same machine

### Pattern 2 - Remote client

This usage pattern works best when you want to access the ADB server from a remote host.

Start the server:

```
docker run -d --privileged -v /dev/bus/usb:/dev/bus/usb --name adbd -p 5037:5037 sorccu/adb
```

Then on the client host:

```
docker run --rm -ti sorccu/adb -H x.x.x.x -P 5037 adb devices
```

Where `x.x.x.x` is the server host machine.

**Pros:**

* Scales better (can use any number of hosts/clients)
* No network limitations

**Cons:**

* Need to be aware of IP addresses
* Higher latency
* You'll need to make other ports (e.g. from `adb forward`) accessible yourself

## Thanks

* [Jérôme Petazzoni's post on the docker-user forum explaining USB device access](https://groups.google.com/d/msg/docker-user/UsekCwA1CSI/RtgmyJOsRtIJ)

## License

See [LICENSE](LICENSE).
