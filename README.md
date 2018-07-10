# vagrant-kubernetes

### Installation

Run the command below under the project folder and wait for the instances to spin up

```
vagrant up
```

The project is set up to launch 3 instances: one master and two worker node. You can configure this under the NODE_COUNT variable at the top of the file.
Instance information like CPU and MEM allocation can be also modified. Default is 4gb mem with 2 VCPU

### Provisioning with ansible

Requirements 
- ansible
- python2 

To install ansible: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

After creating the instances you can run the following command to check the dynamic inventory. THis communicates with vagrant to check available hosts and split them into the master and the nodes group

```
> ./inventory.py --list
{"nodes": ["node1", "node2"], "vagrant": ["master", "node1", "node2"]}
```

To check host level information use:
```
> ./inventory.py --host master
"ansible_port": "2222", "ansible_host": "127.0.0.1", "ansible_user": "vagrant", "ansible_private_key_file": "/Users/mihneaspirescu/Documents/Vubiquity/kubernetes/.vagrant/machines/master/virtualbox/private_key"}%
```

In order to provision just do:

```
> ansible-playbook kubernetes.yml
```

and to check the steps we are doing:

```
> ansible-playbook kubernetes.yml --list-tasks

playbook: kubernetes.yml

  play #1 (all): Initial configuration	TAGS: []
    tasks:
      Add docker repo	TAGS: []
      Add kubernetes repo	TAGS: []
      Install packages	TAGS: []
      Remove swap	TAGS: []
      Comment out swap config	TAGS: []
      Put selinux in Permissive	TAGS: []
      Start and enable Docker	TAGS: []
      Add docker group	TAGS: []
      Add vagrant user to the docker group for easy docker access	TAGS: []
      Update kernel props	TAGS: []
      Create folder for Docker config	TAGS: []
      Copy docker config	TAGS: []
      Run and enable kubernetes	TAGS: []

  play #2 (master): Configure master	TAGS: []
    tasks:
      Initiate cluster	TAGS: []
      Crate kube config folder	TAGS: []
      Copy the admin kubectl config	TAGS: []
      Assign ownership	TAGS: []
      Copy flannel config	TAGS: []
      Create flannel network	TAGS: []
      get token	TAGS: []
      get certificate SHA	TAGS: []
      save SHA and token	TAGS: []
      debug	TAGS: []
      debug	TAGS: []

  play #3 (nodes): Configure nodes	TAGS: []
    tasks:
      Join master	TAGS: []

```


When finished just connect to the master to check the nodes

```
vagrant ssh master
sudo su                       # only the root user was configured with kubectl
kubectl get nodes	      # you should see the node you just attached
```


### Reprovisioning 

If you make some changes to the provisioning section of the Vagrantfile just run

```
vagrant provision
```

### Notes

To cleanup just run
```
vagrant destroy
```
