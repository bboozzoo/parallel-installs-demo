### Parallel installation of snaps demo

Snapd supports installing many instances of the same snap. Instances are
distinguished using a unique instance key appended to the snap name. For
example:

- `<snap>` - is the regular snap name
- `<snap>_foo` - is another instance of snap with `foo` instance key

One can install assign instance keys directly when installing from store:

```
$ snap install somesnap_foo
```

Or from file:

```
snap install --name somesnap_foo somesnap_123.snap
```

Inside the snap's filesystem, locations `$SNAP`, `$SNAP_DATA` and `$SNAP_COMMON`
are set up in such way that they are identical for snaps with and without
instance key. On the outside, each is a separate place in the host filesystem.
This way, snaps that hardcode paths at build time (eg. using
`/var/snap/<snap>/common` explicitly) should continue working without changes.

Due to security concerns, user locations such as `$SNAP_USER_DATA`,
`$SNAP_USER_COMMON` are instance specific both inside and outside of snap.

### Demo

This is a snap for demoing parallel installation. The snap comes with a service
`serve` which is a simple HTTP server hosting files from `$SNAP_DATA`.

By default, the service listens on port 9991 on localhost.


#### Build the snap

```
$ snap pack
built: parallel-installs-demo_0.1_all.snap
```

#### Enable parallel instances feature

The parallel installation of snaps is gated by a feature flag and needs to
enable it explicitly.

```
$ sudo snap set system experimental.parallel-instances=true
```

#### Install snap without the instance key

```
$ sudo snap install --dangerous parallel-installs-demo_0.1_all.snap
parallel-installs-demo 0.1 installed
```

#### Install another instance with `foo` instance key

```
$ sudo snap install --dangerous --name parallel-installs-demo_foo parallel-installs-demo_0.1_all.snap
parallel-installs-demo_foo 0.1 installed
```

#### Configure `foo` instance port to be 9992

`parallel-installs-demo_foo.serve` will fail to start as the port is already
used by `parallel-installs-demo.serve`. We need to adjust the port:

```
$ sudo snap set parallel-installs-demo_foo port=9992
```

At this point the service should have been automatically restarted and using the
new port.

#### Access the services

The non keyed instance:

```
$ curl http://localhost:9991/hello.txt
Hello world!
```

The `foo` instance:

```
$ curl http://localhost:9992/hello.txt
Hello world from instance "foo"!
```

#### Create some files for each snap and view them

```
$ echo foobar | sudo tee /var/snap/parallel-installs-demo/current/some-data
$ echo other-foobar | /var/snap/parallel-installs-demo_foo/current/other-data
```

Open the browser and go to http://localhost:9991 and another tab for http://localhost:9992
