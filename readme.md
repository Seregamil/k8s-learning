# k8s tutorial
## Step 1. Write cloud-init manifest
    - Sample cloud-init file: https://cloudinit.readthedocs.io/en/latest/reference/examples.html#including-users-and-groups

## Step 2. Init multipass nodes (3 master-nodes with 2G RAM and 3 work-nodes with 1.5G RAM)
    - multipass launch --name k8s-master{Node.ID} --memory 2G --cloud-init cloud-init.yml
    - multipass launch --name k8s-node{Node.ID} --memory 1536M --cloud-init cloud-init.yml

## Step 3. Make ansible inventory file with nodes for test successfully connection

```ini
[k8s-master]
k8s-master1 ansible_host=10.57.172.124
k8s-master2 ansible_host=10.57.172.117
k8s-master3 ansible_host=10.57.172.216

[k8s-worker]
k8s-worker1 ansible_host=10.57.172.213
k8s-worker2 ansible_host=10.57.172.221
k8s-worker3 ansible_host=10.57.172.145

[k8s-cluster:children]
k8s-master
k8s-worker
```

And test connection with default ansible ping command
> ansible --inventory ~/cloud-init/ansible/inventory/cluster.ini k8s-cluster -m ping

## Step 3. Clone git kubespray repo and install all requirements

    Clone repo
    $ git clone git@github.com:kubernetes-sigs/kubespray.git

    Install python requirements
    $ pip install -r requirements.txt    

    Make copy of dir
    $ cp -R ./kubespray/inventory/sample ./kubespray/inventory/cluster

    Edit cluster nodes for ansible
    ```ini
        [all]
        master1 ansible_host=10.57.172.124
        master2 ansible_host=10.57.172.117
        master3 ansible_host=10.57.172.216
        worker1 ansible_host=10.57.172.213
        worker2 ansible_host=10.57.172.221
        worker3 ansible_host=10.57.172.145

        # ## configure a bastion host if your nodes are not directly reachable
        # [bastion]
        # bastion ansible_host=x.x.x.x ansible_user=some_user

        [kube_control_plane:children]
        kube_master

        [kube_master]
        master1
        master2
        master3

        [etcd:children]
        kube_master

        [kube_node]
        worker1
        worker2
        worker3

        [calico_rr]

        [k8s_cluster:children]
        kube_master
        kube_node
        calico_rr
    ```

    Edit cluster configuration in group_vars/k8s_cluster/k8s-cluster.yaml (For expert)

## Run ansible-playbook installation tool (Need -b or --become)
    ansible-playbook --inventory=./inventory/cluster/inventory.ini ./cluster.yml --become