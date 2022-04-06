# IPBXComplet



## Summary

 - [Starting materials](#starting-materials)
 - [IP Addresses](#our-ip-adresses)
 - [Virtual machines configuration](#virtual-machines-configuration)
 - [Phones configuration](#phones-configuration)
 - [About the SIP Trunk](#what-is-a-sip-trunk-)

## Starting materials :

We used two different phones for this project. 1 Aastra 6757i and 1 Grandstream GXV3140.

https://www.officeeasy.fr/aastra-6757i-reconditionne.html 

https://www.officeeasy.fr/grandstream-gxv3140.html


## Our IP Adresses

We initialize the IP Addresses of each phone such that:
     GrandStream: 10.33.105.127
     Aastra: 10.33.105.121


## Virtual machines configuration


First step:

We first installed a debian image, then we created a hard disk image with the command:

       ex: qemu-img create -f qcow2 debian-journet-sebastiani.qcow2 4G


Then we use the kvm command to put the downloaded debian image in the previously created hard disk:

       ex: kvm -m 1000 -hda debian-journet-sebastiani.qcow2 -cdrom image-debian.iso


After that, we launch the virtual machine and configure it first:

       ex: kvm -m 1000 -hda debian-journet-sebastiani.qcow2 -net nic,macaddr=42:AA:AA:AA:AA:AA -net vde,sock=/var/run/vde2/kvmtap0.ctl -k fr

(To make it easier to configure everything on the machine, we will connect with the root)


To be sure to find the same machine with the right configurations to be able to resume later at the right place, we transfer a copy of our virtual machine on thorin:

       ex: scp debian-journet-sebastiani.qcow2 nom@thorin:debian-journet-sebastiani.qcow2


## Phones configuration

Now to configure the phones and to be able to do the other objectives we need asterisk on the machine:
apt-get install asterisk


First we move to the right place by doing: cd /etc/asterisk

Then, we enter the sip.conf and we enter the information about our phone:
nano sip.conf

ex:
 
       [gs2]
       type=friend
       secret=admin
       host=dynamic
       context=default


To be able to call with the phones we need to add an instruction to extensions.conf:

We are looking for the line  [default] (with CTRL+W)
       
       exten => 501,1,Dial(SIP/gs2,42)


To be able to call then we need to go to the modules.conf file:

We change the line: 

       noload => chan_oss.so
       
with : 
       
       load => chan_oss.so


After setting up everything we go to the asterisk command:
       
       asterisk -rvvvvv


Then we call our phone:
       
       console dial NUMBER
       
You can also see the status of the communication channels by typing (still in asterisk):
    
       sip show peers

## What is a SIP Trunk ?

SIP, which stands for Session Initiation Protocol, is an IP standard established by the Internet Engineering Task Force (IETF). A standard essentially dictating how a communication session is established and how it is terminated, initiating the connection between IP addresses.

SIP users no longer manage two separate networks for voice and data communications, they all go through the same channel. SIP is one of the most widely used protocols in VoIP systems.



How does SIP Trunk work ? 

See this diagram to better understand how the SIP Trunk works:
https://www.ringcentral.com/fr/fr/blog/wp-content/uploads/2021/11/SIP-Trunk.jpg



Configuring the SIP Trunk :
 
The configuration is done in several steps:

Step 1: 
First you have to edit the sip.conf file located in this directory /etc/asterisk/sip.conf
This first step will consist in creating and configuring the users


Client - Look for the line: OUTBOUND SIP REGISTRATIONS

Add: register => faisceau:tux@10.33.XXX.12


Step 2: 
Server - add the client definition :

[faisceau] 
type=friend ; Defines the type of calls : peer for outgoing calls / user for incoming calls / friend for both.

secret=tux ; Password for the trunk.

context=local ; Name of the context for the trunk in sip.conf

host=dynamic ; Name of the SIP server of the trunk.

insecure=port,prompt


We can check that everything went well by entering the command sip show peers to list the peers and sip show registry.

