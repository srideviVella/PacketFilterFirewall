This project contains mainly three virtual machines namely, server, gateway and client. Gateway is the machine which has access to the internet. 
This setup resembles any network in general. Server side network is considered as the internal network and client side network is considered as the outside network which 
contains any user/attacker possibly. The following diagram shows the architecture of the network.

![Architecture](images/ProjectArch.PNG)

Create a ubuntu virtual machine using Virtual Box. The network settings should be as follows:

All the vms should contain promiscous mode as "Allow All"
For Client:

![ClientNw](https://user-images.githubusercontent.com/102641432/174468606-e9e8bd96-6edf-488e-a167-a26a493c2f55.PNG)


As shown above, in Attached to, give the option as "Internal Network" and give a name for that network. I have given it as "client-net". CLient vm has only one adapter as it is directly connected to the gateway. 

For Server:

![ServerNw](https://user-images.githubusercontent.com/102641432/174468600-e0b529b9-d188-4b6c-bdd6-29917e6457d1.PNG)


Follow the same steps as server but the name should be different as both server and client are in different networks. For the name, I gave it as "server-net".

For Gateway:

For Gateway, it is little different as it will be having 3 adapters each with server, client, and the outside internet. For adapter one, give the name same as that of clients network adapter after choosing "Attached to" as "Internal Network". And remaining options are same.

![GatewayNwClient](https://user-images.githubusercontent.com/102641432/174468594-06792e58-c15a-493f-82fc-e04bc290d4d5.PNG)


For adapter 2, let's say it connects to the server, choose the "Attached to" as "Internal Network" and the name is same as that of server's adatper.
![GatewayNwServer](https://user-images.githubusercontent.com/102641432/174468587-2728e8cd-c4a3-4cf5-bfb0-415239c9c2d3.PNG)


For adapter 3, it connects to the internet. Choose the "Attached to" as "NAT Network". By default there is a NAT Network defined in the virtual box. To view it, close the current window. Go to File->Preferences. In the opened window, click on the Network section. In it there is default network as shown below. If you could not see it, create one. If you open the settings of it, you can see the CIDR IP address range given in it.

![NATDetails](https://user-images.githubusercontent.com/102641432/174468565-3f7f6b45-3d0d-4d72-9500-31a48b887afd.PNG)

![GatewayNw](https://user-images.githubusercontent.com/102641432/174468577-93d968aa-bcc4-4e03-8a65-52cafdecd3e5.PNG)


**Client Setup:**
From the command line, enter following command to get the interfaces available. '-a' option allows to output the inavailable network interfaces as well.
> ifconifg -a

