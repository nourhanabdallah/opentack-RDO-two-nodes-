# openstack-RDO-two-nodes-
OpenStack is a collection of interoperable components that can be deployed to provide computing, networking and storage resources. Those infrastructure resources can then be accessed by end users through programmable APIs.


OpenStack is an open source cloud software which provides infrastructure-as-a-service (IaaS). It can be installed on single and multiple nodes.

In this project we will deploy openStack on two nodes ( compute and ( controller + network) ) on CentOS 8.x using RDO repository and packstack utility. For Single Node OpenStack Installation refer the below :  https://www.rdoproject.org/install/packstack/


You can learn more about the various components in OpenStack at: https://openstack.org/software





## 1.Environment on two nodes :


* Use CentOS Stream 8-stream.


During the installation of the CentOS OS. Move to Install CentOS Stream 8-stream and click tab on your keyboard.
![image](https://user-images.githubusercontent.com/125203973/219944735-d457891f-e089-4f9c-acf0-c5e41f46e2fd.png)


 * Type a space to separate than the last kernel option, type 
 ` net.ifnames=0 biosdevname=0` 

Then hit Enter.

![image](https://user-images.githubusercontent.com/125203973/219944834-2061cc25-56b5-4c8e-b187-9f0a0b54dadf.png)

This will enable old Linux NIC card naming (eth0, eth1, …etc).

The names of the cards as you can see below is eth0 and eth1


![image](https://user-images.githubusercontent.com/125203973/219944924-61504aaf-0308-4a2a-8a59-6f0f0afea507.png)

 * ` Disable IPv6` 

 * Make sure that ` NTP`  on the top right is enabled, click on the gear icon and wait for a moment to make sure it is syncing with centos public NTP servers.

# 2-Prerequisites Deployment:

## compute_node

A single VM of MINIMUM 6 GB RAM and 2 NICs

2.1- Environment Setup:

 * Stop and disable firewalld


`systemctl stop firewalld`
`systemctl disable firewalld`


![image](https://user-images.githubusercontent.com/125203973/219945226-859c760a-03ba-432a-8152-164ca13b94d7.png)

 * Disable SeLinux

`Vim /etc/selinux/config`

![image](https://user-images.githubusercontent.com/125203973/219945300-97ae38d5-305d-4bed-8c6f-2f02bd332585.png)



* Download network-scripts
`dnf install -y network-scripts`


* Stop network manager 
`systemctl stop  NetworkManager` 


![image](https://user-images.githubusercontent.com/125203973/219945343-6a2042d2-a903-47d3-a4d5-3290109679e4.png)

* `Edit /etc/hosts`


![image](https://user-images.githubusercontent.com/125203973/219945395-00dcbfa1-1828-4818-a627-e0a54b1ef82f.png)


* `dnf config-manager --enable powertools`

* `dnf install -y centos-release-openstack-yoga`

* Update the OS

  `Dnf -y update` 

* `dnf install -y openstack-packstack`


* `systemctl list-units --type=service | grep openstack`

![image](https://user-images.githubusercontent.com/125203973/219945476-f9fa33fc-1f8e-4102-bf53-61f14d1c759d.png)




-----------------------------------------------------------------

## controller and network node 
A single VM of MINIMUM 8 GB RAM and 2 NICs


* Stop and disable firewalld

`systemctl stop firewalld`
`systemctl disable firewalld`

* Stop networkmanager

`systemctl status   NetworkManager`


*`dnf install -y network-scripts`

  `systemctl enable network`
  `systemctl start network`
 `systemctl status  network`

* edit hosts file 
* 
 `vim /etc/hosts`
 
 ![image](https://user-images.githubusercontent.com/125203973/219945956-712fcb64-69b4-4385-8abc-3682793848c5.png)
 
* disable selinux `vim /etc/selinux/config`
* `dnf config-manager --enable powertools`

* `dnf install -y centos-release-openstack-yoga`

* Update the OS

`Dnf -y update `

* dnf install -y openstack-packstack


* Generate answer file 


`packstack --gen-answer-file=/root/answers.txt --os-neutron-l2-agent=openvswitch --os-neutron-ml2-mechanism-drivers=openvswitch --os-neutron-ml2-tenant-network-types=vxlan --os-neutron-ml2-type-drivers=vxlan,flat --provision-demo=n --os-neutron-ovs-bridge-mappings=extnet:br-ex --os-neutron-ovs-bridge-interfaces=br-ex:eth0 --keystone-admin-passwd=redhat --os-heat-install=n `



*edit answer file adding ip of compute node  `vim /root/answers.txt`

![image](https://user-images.githubusercontent.com/125203973/219946043-a57e9138-ec05-4fce-b2bd-0155543e973d.png)

* Source keystone file 
* run answer file `packstack --answer-file=/root/answers.txt -t 3600`

``Don’t worry it will take a long while``

* installation done

![image](https://user-images.githubusercontent.com/125203973/219946127-98ab4737-ff60-4980-9feb-38d1908ee3f5.png)



---------------------------------------
## allow two nodes communication passwordless 
Generate ssh between two node passwordless (ssh-key )

`ssh-keygen`

`ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.24.131`
------------------------------
## validation 
* Check that compute, storage and networking service is enabled and UP 

![image](https://user-images.githubusercontent.com/125203973/219946938-740de911-e429-4e98-8afd-72265d7bb96a.png)

*  check 3 endpoints

 ![image](https://user-images.githubusercontent.com/125203973/219947017-41b839b9-bff7-43b9-8064-da02b1e7f175.png)
 
* network components

 ![image](https://user-images.githubusercontent.com/125203973/219947071-f0fe68df-f942-46e4-94f7-69787caf98c6.png)
 
*  through cli

 ![image](https://user-images.githubusercontent.com/125203973/219947170-4211470e-0659-45a9-97f1-cbf01f4f8ce6.png)
-----------------------------------------------------
## networking 

* `neutron net-create  "external" --provider:network_type flat --provider:physical_network extnet --router:external`



* `neutron subnet-create --name "external-subnet" --enable_dhcp=False --allocation-pool start="192.168.24.200",end="192.168.24.250" --gateway="192.168.24.2" "external" "192.168.24.0/24"`


* `neutron net-create "private"`

* `neutron subnet-create --name "private-subnet" "private"  "10.10.10.0/24"`

* `neutron router-create "router"`

* `neutron router-gateway-set "router" "external"`

* `neutron router-interface-add "router" "private-subnet"`
* ![image](https://user-images.githubusercontent.com/125203973/219947352-86979756-8d24-46b0-8b14-8e68dd002ddf.png)
* ![image](https://user-images.githubusercontent.com/125203973/219947374-e15d2e99-35de-479d-b0dc-26ce574831fe.png)
* ![image](https://user-images.githubusercontent.com/125203973/219947521-61af1252-e23c-4482-a749-8cc3d0b5dcde.png)
----------------------------------------------
## create instance 

* Install image file and Create image using glance component
`curl -L http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img | glance image-create --name='cirros image' --visibility=public --container-format=bare --disk-format=qcow2`

*launch instance `openstack server create --flavor 1 --image 21316e2c-899f-40b0-8999-4a15839c0dbc  --nic net-id=594a9aa8-5271-4ea7-b9ca-79f05af76753  "nour"`
* Floating ip and assign it to an instance `openstack floating ip set --port <instance port ID> <floating IP>`   ![image](https://user-images.githubusercontent.com/125203973/219947817-4cc34fe5-11ba-49ba-b0c3-9400a021db63.png)


* ![image](https://user-images.githubusercontent.com/125203973/219947844-b0a1f7d1-c78c-4834-83ce-8f5255717dd2.png)


* ![image](https://user-images.githubusercontent.com/125203973/219947673-2a664b21-fbda-41b8-b06e-f835a1334457.png)
--------------------------------------------------------------------
## Configure security group of the instance 
* ![image](https://user-images.githubusercontent.com/125203973/219947724-d0b2d361-f1e9-4c71-949d-ae77550e9724.png)
----------------------------------------------------------------------------
## Check that the instance is reachable from the external network

*![image](https://user-images.githubusercontent.com/125203973/219947748-69eab30e-82f3-41fd-a526-9e4aefc11c60.png)
 
*![image](https://user-images.githubusercontent.com/125203973/219947766-9b7aa464-3f89-4639-9ee2-282df7a08106.png)

_________________________________________________
## Dashboard 
![image](https://user-images.githubusercontent.com/125203973/219947946-7384f2c0-e6fa-487a-a041-9e55229ee57a.png)











