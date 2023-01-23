# k8s tutorial
## Step 1. Write cloud-init manifest
    - Sample cloud-init file: https://cloudinit.readthedocs.io/en/latest/reference/examples.html#including-users-and-groups

## Step 2. Init multipass nodes (3 master-nodes with 2G RAM and 3 work-nodes with 1.5G RAM)
    - multipass launch --name k8s-master{Node.ID} --memory 2G --cloud-init cloud-init.yml
    - multipass launch --name k8s-node{Node.ID} --memory 1536M --cloud-init cloud-init.yml

## Step 3. Make ansible inventory file with nodes for test successfully connection

## Step 3. Clone git kubespray repo