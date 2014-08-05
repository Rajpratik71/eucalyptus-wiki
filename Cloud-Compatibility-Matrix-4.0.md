This Compatibility Matrix details supported platforms in Eucalyptus 4.0 software.<br><br>For a list of other versions [click here](Eucalyptus-Cloud-Compatibility-Matrix---Overview).

## Compute Compatibility
<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Operating System and Hypervisors</th>
<th align="center">Version</th>
<th align="center">Architecture</th>
</tr>
</thead>
<tbody>
<tr class="even">
<td align="left">CentOS+KVM</td>
<td align="center">6.5</td>
<td align="center">x86_64</td>
</tr>
<tr class="odd">
<td align="left">RHEL+KVM</td>
<td align="center">6.5</td>
<td align="center">x86_64</td>
</tr>
<tr class="even">
<td align="left">VMware ESXi</td>
<td align="center">5.1</td>
<td align="center">x86_64</td>
</tr>
<tr class="odd">
<td align="left">VMware vCenter</td>
<td align="center">5.1</td>
<td align="center">x86_64</td>
</tr>
</tbody>
</table>
## Guest Operating Systems

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Operating System Type</th>
<th align="center">Version/Edition</th>
<th align="center">Architecture</th>
</tr>
</thead>
<tbody>
<tr class="even">
<td align="left">Windows Server 2003 R2</td>
<td align="center">Enterprise</td>
<td align="center">i386/x86_64</td>
</tr>
<tr class="odd">
<td align="left">Windows Server 2008</td>
<td align="center">Datacenter</td>
<td align="center">i386/x86_64</td>
</tr>
<tr class="even">
<td align="left">Windows Server 2008 R2</td>
<td align="center">Datacenter</td>
<td align="center">i386/x86_64</td>
</tr>
<tr class="odd">
<td align="left">Windows Server 2012</td>
<td align="center">Datacenter</td>
<td align="center">x86_64</td>
</tr>
<tr class="odd">
<td align="left">All Modern Linux Distributions</td>
<td align="center"> RedHat, CentOS, Ubuntu, Fedora, Debian, SLES, OpenSUSE, etc. </td>
<td align="center">i386/x86_64</td>
</tr>
</tbody>
</table>

## Network Modes
<table>
<colgroup>
<col width="25%" />
<col width="75%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Networking Mode</th>
<th align="left">Compatibility Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Edge</td>
<td align="left"> Layer 2 and 3 VM isolation; uses built-in DHCP service<br><br>Requires that any external DHCP service be isolated from Eucalyptus VMs</td>
</tr>
<tr class="even">
<td align="left">Managed</td>
<td align="left"> Layer 2 and 3 VM isolation; uses built-in DHCP service<br><br>Requires a cloud switch to forward a configurable range of VLAN-tagged packets, and requires that any external DHCP service be isolated from Eucalyptus VMs</td>
</tr>
<tr class="odd">
<td align="left">Managed (No VLAN)</td>
<td align="left"> Layer 3 VM isolation; uses built-in DHCP service and requires that any external DHCP service be isolated from Eucalyptus VMs. </td>
</tr>
</tbody>
</table>

## Object Storage Compatibility

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Storage Array</th>
<th align="left">Type and Series</th>
<th align="left">Version</th>
<th align="left">High Availability Support</th>
<th align="left">Notes</th>
</thead>
<tbody>
</tr>
<tr class="even">
<td align="left"> RiakCS</td>
<td align="center"> Open Source<br>Enterprise</td>
<td align="center"> 1.4.5 </td>
<td align="center"> Yes </td>
<td align="center"> </td>
</tr>
</table>

## Elastic Block Storage Compatibility

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">Storage Array</th>
<th align="left">Type and Series</th>
<th align="left">Version</th>
<th align="left">High Availability Support</th>
<th align="left">Notes</th>
</thead>
<tbody>
</tr>
<tr class="even">
<td align="left"> RiakCS</td>
<td align="center"> Open Source<br>Enterprise</td>
<td align="center"> 1.4.5 </td>
<td align="center"> Yes </td>
<td align="center"> </td>
</tr>
<tr class="odd">
<td align="left"> NetApp</td>
<td align="center"> FAS 2000 series<br>FAS 3000 series<br>FAS 6000 series</td>
<td align="center"> OnTAPi 7.3.3 or higher, 7-mode or C-mode </td>
<td align="center"> Yes </td>
<td align="center"> FlexClone and iSCSI required </td>
</tr>
<tr class="even">
<td align="left"> EqualLogic</td>
<td align="center"> PS 4000 series<br>PS 6000 series </td>
<td align="center"> Compatible CLI versions </td>
<td align="center"> Yes </td>
<td align="center"> </td>
</tr>
<tr class="odd">
<td align="left"> EMC </td>
<td align="center"> VNX series</td>
<td align="center"> FLARE v5.32</td>
<td align="center"> Yes</td>
<td align="center"> Navisphere CLI 7.32.0.5.54 or higher required</td>
</tr>
<tr class="even">
<td align="left"> Direct Attached Storage </td>
<td align="center"> Any JBOD </td>
<td align="center"> </td>
<td align="center"> No </td>
<td align="center"> Linux SCSI Target Framework is required </td>
</tr>
<tr class="odd">
<td align="left"> Local Disks</td>
<td align="center"> N/A </td>
<td align="center"> N/A </td>
<td align="center"> No </td>
<td align="center"> Linux SCSI Target Framework is required </td>
</tr>
<tr class="even">
<td align="left"> Remote Disk</td>
<td align="center"> N/A </td>
<td align="center"> N/A </td>
<td align="center"> </td>
<td align="center"> Linux SCSI Target Framework is required </td>
</tr>
</tbody>
</table>

## Minimum Hardware Requirements
* **Front End Components** <br> Cloud Controller, Walrus, Storage Controller, Cluster Controller, VMware Broker<br><br>8 Cores, 2.4GHz, 8GB RAM, 500GB Disk space. (multiple disks in a RAID 5/10 recommended)<br><br> High Availability (HA) configurations must be equal in capacity.<br><br>
* **Node Controllers** <br>4 Cores, 2.4GHz, 8GB RAM, 160GB Disks (multiple disks recommended).<br><br> Note: Hardware virtualization must be enabled in the BIOS.