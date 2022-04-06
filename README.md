# IPBXComplet

## Summary

 - [Starting materials](#startint-materials)
 - [IP Addresses](#our-ip-addresses)
 - [Virtual machines configuration](#virtual-machines-configuration)
 - [Phones configuration](#phones-configuration)
 - [Créer un projet](#créer-un-projet)
 - [Fourcher (forker) un projet](#fourcher-forker-un-projet)
 - [Gestion des fichiers](#gestion-des-fichiers)
 - [Demandes de fusion](#demandes-de-fusion)
 - [Le format Markdown](#am%C3%A9liorer-ses-textes-avec-le-format-markdown)
 - [Gestion des issues](#les-issues)
 - [FAQ](#faq)
 - [Liens](#liens)
 - [Glossaire](#glossaire)

### Starting materials :

We used two different phones for this project. 1 Aastra 6757i and 1 Grandstream GXV3140.

https://www.officeeasy.fr/aastra-6757i-reconditionne.html 
https://www.officeeasy.fr/grandstream-gxv3140.html


#### Our IP Adresses

We initialize the IP Addresses of each phone such that:
     GrandStream: 10.33.105.127
     Aastra: 10.33.105.121


##### Virtual machines configuration


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


###### Phones configuration

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

We change the line: noload => chan_oss.so
with : load => chan_oss.so


After setting up everything we go to the asterisk command:
asterisk -rvvvvv


Then we call our phone:
    console dial NUMBER
