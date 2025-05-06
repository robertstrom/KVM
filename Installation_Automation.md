#QEMU 
#KVM 
#Virtualization 
#qcow2 
#Automation 

# Resources

Some of the ideas for the templating of the install came from this post

[Configuring a VM before starting with virt-install?](https://www.reddit.com/r/qemu_kvm/comments/14eeaom/configuring_a_vm_before_starting_with_virtinstall/)
[KVM VirtIO Driver downloads](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/)


# Introduction


- This page will document an example KVM XML templates for Windows and Linux (the Windows one has configurations that optimize the virtual machine for Windows 11) which will be used to produce an XML template which will define the virtual machine using a script that also creates the associated virtual hard drive
- The XML example template is shown below
	- Make note of the variables in the XML file
		- ${KVM_VM_NAME}
		- ${KVM_VM_MEMORY}
		- ${KVM_VM_CPUS}
		- ${KVM_INSTALL_ISO}
		- ${KVM_VIRTIO_ISO} - only needed for Windows installs
	- The script has another variable that is used to create the virtual hard disk
		- KVM_HD_SIZE


## Windows KVM Template

**kvm_xml_template_windows.xml**
```
<domain type="kvm">
  <name>${KVM_VM_NAME}</name>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://microsoft.com/win/11"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="GiB">${KVM_VM_MEMORY}</memory>
  <currentMemory unit="GiB">${KVM_VM_MEMORY}</currentMemory>
  <memoryBacking>
    <source type="memfd"/>
    <access mode="shared"/>
  </memoryBacking>
  <vcpu placement="static">${KVM_VM_CPUS}</vcpu>
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-7.2">hvm</type>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <runtime state="on"/>
      <synic state="on"/>
      <stimer state="on">
        <direct state="on"/>
      </stimer>
      <reset state="on"/>
      <vendor_id state="on" value="KVM Hv"/>
      <frequencies state="on"/>
      <reenlightenment state="on"/>
      <tlbflush state="on"/>
      <ipi state="on"/>
      <evmcs state="on"/>
    </hyperv>
    <vmport state="off"/>
    <smm state="on"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on"/>
  <clock offset="localtime">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
        <disk type="file" device="cdrom">
      <driver name="qemu" type="raw"/>
      <source file="${KVM_INSTALL_ISO}"/>
      <target dev="sdb" bus="sata"/>
      <readonly/>
      <boot order="2"/>
      <address type="drive" controller="0" bus="0" target="0" unit="1"/>
    </disk>
    <disk type="file" device="cdrom">
      <driver name="qemu" type="raw"/>
      <source file="${KVM_VIRTIO_ISO}"/>
      <target dev="sdc" bus="sata"/>
      <readonly/>
      <address type="drive" controller="0" bus="0" target="0" unit="2"/>
    </disk>
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" cache="none" discard="unmap"/>
      <source file="/var/lib/libvirt/images/${KVM_VM_NAME}.qcow2"/>
      <target dev="vda" bus="virtio"/>
      <boot order="1"/>
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x02" slot="0x00" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pcie-root"/>
    <controller type="pci" index="1" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="1" port="0x10"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="2" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="2" port="0x11"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x1"/>
    </controller>
    <controller type="pci" index="3" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="3" port="0x12"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x2"/>
    </controller>
    <controller type="pci" index="4" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="4" port="0x13"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x3"/>
    </controller>
    <controller type="pci" index="5" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="5" port="0x14"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x4"/>
    </controller>
    <controller type="pci" index="6" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="6" port="0x15"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x5"/>
    </controller>
    <controller type="pci" index="7" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="7" port="0x16"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x6"/>
    </controller>
    <controller type="pci" index="8" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="8" port="0x17"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x7"/>
    </controller>
    <controller type="pci" index="9" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="9" port="0x18"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="10" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="10" port="0x19"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x1"/>
    </controller>
    <controller type="pci" index="11" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="11" port="0x1a"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x2"/>
    </controller>
    <controller type="pci" index="12" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="12" port="0x1b"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x3"/>
    </controller>
    <controller type="pci" index="13" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="13" port="0x1c"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x4"/>
    </controller>
    <controller type="pci" index="14" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="14" port="0x1d"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x5"/>
    </controller>
    <controller type="sata" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1f" function="0x2"/>
    </controller>
    <controller type="virtio-serial" index="0">
      <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
    </controller>
    <interface type="network">
      <mac address="52:54:00:60:ca:48"/>
      <source network="default"/>
      <model type="virtio"/>
      <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
    </interface>
    <serial type="pty">
      <target type="isa-serial" port="0">
        <model name="isa-serial"/>
      </target>
    </serial>
    <console type="pty">
      <target type="serial" port="0"/>
    </console>
    <channel type="spicevmc">
      <target type="virtio" name="com.redhat.spice.0"/>
      <address type="virtio-serial" controller="0" bus="0" port="1"/>
    </channel>
    <channel type="unix">
      <target type="virtio" name="org.qemu.guest_agent.0"/>
      <address type="virtio-serial" controller="0" bus="0" port="2"/>
    </channel>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <tpm model="tpm-crb">
      <backend type="emulator" version="2.0"/>
    </tpm>
    <graphics type="spice" autoport="yes">
      <listen type="address"/>
      <image compression="off"/>
    </graphics>
    <sound model="ich9">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1b" function="0x0"/>
    </sound>
    <audio id="1" type="spice"/>
    <video>
      <model type="qxl" ram="65536" vram="65536" vgamem="16384" heads="1" primary="yes"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
    </video>
    <redirdev bus="usb" type="spicevmc">
      <address type="usb" bus="0" port="1"/>
    </redirdev>
    <redirdev bus="usb" type="spicevmc">
      <address type="usb" bus="0" port="2"/>
    </redirdev>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>
    </memballoon>
  </devices>
</domain>
```


## Linux KVM Template

**kvm_xml_template_linux.xml**
```
<domain type="kvm">
  <name>${KVM_VM_NAME}</name>
  <memory unit="GiB">${KVM_VM_MEMORY}</memory>
  <currentMemory unit="GiB">${KVM_VM_MEMORY}</currentMemory>
  <vcpu placement="static">${KVM_VM_CPUS}</vcpu>
  <os>
    <type arch="x86_64" machine="pc-q35-7.2">hvm</type>
  </os>
  <features>
    <acpi/>
    <apic/>
    <vmport state="off"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on"/>
  <clock offset="utc">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" cache="none" discard="unmap"/>
      <source file="/var/lib/libvirt/images/${KVM_VM_NAME}.qcow2"/>
      <target dev="vda" bus="virtio"/>
      <boot order="1"/>
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </disk>
    <disk type="file" device="cdrom">
      <driver name="qemu" type="raw"/>
      <source file="${KVM_INSTALL_ISO}"/>
      <target dev="sda" bus="sata"/>
      <readonly/>
      <boot order="2"/>
      <address type="drive" controller="0" bus="0" target="0" unit="0"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15">
      <address type="pci" domain="0x0000" bus="0x02" slot="0x00" function="0x0"/>
    </controller>
    <controller type="pci" index="0" model="pcie-root"/>
    <controller type="pci" index="1" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="1" port="0x10"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="2" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="2" port="0x11"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x1"/>
    </controller>
    <controller type="pci" index="3" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="3" port="0x12"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x2"/>
    </controller>
    <controller type="pci" index="4" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="4" port="0x13"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x3"/>
    </controller>
    <controller type="pci" index="5" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="5" port="0x14"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x4"/>
    </controller>
    <controller type="pci" index="6" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="6" port="0x15"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x5"/>
    </controller>
    <controller type="pci" index="7" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="7" port="0x16"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x6"/>
    </controller>
    <controller type="pci" index="8" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="8" port="0x17"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x02" function="0x7"/>
    </controller>
    <controller type="pci" index="9" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="9" port="0x18"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x0" multifunction="on"/>
    </controller>
    <controller type="pci" index="10" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="10" port="0x19"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x1"/>
    </controller>
    <controller type="pci" index="11" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="11" port="0x1a"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x2"/>
    </controller>
    <controller type="pci" index="12" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="12" port="0x1b"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x3"/>
    </controller>
    <controller type="pci" index="13" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="13" port="0x1c"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x4"/>
    </controller>
    <controller type="pci" index="14" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="14" port="0x1d"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x5"/>
    </controller>
    <controller type="pci" index="15" model="pcie-root-port">
      <model name="pcie-root-port"/>
      <target chassis="15" port="0x1e"/>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x03" function="0x6"/>
    </controller>
    <controller type="pci" index="16" model="pcie-to-pci-bridge">
      <model name="pcie-pci-bridge"/>
      <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>
    </controller>
    <controller type="sata" index="0">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1f" function="0x2"/>
    </controller>
    <controller type="virtio-serial" index="0">
      <address type="pci" domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
    </controller>
    <interface type="network">
      <mac address="52:54:00:9f:27:5a"/>
      <source network="default"/>
      <model type="virtio"/>
      <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
    </interface>
    <serial type="pty">
      <target type="isa-serial" port="0">
        <model name="isa-serial"/>
      </target>
    </serial>
    <console type="pty">
      <target type="serial" port="0"/>
    </console>
    <channel type="unix">
      <target type="virtio" name="org.qemu.guest_agent.0"/>
      <address type="virtio-serial" controller="0" bus="0" port="1"/>
    </channel>
    <channel type="spicevmc">
      <target type="virtio" name="com.redhat.spice.0"/>
      <address type="virtio-serial" controller="0" bus="0" port="2"/>
    </channel>
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    <graphics type="spice" autoport="yes">
      <listen type="address"/>
      <image compression="off"/>
    </graphics>
    <graphics type="vnc" port="-1" autoport="yes" keymap="en-us">
      <listen type="address"/>
    </graphics>
    <sound model="ich9">
      <address type="pci" domain="0x0000" bus="0x00" slot="0x1b" function="0x0"/>
    </sound>
    <audio id="1" type="spice"/>
    <video>
      <model type="virtio" heads="1" primary="yes">
        <acceleration accel3d="no"/>
      </model>
      <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
    </video>
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>
    </memballoon>
    <rng model="virtio">
      <backend model="random">/dev/urandom</backend>
      <address type="pci" domain="0x0000" bus="0x06" slot="0x00" function="0x0"/>
    </rng>
  </devices>
</domain>

```
# KVM XML definition creation and launching of installation via script


- The script below is used to create an XML file from an XML template file that defines the KVM **domain** (the KVM virtual machine definition) and create the associated virtual hard disk
- The script can be used to either prompt for the information or code the information directly into the script (when using one method, the other method needs to be commented out)


## Windows KVM Template script

**kvm_template_vm_windows.sh**
```
#!/bin/bash

## Prompt for the information needed method
read -p 'Please enter the Virtual Machine Name (no spaces): ' KVM_VM_NAME
read -p 'Please enter the amount of Memory (how many GB): ' KVM_VM_MEMORY
read -p 'Please enter the number of CPUs: ' KVM_CPUS
read -p 'Please enter the Full Path to Installation ISO: ' KVM_INSTALL_ISO
read -p 'Please enter the Full Path to VirtIO driver ISO: ' KVM_VIRTIO_ISO
read -p 'Please enter the size of virtual hard disk (how many GB): ' KVM_HD_SIZE


## Hard code the information needed method
# KVM_VM_NAME="testing-robert-2"
# KVM_VM_MEMORY=16
# KVM_CPUS=4
# KVM_INSTALL_ISO="/home/rstrom/Downloads/SW_DVD9_Win_Pro_11_24H2_64BIT_English_Pro_Ent_EDU_N_MLF_X23-69812.ISO"
# KVM_VIRTIO_ISO="/home/rstrom/Downloads/virtio-win-0.1.271.iso"
# KVM_HD_SIZE="125"

rm -rf $KVM_VM_NAME.xml

sed "s|\${KVM_VM_NAME}|$KVM_VM_NAME|g" kvm_xml_template_windows.xml > $KVM_VM_NAME.xml 
sed -i "s|\${KVM_VM_MEMORY}|$KVM_VM_MEMORY|g" $KVM_VM_NAME.xml
sed -i "s|\${KVM_VM_CPUS}|$KVM_CPUS|g" $KVM_VM_NAME.xml 
sed -i "s|\${KVM_INSTALL_ISO}|$KVM_INSTALL_ISO|g" $KVM_VM_NAME.xml
sed -i "s|\${KVM_VIRTIO_ISO}|$KVM_VIRTIO_ISO|g" $KVM_VM_NAME.xml 

# Example command to create a new qcow2 virtual hard disk
# qemu-img create -f <type of disk to create> </path/to/harddisk/location <hard disk size>
# qemu-img create -f qcow2 /var/lib/libvirt/images/my-windws-virtual-machine.qcow2 125G
qemu-img create -f qcow2 /var/lib/libvirt/images/${KVM_VM_NAME}.qcow2 $KVM_HD_SIZE'G'

virsh define $KVM_VM_NAME.xml

virsh start $KVM_VM_NAME  && virt-viewer $KVM_VM_NAME &

rm $KVM_VM_NAME.xml
```


## Linux KVM Template script


```
#!/bin/bash

## Prompt for the information needed method
read -p 'Please enter the Virtual Machine Name (no spaces): ' KVM_VM_NAME
read -p 'Please enter the amount of Memory (how many GB): ' KVM_VM_MEMORY
read -p 'Please enter the number of CPUs: ' KVM_CPUS
read -p 'Please enter the Full Path to Installation ISO: ' KVM_INSTALL_ISO
# The Virtio drivers are not needed for Linux
# read -p 'Please enter the Full Path to VirtIO driver ISO: ' KVM_VIRTIO_ISO
read -p 'Please enter the size of virtual hard disk (how many GB): ' KVM_HD_SIZE


## Hard code the information needed method
# KVM_VM_NAME="testing-robert-2"
# KVM_VM_MEMORY=16
# KVM_CPUS=4
# KVM_INSTALL_ISO="/home/rstrom/Downloads/SW_DVD9_Win_Pro_11_24H2_64BIT_English_Pro_Ent_EDU_N_MLF_X23-69812.ISO"
# KVM_VIRTIO_ISO="/home/rstrom/Downloads/virtio-win-0.1.271.iso"
# KVM_HD_SIZE="125"

rm -rf $KVM_VM_NAME.xml

sed "s|\${KVM_VM_NAME}|$KVM_VM_NAME|g" kvm_xml_template_linux.xml > $KVM_VM_NAME.xml 
sed -i "s|\${KVM_VM_MEMORY}|$KVM_VM_MEMORY|g" $KVM_VM_NAME.xml
sed -i "s|\${KVM_VM_CPUS}|$KVM_CPUS|g" $KVM_VM_NAME.xml 
sed -i "s|\${KVM_INSTALL_ISO}|$KVM_INSTALL_ISO|g" $KVM_VM_NAME.xml
# The Virtio drivers are not needed for Linux
# sed -i "s|\${KVM_VIRTIO_ISO}|$KVM_VIRTIO_ISO|g" $KVM_VM_NAME.xml 

# Example command to create a new qcow2 virtual hard disk
# qemu-img create -f <type of disk to create> </path/to/harddisk/location <hard disk size>
# qemu-img create -f qcow2 /var/lib/libvirt/images/my-windws-virtual-machine.qcow2 125G
qemu-img create -f qcow2 /var/lib/libvirt/images/${KVM_VM_NAME}.qcow2 $KVM_HD_SIZE'G'

virsh define $KVM_VM_NAME.xml

virsh start $KVM_VM_NAME  && virt-viewer $KVM_VM_NAME &

rm $KVM_VM_NAME.xml
```
