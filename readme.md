# Installing ESX 6 / 7 / 8

Esx 7

## vCreate ISO + usb nic fling
## Install ESX 7
## Enable SSH / Autostart SSH

Enable SSH via html interface

After that, login to the server and permanently enable ssh.

```
[root@kuiper1:~] vim-cmd hostsvc/enable_ssh

[root@kuiper1:~] sed -i '/exit 0/d' /etc/rc.local.d/local.sh
[root@kuiper1:~] echo "vim-cmd hostsvc/enable_ssh" >> /etc/rc.local.d/local.sh && \
[root@kuiper1:~] echo "exit 0" >> /etc/rc.local.d/local.sh
[root@kuiper1:~] cat /etc/rc.local.d/local.sh 
```

### store SSH keys for autologin

ssh keys are stored in /etc/ssh/keys-[username]

so create a file authorized_keys and set the rights to rw - - and copy the id_key.pub of the other user.

```
[root@kuiper2:~] ls -l /etc/ssh/keys-root/authorized_keys 
-rw------T    1 root     root           393 Jan 18 11:50 /etc/ssh/keys-root/authorized_keys
[root@kuiper2:~]
```

## Set Hostname + DHCP hostname
### Set Hostname
```
[root@localhost:~] esxcli system hostname set --host=kuiper1
[root@kuiper1:~] esxcli system hostname set --fqdn=kuiper1.fritz.box
[root@kuiper1:~]
```

### DHCP Hostname
```
[root@kuiper2:~] echo "send host-name = \"`hostname -s`\";" >> /etc/dhclient-vmk0.conf
[root@kuiper2:~] cat /etc/dhclient-vmk0.conf 
#
# This file can be used to specify advanced dhclient options for vmk0
# (usually the management interface). Please refer to dhclient.conf(5)
# packaged with ISC DHCP 4.0.0 for available options.
#
send host-name = "kuiper2";
[root@kuiper2:~]
```

## Permanently enable USB-NIC

```
[root@kuiper2:~] esxcli system module parameters set -p "usbBusFullScanOnBootEnabled=1" -m vmkusb_nic_fling
[root@kuiper2:~] 
```

## Enable NTP / Autostart NTP

```
[root@localhost:~] esxcli system ntp set --server 1.nl.pool.org --server 2.nl.pool
.org --server 3.nl.pool.org --enabled=true
[root@localhost:~] esxcli system ntp get
   Enabled: true
   Loglevel: warning
   Servers: 1.nl.pool.org, 2.nl.pool.org, 3.nl.pool.org
[root@localhost:~]
```

## Set powersave

```
[root@kuiper3:~] vsish -e get /power/currentPolicy
Host power management policy {
   ID: 2
   Short name:dynamic
   Long name:Balanced
   Description:Reduce energy consumption with minimal performance compromise
}
[root@kuiper3:~] vsish -e get /power/hardwareSupport
Hardware power management support {
   CPU power management:ACPI P-states, ACPI C-states
}
[root@kuiper3:~]  vsish -e get /power/policy/1
Host power management policy {
   ID: 1
   Short name:static
   Long name:High Performance
   Description:Do not use any power management features
}
[root@kuiper3:~]  vsish -e get /power/policy/3
Host power management policy {
   ID: 3
   Short name:low
   Long name:Low Power
   Description:Reduce energy consumption at the risk of lower performance
}
[root@kuiper3:~] vsish -e set /power/currentPolicy 3
[root@kuiper3:~]
```

## Enter license

```
[root@kuiper3:~] vim-cmd vimsvc/license --set=xxxx-xxxx-xxxx-xxxx-xxxx

   serial: xxxx-xxxx-xxxx-xxxx-xxxx
   vmodl key: esx.hypervisor.cpuPackageCoreLimited
   name: vSphere 7 Hypervisor
   total: 0
   used: 1
   unit: cpuPackage:32core
   Properties:
     [ProductName] = VMware ESX Server
     [ProductVersion] = 7.0
     [count_disabled] = This license is unlimited
     [feature] = vsmp:8 ("Up to 8-way virtual SMP")
     [FileVersion] = 7.0.3.1
```

