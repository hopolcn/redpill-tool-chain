### redpill-loader - create a proxmox vm from cli

```shell
# set vm id
id=930

# create image directory, download and uncomporess
mkdir -p /var/lib/vz/images/${id}
curl --location https://github.com/pocopico/tinycore-redpill/raw/main/tinycore-redpill.v0.4.4.img.gz --output /var/lib/vz/images/${id}/tinycore-redpill.img.gz
gzip --decompress /var/lib/vz/images/${id}/tinycore-redpill.img.gz --keep

# create vm
qm create ${id} \
  --args "-device 'qemu-xhci,addr=0x18' -drive 'id=synoboot,file=/var/lib/vz/images/${id}/tinycore-redpill.img,if=none,format=raw' -device 'usb-storage,drive=synoboot,bootindex=1'" \
  --cores 2 \
  --cpu host \
  --machine q35 \
  --memory 2048 \
  --name DSM \
  --net0 virtio,bridge=vmbr0 \
  --numa 0 \
  --onboot 0 \
  --ostype l26 \
  --scsihw virtio-scsi-pci \
  --sata0 local-lvm:vm-${id}-disk-0,discard=on,size=100G,ssd=1 \
  --sockets 1 \
  --serial0 socket \
  --serial1 socket \
  --tablet 1

# create disk for sata0
pvesm alloc local-lvm ${id} vm-${id}-disk-0 100G
```

### redpill-loader - proxmox user_config.json for rploader.sh

```
{
    "extra_cmdline": {
        "vid": "0x46f4",
        "pid": "0x0001",
        "sn": "1510LWN123456",
        "mac1": "00123456789A",
        "SasIdxMap": "0",
        "SataPortMap": "66",
        "DiskIdxMap": "0600"
    },
    "synoinfo": {
        "internalportcfg": "0xffff",
        "maxdisks": "16"
    },
    "ramdisk_copy": {}
}
```

### redpill-loader - bring your own configuration

```
1. in tc: open a terminal

2. in tc (in terminal): passwd  # to change the password 

3. in tc (in terminal): ifconfig  # to see the ip of tc

4. ssh host: scp custom_config.json tc@ip-of-tc:~/custom_config.json

5. ssh host: scp user_config.json tc@ip-of-tc:~/user_config.json

6. ssh host: ssh tc@ip-of-tc

7. ssh-host (while connected to tc): sudo ./rploader.sh update now # wasn't working without sudo for me.

8. ssh-host (while connected to tc): ./rploader.sh build bromolow-7.0.1-42218 static

9. ssh-host (while connected to tc): ./rploader.sh clean now # required, otherwise the next command will fail

10. ssh-host (while connected to tc): filetool.sh -b sdb3 # persist tc configuration back to tc partition. Your tc partition might be on a different disk.. change sdb to your device if needed.

11. ssh-host (while connected to tc): sudo reboot
```

