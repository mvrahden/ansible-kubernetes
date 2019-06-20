# ansible-k8s <!-- omit in toc -->

An [Ansible](https://www.ansible.com) role to deploy [Kubernetes](https://kubernetes.io)  (k8s) on a cluster of debian based systems.

## Table of Contents <!-- omit in toc -->

- [Side notes](#Side-notes)
- [Role Variables](#Role-Variables)
- [Dependencies](#Dependencies)
- [Use the `Vagrant` Playground](#Use-the-Vagrant-Playground)
  - [Requirements](#Requirements)
  - [How to use it](#How-to-use-it)
- [Additional Info](#Additional-Info)
  - [Reset `k8s` cluster](#Reset-k8s-cluster)
  - [Get a list of pods and information on them](#Get-a-list-of-pods-and-information-on-them)
- [License](#License)
- [Author Information](#Author-Information)

## Side notes

- By default [coreos/flannel](https://coreos.com/flannel/docs/latest/) is used for
  the pod networking.
- but this role is as well compatible to [weaveworks/weave](https://www.weave.works/docs/net/latest/kube-addon/).

## Role Variables

[defaults/main.yml](defaults/main.yml)

## Dependencies

None

## Use the `Vagrant` Playground

Included in the `Vagrant` folder is a testing environment with `3` nodes.

- `node0` - K8s Cluster Master (`192.168.250.10`)
- `node1` - K8s Cluster Member (`192.168.250.11`)
- `node2` - K8s Cluster Member (`192.168.250.12`)

The folder includes an examplary `Ansible` playbook.

### Requirements

Packages installed on your system:

  - [Ansible](https://www.ansible.com)
  - [Vagrant](https://www.vagrantup.com/)
  - [Virtualbox](https://www.virtualbox.org/)

### How to use it

`cd` into that directory and install additional required [Ansible](https://www.ansible.com) roles:

```bash
cd Vagrant/
ansible-galaxy install -r requirements.yml
```

Now you can easily spin this up for learning purposes:

```bash
vagrant up
```

Once the environment spins up you will see the following:

```bash
TASK [ansible-k8s : cluster_summary | Displaying Cluster Nodes] ****************
skipping: [node1]
ok: [node0] => {
    "_k8s_cluster_nodes['stdout_lines']": [
        "NAME      STATUS     AGE       VERSION",
        "node0     Ready      1m        v1.14.1",
        "node1     NotReady   4s        v1.14.1",
        "node2     NotReady   6s        v1.14.1"
    ],
    "changed": false
}
skipping: [node2]
```

Do not worry about the above as the additional nodes did not completely join
the cluster before the provisioning completed. You can quickly validate that
the additional nodes are up and `Ready` by running:

```bash
ansible-playbook -i hosts playbook.yml --tags k8s_cluster_nodes
```

The above `NotReady` should no longer be an issue as we now wait for all nodes
in the cluster to become `Ready`. However, there may be an instance where this
may not work as expected.

```bash
TASK [ansible-k8s : cluster_summary | Displaying Cluster Nodes] ******************************************************************************************************************
skipping: [node1]
skipping: [node2]
ok: [node0] => {
    "_k8s_cluster_nodes['stdout_lines']": [
        "NAME      STATUS    ROLES     AGE       VERSION",
        "node0     Ready     master    3m        v1.14.3",
        "node1     Ready     <none>    2m        v1.14.3",
        "node2     Ready     <none>    2m        v1.14.3"
    ]
}
```

Once the cluster is up `ssh` to `node0` and begin playing:

```bash
vagrant ssh node0
```

When you are all done using the environment easily tear it down:

```bash
./cleanup.sh

==> node2: Forcing shutdown of VM...
==> node2: Destroying VM and associated drives...
==> node1: Forcing shutdown of VM...
==> node1: Destroying VM and associated drives...
==> node0: Forcing shutdown of VM...
==> node0: Destroying VM and associated drives...
```

## Additional Info

### Reset `k8s` cluster

```bash
ansible-playbook -i hosts playbook.yml --tags k8s_reset -e "k8s_reset_cluster=true"
```

### Get a list of pods and information on them

```bash
ansible-playbook -i hosts playbook.yml --tags k8s_pods
```

```javascript
{
        "containers": [
            {
                "hostIP": "192.168.250.10",
                "image": "gcr.io/google_containers/etcd-amd64:3.1.10",
                "name": "etcd",
                "nodeName": "node0",
                "phase": "Running",
                "podIP": "192.168.250.10",
                "resources": {}
            },
            {
                "hostIP": "192.168.250.10",
                "image": "gcr.io/google_containers/kube-apiserver-amd64:v1.14.3",
                "name": "kube-apiserver",
                "nodeName": "node0",
                "phase": "Running",
                "podIP": "192.168.250.10",
                "resources": {
                    "requests": {
                        "cpu": "250m"
                    }
                }
            },
            /* ... */
        ]
    }
}
```

## License

MIT

## Author Information

- Larry Smith Jr.
- Menno van Rahden