## Update
I'm installing 7.02. The update to 7.03k needs 3 steps.
update to 7.03

```
[root@kuiper1:~] esxcli system maintenanceMode set -e true
[root@kuiper1:~] vmware -l
VMware ESXi 7.0 Update 3
[root@kuiper1:~] vmware -v
VMware ESXi 7.0.3 build-20328353

[root@kuiper1:~] esxcli software vib install -d /vmfs/volumes/datastore1/update/ESXi-7.03g-20328353-USB.zip


[root@kuiper3:~] esxcli software vib install -d /vmfs/volumes/datastore1/update/ESXi-7.03g-20328353-USB.zip 
Installation Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: VMW_bootbank_atlantic_1.0.3.0-8vmw.703.0.20.19193900, VMW_bootbank_bnxtnet_216.0.50.0-44vmw.703.0.50.20036589, VMW_bootbank_bnxtroce_216.0.58.0-23vmw.703.0.50.20036589, VMW_bootbank_brcmfcoe_12.0.1500.2-3vmw.703.0.20.19193900, VMW_bootbank_elxiscsi_12.0.1200.0-9vmw.703.0.20.19193900, VMW_bootbank_elxnet_12.0.1250.0-5vmw.703.0.20.19193900, VMW_bootbank_i40en_1.11.1.31-1vmw.703.0.20.19193900, VMW_bootbank_iavmd_2.7.0.1157-2vmw.703.0.20.19193900, VMW_bootbank_icen_1.4.1.20-1vmw.703.0.50.20036589, VMW_bootbank_igbn_1.4.11.2-1vmw.703.0.20.19193900, VMW_bootbank_ionic-en_16.0.0-16vmw.703.0.20.19193900, VMW_bootbank_irdman_1.3.1.22-1vmw.703.0.50.20036589, VMW_bootbank_iser_1.1.0.1-1vmw.703.0.50.20036589, VMW_bootbank_ixgben_1.7.1.35-1vmw.703.0.20.19193900, VMW_bootbank_lpfc_14.0.169.26-5vmw.703.0.50.20036589, VMW_bootbank_lpnic_11.4.62.0-1vmw.703.0.20.19193900, VMW_bootbank_lsi-mr3_7.718.02.00-1vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt2_20.00.06.00-4vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt35_19.00.02.00-1vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt3_17.00.12.00-1vmw.703.0.20.19193900, VMW_bootbank_mtip32xx-native_3.9.8-1vmw.703.0.20.19193900, VMW_bootbank_ne1000_0.9.0-1vmw.703.0.50.20036589, VMW_bootbank_nenic_1.0.33.0-1vmw.703.0.20.19193900, VMW_bootbank_nfnic_4.0.0.70-1vmw.703.0.20.19193900, VMW_bootbank_nhpsa_70.0051.0.100-4vmw.703.0.20.19193900, VMW_bootbank_nmlx4-core_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx4-en_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx4-rdma_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx5-core_4.19.16.11-1vmw.703.0.20.19193900, VMW_bootbank_nmlx5-rdma_4.19.16.11-1vmw.703.0.20.19193900, VMW_bootbank_ntg3_4.1.7.0-0vmw.703.0.20.19193900, VMW_bootbank_nvme-pcie_1.2.3.16-1vmw.703.0.20.19193900, VMW_bootbank_nvmerdma_1.0.3.5-1vmw.703.0.20.19193900, VMW_bootbank_nvmetcp_1.0.0.1-1vmw.703.0.35.19482537, VMW_bootbank_nvmxnet3-ens_2.0.0.22-1vmw.703.0.20.19193900, VMW_bootbank_nvmxnet3_2.0.0.30-1vmw.703.0.20.19193900, VMW_bootbank_pvscsi_0.1-4vmw.703.0.20.19193900, VMW_bootbank_qcnic_1.0.15.0-14vmw.703.0.20.19193900, VMW_bootbank_qedentv_3.40.5.53-22vmw.703.0.20.19193900, VMW_bootbank_qedrntv_3.40.5.53-18vmw.703.0.20.19193900, VMW_bootbank_qfle3_1.0.67.0-22vmw.703.0.20.19193900, VMW_bootbank_qfle3f_1.0.51.0-22vmw.703.0.20.19193900, VMW_bootbank_qfle3i_1.0.15.0-15vmw.703.0.20.19193900, VMW_bootbank_qflge_1.1.0.11-1vmw.703.0.20.19193900, VMW_bootbank_rste_2.0.2.0088-7vmw.703.0.20.19193900, VMW_bootbank_sfvmk_2.4.0.2010-6vmw.703.0.20.19193900, VMW_bootbank_smartpqi_70.4149.0.5000-1vmw.703.0.20.19193900, VMW_bootbank_vmkata_0.1-1vmw.703.0.20.19193900, VMW_bootbank_vmkfcoe_1.0.0.2-1vmw.703.0.20.19193900, VMW_bootbank_vmkusb-nic-fling_1.10-1vmw.703.0.50.55634242, VMW_bootbank_vmkusb_0.1-7vmw.703.0.50.20036589, VMW_bootbank_vmw-ahci_2.0.11-1vmw.703.0.20.19193900, VMware_bootbank_bmcal_7.0.3-0.55.20328353, VMware_bootbank_cpu-microcode_7.0.3-0.55.20328353, VMware_bootbank_crx_7.0.3-0.55.20328353, VMware_bootbank_elx-esx-libelxima.so_12.0.1200.0-4vmw.703.0.20.19193900, VMware_bootbank_esx-base_7.0.3-0.55.20328353, VMware_bootbank_esx-dvfilter-generic-fastpath_7.0.3-0.55.20328353, VMware_bootbank_esx-ui_1.43.8-19798623, VMware_bootbank_esx-update_7.0.3-0.55.20328353, VMware_bootbank_esx-xserver_7.0.3-0.55.20328353, VMware_bootbank_esxio-combiner_7.0.3-0.55.20328353, VMware_bootbank_gc_7.0.3-0.55.20328353, VMware_bootbank_loadesx_7.0.3-0.55.20328353, VMware_bootbank_lsuv2-hpv2-hpsa-plugin_1.0.0-3vmw.703.0.20.19193900, VMware_bootbank_lsuv2-intelv2-nvme-vmd-plugin_2.7.2173-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-lsiv2-drivers-plugin_1.0.0-12vmw.703.0.50.20036589, VMware_bootbank_lsuv2-nvme-pcie-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-dell-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-hp-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-lenovo-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-smartpqiv2-plugin_1.0.0-8vmw.703.0.20.19193900, VMware_bootbank_native-misc-drivers_7.0.3-0.55.20328353, VMware_bootbank_qlnativefc_4.1.14.0-26vmw.703.0.20.19193900, VMware_bootbank_trx_7.0.3-0.55.20328353, VMware_bootbank_vdfs_7.0.3-0.55.20328353, VMware_bootbank_vmware-esx-esxcli-nvme-plugin_1.2.0.44-1vmw.703.0.20.19193900, VMware_bootbank_vsan_7.0.3-0.55.20328353, VMware_bootbank_vsanhealth_7.0.3-0.55.20328353, VMware_locker_tools-light_12.0.0.19345655-20036586
   VIBs Removed: VMW_bootbank_atlantic_1.0.3.0-8vmw.702.0.0.17867351, VMW_bootbank_bnxtnet_216.0.50.0-34vmw.702.0.20.18426014, VMW_bootbank_bnxtroce_216.0.58.0-20vmw.702.0.20.18426014, VMW_bootbank_brcmfcoe_12.0.1500.1-2vmw.702.0.0.17867351, VMW_bootbank_brcmnvmefc_12.8.298.1-1vmw.702.0.0.17867351, VMW_bootbank_elxiscsi_12.0.1200.0-8vmw.702.0.0.17867351, VMW_bootbank_elxnet_12.0.1250.0-5vmw.702.0.0.17867351, VMW_bootbank_i40enu_1.8.1.137-1vmw.702.0.20.18426014, VMW_bootbank_iavmd_2.0.0.1152-1vmw.702.0.0.17867351, VMW_bootbank_icen_1.0.0.10-1vmw.702.0.0.17867351, VMW_bootbank_igbn_1.4.11.2-1vmw.702.0.0.17867351, VMW_bootbank_irdman_1.3.1.19-1vmw.702.0.0.17867351, VMW_bootbank_iser_1.1.0.1-1vmw.702.0.0.17867351, VMW_bootbank_ixgben_1.7.1.35-1vmw.702.0.0.17867351, VMW_bootbank_lpfc_12.8.298.3-2vmw.702.0.20.18426014, VMW_bootbank_lpnic_11.4.62.0-1vmw.702.0.0.17867351, VMW_bootbank_lsi-mr3_7.716.03.00-1vmw.702.0.0.17867351, VMW_bootbank_lsi-msgpt2_20.00.06.00-3vmw.702.0.0.17867351, VMW_bootbank_lsi-msgpt35_17.00.02.00-1vmw.702.0.0.17867351, VMW_bootbank_lsi-msgpt3_17.00.10.00-2vmw.702.0.0.17867351, VMW_bootbank_mtip32xx-native_3.9.8-1vmw.702.0.0.17867351, VMW_bootbank_ne1000_0.8.4-11vmw.702.0.0.17867351, VMW_bootbank_nenic_1.0.33.0-1vmw.702.0.0.17867351, VMW_bootbank_nfnic_4.0.0.63-1vmw.702.0.0.17867351, VMW_bootbank_nhpsa_70.0051.0.100-2vmw.702.0.0.17867351, VMW_bootbank_nmlx4-core_3.19.16.8-2vmw.702.0.0.17867351, VMW_bootbank_nmlx4-en_3.19.16.8-2vmw.702.0.0.17867351, VMW_bootbank_nmlx4-rdma_3.19.16.8-2vmw.702.0.0.17867351, VMW_bootbank_nmlx5-core_4.19.16.10-1vmw.702.0.0.17867351, VMW_bootbank_nmlx5-rdma_4.19.16.10-1vmw.702.0.0.17867351, VMW_bootbank_ntg3_4.1.5.0-0vmw.702.0.0.17867351, VMW_bootbank_nvme-pcie_1.2.3.11-1vmw.702.0.0.17867351, VMW_bootbank_nvmerdma_1.0.2.1-1vmw.702.0.0.17867351, VMW_bootbank_nvmxnet3-ens_2.0.0.22-1vmw.702.0.0.17867351, VMW_bootbank_nvmxnet3_2.0.0.30-1vmw.702.0.0.17867351, VMW_bootbank_pvscsi_0.1-2vmw.702.0.0.17867351, VMW_bootbank_qcnic_1.0.15.0-11vmw.702.0.0.17867351, VMW_bootbank_qedentv_3.40.5.53-20vmw.702.0.20.18426014, VMW_bootbank_qedrntv_3.40.5.53-17vmw.702.0.20.18426014, VMW_bootbank_qfle3_1.0.67.0-14vmw.702.0.0.17867351, VMW_bootbank_qfle3f_1.0.51.0-19vmw.702.0.0.17867351, VMW_bootbank_qfle3i_1.0.15.0-12vmw.702.0.0.17867351, VMW_bootbank_qflge_1.1.0.11-1vmw.702.0.0.17867351, VMW_bootbank_rste_2.0.2.0088-7vmw.702.0.0.17867351, VMW_bootbank_sfvmk_2.4.0.2010-4vmw.702.0.0.17867351, VMW_bootbank_smartpqi_70.4000.0.100-6vmw.702.0.0.17867351, VMW_bootbank_vmkata_0.1-1vmw.702.0.0.17867351, VMW_bootbank_vmkfcoe_1.0.0.2-1vmw.702.0.0.17867351, VMW_bootbank_vmkusb-nic-fling_1.8-3vmw.702.0.20.47140841, VMW_bootbank_vmkusb_0.1-4vmw.702.0.20.18426014, VMW_bootbank_vmw-ahci_2.0.9-1vmw.702.0.0.17867351, VMware_bootbank_clusterstore_7.0.2-0.30.19290878, VMware_bootbank_cpu-microcode_7.0.2-0.30.19290878, VMware_bootbank_crx_7.0.2-0.30.19290878, VMware_bootbank_elx-esx-libelxima.so_12.0.1200.0-4vmw.702.0.0.17867351, VMware_bootbank_esx-base_7.0.2-0.30.19290878, VMware_bootbank_esx-dvfilter-generic-fastpath_7.0.2-0.30.19290878, VMware_bootbank_esx-ui_1.34.8-17417756, VMware_bootbank_esx-update_7.0.2-0.30.19290878, VMware_bootbank_esx-xserver_7.0.2-0.30.19290878, VMware_bootbank_gc_7.0.2-0.30.19290878, VMware_bootbank_loadesx_7.0.2-0.30.19290878, VMware_bootbank_lsuv2-hpv2-hpsa-plugin_1.0.0-3vmw.702.0.0.17867351, VMware_bootbank_lsuv2-intelv2-nvme-vmd-plugin_2.0.0-2vmw.702.0.0.17867351, VMware_bootbank_lsuv2-lsiv2-drivers-plugin_1.0.0-5vmw.702.0.0.17867351, VMware_bootbank_lsuv2-nvme-pcie-plugin_1.0.0-1vmw.702.0.0.17867351, VMware_bootbank_lsuv2-oem-dell-plugin_1.0.0-1vmw.702.0.0.17867351, VMware_bootbank_lsuv2-oem-hp-plugin_1.0.0-1vmw.702.0.0.17867351, VMware_bootbank_lsuv2-oem-lenovo-plugin_1.0.0-1vmw.702.0.0.17867351, VMware_bootbank_lsuv2-smartpqiv2-plugin_1.0.0-6vmw.702.0.0.17867351, VMware_bootbank_native-misc-drivers_7.0.2-0.30.19290878, VMware_bootbank_qlnativefc_4.1.14.0-5vmw.702.0.0.17867351, VMware_bootbank_vdfs_7.0.2-0.30.19290878, VMware_bootbank_vmware-esx-esxcli-nvme-plugin_1.2.0.42-1vmw.702.0.0.17867351, VMware_bootbank_vsan_7.0.2-0.30.19290878, VMware_bootbank_vsanhealth_7.0.2-0.30.19290878, VMware_locker_tools-light_11.2.6.17901274-18295176
   VIBs Skipped: 
[root@kuiper3:~] 
```

