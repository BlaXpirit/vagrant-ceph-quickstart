## Implementation of Ceph [Quick Start examples](http://docs.ceph.com/docs/master/start/) in Vagrant

This allows you to run a [Ceph][] cluster in [VirtualBox][] virtual machines, in a minimal, reproducible environment, thanks to [Vagrant][].

```bash
vagrant plugin install vagrant-vbguest
vagrant up  # Inside the folder with Vagrantfile
# Wait...
```

```bash
vagrant ssh        # admin node
vagrant ssh node1  # mon+mds+rgw node
vagrant ssh node2  # osd node
vagrant ssh node3  # osd node
```

Things to try (inside the admin node):

```bash
ceph status

ceph osd pool create data 48
rados put --pool=data test-object <(echo test-data)
ceph osd map data test-object
```

With this setup you can access the cluster from the host machine. The folder with *Vagrantfile* is synchronized to contain the Ceph's keys. You can do all the same things as from the admin node, or even use this as a client.

```bash
ceph -k ceph.client.admin.keyring status

key='-k ceph.client.admin.keyring'
rados $key --pool=data get test-object - 
```

The next fun thing to do is [making a block device](http://docs.ceph.com/docs/master/start/quick-rbd/#configure-a-block-device) on your host machine, to store arbitrary file systems... inside a cluster of VMs.  
Just don't forget to pass the `-k` argument!

[ceph]: //ceph.com
[vagrant]: //vagrantup.com
[virtualbox]: //virtualbox.org
