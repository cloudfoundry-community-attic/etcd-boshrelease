# BOSH Release for etcd

## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/etcd-boshrelease.git
cd etcd-boshrelease
bosh upload release releases/etcd-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a 3 VM cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For Openstack (Nova Networks), create a three-node cluster:

```
templates/make_manifest openstack-nova
bosh -n deploy
```

For AWS EC2, create a three-node cluster:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

Now store some data on one node:

```
curl -X PUT -L http://10.244.0.10:4001/v2/keys/url -d value="db.example.com"
{"action":"set","node":{"key":"/url","value":"db.example.com","modifiedIndex":4,"createdIndex":4}}
```

And fetch it from another node:

```
curl http://10.244.0.6:4001/v2/keys/url                                     
{"action":"get","node":{"key":"/url","value":"db.example.com","modifiedIndex":4,"createdIndex":4}}
```

You can now start/stop nodes (bosh jobs) and the cluster recovers:

```
bosh stop etcd_leader_z1 0
bosh start etcd_leader_z1 0
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

The security group should allow port 22 and 4001 (for incoming etcd traffic), and port 7001 between the servers themselves.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: etcd1
    type: dynamic
    cloud_properties:
      security_groups:
        - etcd
```

Where `- etcd` means you wish to use an existing security group called `etcd`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```