After that, update to 7,03k succeeded

```
[root@kuiper3:~] esxcli software vib update -d /vmfs/volumes/datastore1/update/VMware-ESXi-7.0U3k-21313628-depot.zip
Installation Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: VMW_bootbank_ntg3_4.1.8.0-4vmw.703.0.65.20842708, VMware_bootbank_bmcal_7.0.3-0.75.21313628, VMware_bootbank_cpu-microcode_7.0.3-0.75.21313628, VMware_bootbank_crx_7.0.3-0.75.21313628, VMware_bootbank_esx-base_7.0.3-0.75.21313628, VMware_bootbank_esx-dvfilter-generic-fastpath_7.0.3-0.75.21313628, VMware_bootbank_esx-ui_2.1.1-20188605, VMware_bootbank_esx-update_7.0.3-0.75.21313628, VMware_bootbank_esx-xserver_7.0.3-0.75.21313628, VMware_bootbank_esxio-combiner_7.0.3-0.75.21313628, VMware_bootbank_gc_7.0.3-0.75.21313628, VMware_bootbank_loadesx_7.0.3-0.75.21313628, VMware_bootbank_native-misc-drivers_7.0.3-0.75.21313628, VMware_bootbank_trx_7.0.3-0.75.21313628, VMware_bootbank_vdfs_7.0.3-0.75.21313628, VMware_bootbank_vsan_7.0.3-0.75.21313628, VMware_bootbank_vsanhealth_7.0.3-0.75.21313628, VMware_locker_tools-light_12.1.0.20219665-20841705
   VIBs Removed: VMW_bootbank_ntg3_4.1.7.0-0vmw.703.0.20.19193900, VMware_bootbank_bmcal_7.0.3-0.55.20328353, VMware_bootbank_cpu-microcode_7.0.3-0.55.20328353, VMware_bootbank_crx_7.0.3-0.55.20328353, VMware_bootbank_esx-base_7.0.3-0.55.20328353, VMware_bootbank_esx-dvfilter-generic-fastpath_7.0.3-0.55.20328353, VMware_bootbank_esx-ui_1.43.8-19798623, VMware_bootbank_esx-update_7.0.3-0.55.20328353, VMware_bootbank_esx-xserver_7.0.3-0.55.20328353, VMware_bootbank_esxio-combiner_7.0.3-0.55.20328353, VMware_bootbank_gc_7.0.3-0.55.20328353, VMware_bootbank_loadesx_7.0.3-0.55.20328353, VMware_bootbank_native-misc-drivers_7.0.3-0.55.20328353, VMware_bootbank_trx_7.0.3-0.55.20328353, VMware_bootbank_vdfs_7.0.3-0.55.20328353, VMware_bootbank_vsan_7.0.3-0.55.20328353, VMware_bootbank_vsanhealth_7.0.3-0.55.20328353, VMware_locker_tools-light_12.0.0.19345655-20036586
   VIBs Skipped: VMW_bootbank_atlantic_1.0.3.0-8vmw.703.0.20.19193900, VMW_bootbank_bnxtnet_216.0.50.0-44vmw.703.0.50.20036589, VMW_bootbank_bnxtroce_216.0.58.0-23vmw.703.0.50.20036589, VMW_bootbank_brcmfcoe_12.0.1500.2-3vmw.703.0.20.19193900, VMW_bootbank_elxiscsi_12.0.1200.0-9vmw.703.0.20.19193900, VMW_bootbank_elxnet_12.0.1250.0-5vmw.703.0.20.19193900, VMW_bootbank_i40en_1.11.1.31-1vmw.703.0.20.19193900, VMW_bootbank_iavmd_2.7.0.1157-2vmw.703.0.20.19193900, VMW_bootbank_icen_1.4.1.20-1vmw.703.0.50.20036589, VMW_bootbank_igbn_1.4.11.2-1vmw.703.0.20.19193900, VMW_bootbank_ionic-en_16.0.0-16vmw.703.0.20.19193900, VMW_bootbank_irdman_1.3.1.22-1vmw.703.0.50.20036589, VMW_bootbank_iser_1.1.0.1-1vmw.703.0.50.20036589, VMW_bootbank_ixgben_1.7.1.35-1vmw.703.0.20.19193900, VMW_bootbank_lpfc_14.0.169.26-5vmw.703.0.50.20036589, VMW_bootbank_lpnic_11.4.62.0-1vmw.703.0.20.19193900, VMW_bootbank_lsi-mr3_7.718.02.00-1vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt2_20.00.06.00-4vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt35_19.00.02.00-1vmw.703.0.20.19193900, VMW_bootbank_lsi-msgpt3_17.00.12.00-1vmw.703.0.20.19193900, VMW_bootbank_mtip32xx-native_3.9.8-1vmw.703.0.20.19193900, VMW_bootbank_ne1000_0.9.0-1vmw.703.0.50.20036589, VMW_bootbank_nenic_1.0.33.0-1vmw.703.0.20.19193900, VMW_bootbank_nfnic_4.0.0.70-1vmw.703.0.20.19193900, VMW_bootbank_nhpsa_70.0051.0.100-4vmw.703.0.20.19193900, VMW_bootbank_nmlx4-core_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx4-en_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx4-rdma_3.19.16.8-2vmw.703.0.20.19193900, VMW_bootbank_nmlx5-core_4.19.16.11-1vmw.703.0.20.19193900, VMW_bootbank_nmlx5-rdma_4.19.16.11-1vmw.703.0.20.19193900, VMW_bootbank_nvme-pcie_1.2.3.16-1vmw.703.0.20.19193900, VMW_bootbank_nvmerdma_1.0.3.5-1vmw.703.0.20.19193900, VMW_bootbank_nvmetcp_1.0.0.1-1vmw.703.0.35.19482537, VMW_bootbank_nvmxnet3-ens_2.0.0.22-1vmw.703.0.20.19193900, VMW_bootbank_nvmxnet3_2.0.0.30-1vmw.703.0.20.19193900, VMW_bootbank_pvscsi_0.1-4vmw.703.0.20.19193900, VMW_bootbank_qcnic_1.0.15.0-14vmw.703.0.20.19193900, VMW_bootbank_qedentv_3.40.5.53-22vmw.703.0.20.19193900, VMW_bootbank_qedrntv_3.40.5.53-18vmw.703.0.20.19193900, VMW_bootbank_qfle3_1.0.67.0-22vmw.703.0.20.19193900, VMW_bootbank_qfle3f_1.0.51.0-22vmw.703.0.20.19193900, VMW_bootbank_qfle3i_1.0.15.0-15vmw.703.0.20.19193900, VMW_bootbank_qflge_1.1.0.11-1vmw.703.0.20.19193900, VMW_bootbank_rste_2.0.2.0088-7vmw.703.0.20.19193900, VMW_bootbank_sfvmk_2.4.0.2010-6vmw.703.0.20.19193900, VMW_bootbank_smartpqi_70.4149.0.5000-1vmw.703.0.20.19193900, VMW_bootbank_vmkata_0.1-1vmw.703.0.20.19193900, VMW_bootbank_vmkfcoe_1.0.0.2-1vmw.703.0.20.19193900, VMW_bootbank_vmkusb_0.1-7vmw.703.0.50.20036589, VMW_bootbank_vmw-ahci_2.0.11-1vmw.703.0.20.19193900, VMware_bootbank_elx-esx-libelxima.so_12.0.1200.0-4vmw.703.0.20.19193900, VMware_bootbank_lsuv2-hpv2-hpsa-plugin_1.0.0-3vmw.703.0.20.19193900, VMware_bootbank_lsuv2-intelv2-nvme-vmd-plugin_2.7.2173-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-lsiv2-drivers-plugin_1.0.0-12vmw.703.0.50.20036589, VMware_bootbank_lsuv2-nvme-pcie-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-dell-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-hp-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-oem-lenovo-plugin_1.0.0-1vmw.703.0.20.19193900, VMware_bootbank_lsuv2-smartpqiv2-plugin_1.0.0-8vmw.703.0.20.19193900, VMware_bootbank_qlnativefc_4.1.14.0-26vmw.703.0.20.19193900, VMware_bootbank_vmware-esx-esxcli-nvme-plugin_1.2.0.44-1vmw.703.0.20.19193900
[root@kuiper3:~]

[root@kuiper1:~] esxcli system maintenanceMode get
Enabled
[root@kuiper1:~] esxcli system maintenanceMode set -e false
[root@kuiper1:~]
```

## NFS mount diskstation

This command mounts the NAS file system and adds it to the list of known file systems. 

```
[root@kuiper2:~] esxcli storage nfs add --host=192.168.1.22 --share=/volume1/vmnfsbackup --volume-name=nfsbackup 
 192.168.1.22:/volume1/vmnfsbackup
[root@kuiper2:~] esxcli storage nfs list
Volume Name  Host          Share                 Accessible  Mounted  Read-Only   isPE  Hardware Acceleration
-----------  ------------  --------------------  ----------  -------  ---------  -----  ---------------------
nfsbackup    192.168.1.22  /volume1/vmnfsbackup        true     true      false  false  Not Supported
[root@kuiper2:~]
```

## Backup VM

```
rm /vmfs/volumes/nfsbackup/uranus3/*
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/snapshot.create 2 "snapshot_backup" 0 1
cp /vmfs/volumes/datastore1/uranus3/* /vmfs/volumes/nfsbackup/uranus3/
vim-cmd vmsvc/snapshot.removeall 2
```
