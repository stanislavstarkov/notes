# TLDR
# Main
# Troubleshooting
###### Fix exit code 43 error
Add `vendor_id` and `hidden state`
```xml
...
<features>
  <acpi/>
  <apic/>
  <hyperv>
    <relaxed state='on'/>
    <vapic state='on'/>
    <spinlocks state='on' retries='8191'/>

    <vendor_id state='on' value='1234567890ab'/>

  </hyperv>

  <kvm>
    <hidden state='on'/>
  </kvm>

  <vmport state='off'/>
  <ioapic driver='kvm'/>
</features>
...
```
###### Passthrough CPU architecture
> Add before windows installation
> Need to use cpu pinning as well - otherwise I get VIDEO TDR FAILURE error

```xml
<cpu mode='host-passthrough' check='none'>
<topology sockets='1' cores='4' threads='2'/>
<feature policy='require' name='topoext'/>
</cpu>
```
```xml
<vcpu placement='static'>8</vcpu>
<iothreads>2</iothreads>
<cputune>
  <vcpupin vcpu='0' cpuset='4'/>
  <vcpupin vcpu='1' cpuset='12'/>
  <vcpupin vcpu='2' cpuset='5'/>
  <vcpupin vcpu='3' cpuset='13'/>
  <vcpupin vcpu='4' cpuset='6'/>
  <vcpupin vcpu='5' cpuset='14'/>
  <vcpupin vcpu='6' cpuset='7'/>
  <vcpupin vcpu='7' cpuset='15'/>
  <emulatorpin cpuset='0-1'/>
  <iothreadpin iothread='1' cpuset='0-1'/>
  <iothreadpin iothread='2' cpuset='2-3'/>
</cputune>
```
###### Windows drivers VFIO
https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html
# Sources
- https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
- https://mathiashueber.com/windows-virtual-machine-gpu-passthrough-ubuntu/
