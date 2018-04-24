# vagrant-kubernetes

###Installation

Run the command below under the project folder and wait for the instances to provision

```
vagrant up
```

The project is set up to launch 2 instances: one master and one worker node. You can configure this under the NODE_COUNT variable at the top of the file.
Instance information like CPU and MEM allocation can be also modified. Default is 4gb mem with 2 VCPU

The master node is already provisioned as a kubernetes cluster of one. To add the other nodes (here node1, node2 ...) to the clsuter use the following commands

On master
```
vagrant ssh master
cat /root/kubeadm.output      # this is the output of the cluster initiation. use the kubeadm join command presented there 

```

On nodeX
```
vagrant ssh node1
kubeadm join 10.0.0.10:6443 --token YYYYYYYYYYY  --discovery-token-ca-cert-hash sha256:XXXX     # this is the command you copied from master /root/kubeadm.output
```

On master
```
sudo su                       # only the root user was configured with kubectl
kubectl get nodes	      # you should see the node you just attached
```


### Reprovisioning 

If you make some changes to the provisioning section of the Vagrantfile just run

```
vagrant provision
```

###Notes

Note that this also exposes an NFS server on the master for testing. It can be monted on the other nodes with
```
mount -t nfs 10.0.0.10:/nfs /mnt 
```

To cleanup just run
```
vagrant destroy
```
