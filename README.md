# Working-with-kvm-Configuration-Files


## Intro
In this write-up we will perform several techniques for working with virtual machines (a.k.a. `domains`) in Linux `kvm`.

## Requirements
You should already have `kvm` up and running, if not check out the relevant section at the following.

    https://github.com/mikenizo808/Building-the-Ultimate-Ubuntu-Desktop-on-a-Pre-Made-Gaming-PC


## Approaches
When working with `kvm` virtual machines (a.k.a. `domains`) we can power on, clone and otherwise manage from the command line or GUI. Most things we will do from terminal, but often we recommend simply using the GUI for simplicity.

## List VMs

    sudo virsh list --all


## Example Output

    mike@ubuntu03:/var/lib/libvirt$ sudo virsh list --all
    Id   Name                State
    ------------------------------------
    -    dc01.lab.local      shut off
    -    dc02.lab.local      shut off
    -    jump01              shut off
    -    kali001             shut off
    -    Win-2019-Template   shut off

## Use `dumpxml` to Export Configuration
Ideally, we should use the "GUI" or `virsh edit`, but we can also use `dumpxml` to export the configuration for a virtual machine. Later, we learn to edit/import this configuration.

    #syntax
    sudo virsh dumpxml VMNAME > domxml.xml

    #example
    sudo virsh dumpxml 'Win-2019-Template' > ~/Downloads/'Win-2019-Template.xml'

*Note: For more detail on `dumpxml` see the discussion at https://serverfault.com/questions/434064/correct-way-to-move-kvm-vm.*

## Optional - Look at Configuration
We can use `cat` to look at the entire file or `head` to just look at the beginning.

    cat /path/to/exported.xml

## Example Output

    mike@ubuntu03:$ head  ~/Downloads/'Win-2019-Template.xml'
    <domain type='kvm'>
    <name>Win-2019-Template</name>
    <uuid>bbdfbdcd-f419-4086-a074-0624c64d879c</uuid>
    <metadata>
        <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
        <libosinfo:os id="http://microsoft.com/win/2k19"/>
        </libosinfo:libosinfo>
    </metadata>
    <memory unit='KiB'>4194304</memory>
    <currentMemory unit='KiB'>4194304</currentMemory>
    mike@ubuntu03:/var/lib/libvirt$ 

## Using scp
If interested in using the GUI, see the end of this article to add ssh keys and connect multiple `kvm` hosts.

However, here we use `scp` to manually copy an iso or an entire virtual machine.

    #copy an iso
    sudo scp /var/lib/libvirt/images/my-windows.iso mike@10.200.0.215:/home/mike/Downloads/

    #copy entire VM
    sudo scp /var/lib/libvirt/images/Win-2019-Template.qcow2 mike@10.200.0.215:/home/mike/Downloads/

    #copy config file
    scp /home/mike/Downloads/Win-2019-Template.xml mike@10.200.0.215:/home/mike/Downloads/

## SSH to target host
Start an `ssh` session to the desired host that will run the copied VMs.

    ssh mike@10.200.0.215

## Optional - Edit the Config File

    nano ~/Downloads/Win-2019-Template.xml

## Move Disks as Needed
Here we move the disks (if needed) and also rename as desired.

    mike@ubuntu004:~/Downloads$ sudo mv jump01-clone.qcow2 /var/lib/libvirt/images/Win-2019-Template.qcow2

*Note: Remember that the config file should specify the desired disk name as well.*

## Register VM

    virsh define ~/Downloads/Win-2019-Template.xml

## Example Output

    mike@ubuntu004:~/Downloads$ sudo virsh define ~/Downloads/Win-2019-Template.xml
    Domain Win-2019-Template defined from /home/mike/Downloads/Win-2019-Template.xml

## List VMs

    sudo virsh list --all

## Example Output

    mike@ubuntu004:~/Downloads$ sudo virsh list --all
    Id   Name                State
    ------------------------------------
    -    Win-2019-Template   shut off


## Optional - Clone a VM

    sudo virt-clone --original Win-2019-Template --name dc03.lab.local --auto-clone


*Note: If you get an error about disk path, you may need to update your config file*

## Update a Device from Config File
The easiest way to update a config is with `virsh edit <nameofvm>`, but you can also update from a config file.

    sudo virsh update-device --domain Win-2019-Template --file ~/Downloads/Win-2019-Template.xml


## Delete a VM
If needed, you can un-register a domain, with `virsh undefine`.

    sudo virsh undefine <nameofvm>

*Note: The underlying disk should be deleted manually (i.e. from `/var/lib/libvirt/images`).*

## Example Output

    mike@ubuntu004:/var/lib/libvirt/images$ sudo virsh undefine Win-2019-Template
    Domain Win-2019-Template has been undefined

## Shutdown a VM

    #list VMS
    sudo virsh list --all

    #shutdown guest
    sudo virsh shutdown <nameofvm>

    #finish the shutdown (if still shows as running)
    sudo virsh destroy <nameofvm>


## Power on from Command line
This can be done from the gui using the `vmm` application, but we can optionally start from command line.

    #list virtual machines
    virsh list --all

    #start a particular vm
    virsh start yourvm


## Headless vs. Desktop GUI
If the guest operating system you deployed is configured for a headless mode then you can power on from the command line and start using it.  However, if you are using `Desktop Experience` (the Windows GUI), you might need to connect a monitor/keyboard/mouse to your host (at least until the guest is configured). Also note that you can use the `vmm` application and connect to a remote hypervisor using ssh keys and consume the display over the network.


## SSH Keys
To connect to `kvm` on a remote host using the GUI, we first must enable `ssh` keys for passwordless connection.

    #check if key exists
    ls -l ~/.ssh/id_*.pub

    #create key
    ssh-keygen -t rsa -b 4096 -C "mike@lab.local"

    #show if it exists now
    ls ~/.ssh/id_*

    #copy key to remote node
    ssh-copy-id mike@10.200.0.215

    #connect to remote node
    ssh mike@10.200.0.215

*Note: Once you have ssh keys setup like above, launch the `vmm` GUI application and navigate to `File > Add Connection`.*

