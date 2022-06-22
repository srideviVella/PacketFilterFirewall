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

Until here, only external settings are given. Until and unless the actual configurations are made after switching on the machines, the network will not work properly. Now switch on the machines and follow along.

**Client Setup:**
From the command line, enter following command to get the interfaces available. '-a' option allows to output the inavailable network interfaces as well.
> ifconifg -a

Here the previously given network adapter as interface and if you want, you can check the MAC address given with the one in the settings.As there are no IP addresses previously, it will not show the IP address. To assign an IP address to the interface,  enter the following command:
> sudo ifconfig enps03 192.168.0.10/24 netmask 255.255.255.0 up

enp0s3 is the name of the interface from the ifconfig command. It could vary from system to system. The newly assigned IP is 192.168.0.10

Next step is to add a default gateway to the client. Whenever it needs to send a packet to a host in a remote network, it will send the packet to a default gateway. To add the gateway, enter the following command:
> sudo route add default gw 192.168.0.100

Note that we need to give the Gateway's ip address as 192.168.0.100. To confirm that the default gateway got set correctly, enter the following command and check whether the first line contains the ip address of the gateway.
> route -n

![routeClient](https://user-images.githubusercontent.com/102641432/174470744-44e8fbeb-0b66-4244-a4db-c7f52bb8f0bd.PNG)

The setup is complete. However, these network changes are not permanent. They will change to default values once the machines are restarted. In order to avoid that, the default configuration file in "/etc/netplan" folder needs to edited. To open,
> sudo vim /etc/netplan/01-netcfg.yaml

Add the following content:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.10/24
      gateway4: 192.168.0.100
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
The yaml file should be edited correctly. The indentation should be followed appropriately otherwise might run into issues.
To enable the configurations, enter the following command:
> sudo netplan apply

If it runs into any errors, run the same command with option -d to have debug information.

**Gateway Setup:**

The process is similar to that of client with a little changes as there are three interfaces to configure on gateway. After giving `ifconfig -a`, you should see three interfaces without any IP addresses being set. Using the following commands to configure the IP addresses on the client-side and server-side networks.

```
sudo ifconfig enp0s3 192.168.0.100 netmask 255.255.255.0 up
sudo ifconfig enp0s8 10.0.0.100 netmask 255.0.0.0 up
```

To check which interface is that of which network, you can look at the MAC addresses in the network settings you had set up previously. If you look at the screenshots of gateways above, you can see the MAC address for that adapter. Compare that with ifconfig output on gateway.

For example, in the following output, the highlighted MAC address matches with the MAC address from the client-net settings. So, the interface with name as "enp0s3" is the client-side network interface on the gateway.
![ifconfigGateway](https://user-images.githubusercontent.com/102641432/174934346-6888e4f4-656b-424f-8914-bda6086c941f.PNG)

One more interface is remaining. That is for the NAT network. As that is the interface that enables the gateway to access the outside network, DHCP client is enabled on it using the following command:

> sudo dhclient enp0s9

Now if you give the ifconfig command, you can see the ip address automatically assigned to the third interface.

Again, the assignment is not permanent. So it can be made permananent by adding the following content to /etc/netplan/01-netcfg.yaml

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.100/24
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    enp0s9:
      dhcp4: yes
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    enp0s8:
      dhcp4: no
      addresses:
        - 10.0.0.100/8
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

After saving the file with above content, to apply the configuration, give the following command:
> sudo netplan -d appy

The server and client machines doesn't have direct access to the outside network, i.e., internet. To allow the gateway to forwards the packets from client/server to external networks, enter the following command in the gateway VM.

> iptables -t nat -A POSTROUTING -o enp0s9 -j MASQUERADE

where enp0s9 is the interface connecting gateway to the outside network. For POSTROUTING, whenever the client sends a packet to say google site, the gateway changes the source address to it's own address as the client doesn't have the external address and sends the packet to the google. Google thinks that it received request from the gateway and sends the reply to it. Because of the MASQUERADE option, the gateway will remember the previous request and where it should send the reply in case it gets one. After receiving the reply from Google, gateway realises it is to be sent to the client. So it sends the packet to the client. This whole process is called NAT.
To enable the packet forwarding in a linux machine, the following command should also be entered:
> sudo sysctl -w net.ipv4.ip_forward=1

It writes the net.ipv4.ip_forward=1 to the sysctl configuration file and commits the change.

Now from the client, try the following command:
> ping 8.8.8.8

It should receive the replies from Google. If not, follow the steps properly. Note that in the iptables command, the interface should be that of NATNetworking. After server setup, server should also be able to access internet.

**Server Setup:**
The server setup is similar to client. Setup the ip address using the following command:
> sudo ifconfig enp0s3 10.0.0.10 netmask 255.0.0.0 up

Setup the default gateway address same as the gateway's address setup above on the server-side network:
> sudo route add default gw 10.0.0.100

Edit the /etc/netplan/01-netcfg.yaml file to make changes permanent:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
    dhcp4: no
    addresses:
      - 10.0.0.10/8
    gateway4: 10.0.0.100
    nameservers:
      addresses: [8.8.8.8, 8.8.4.4]
```

Finally, apply the new settings using following command:
> sudo netplan -d apply

You can check for server whether it is able to access the internet or not. If everything done correctly, it should be able to access the internet.
With this, the setting up virtual machines is done.
