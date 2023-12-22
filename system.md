# System Setup
## Desktop Hardware Spec
1. Intel Xeon Silver 4109T, 64 GB.
2. Nvidia Quadro P400.
3. Hard drive, 2TB.
4. Static IP address behind a proxy.

## Create a LiveUSB and prepare BIOS
1. Download Archlinux installation ISO file.
2. Download Rufus from https://rufus.ie or other equivalent.
3. Create a bootable LiveUSB with GPT partition and UEFI mode.
4. Disable secure boot in BIOS.
5. Insert LiveUSB to desktop and boot into Live Archlinux.

## Enable SSH access
```
passwd
sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/g' /etc/ssh/sshd_config
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's///g' /etc/ssh/sshd_config
sed -i 's///g' /etc/ssh/sshd_config
```

`vim /etc/ssh/sshd_config` - Update SSH configuration 

1. Uncomment `ListenAddress 0.0.0.0`  
2. Uncomment and change `PermitRootLogin yes`
3. Uncomment `PubkeyAuthentication yes`
4. Uncomment `PasswordAuthentication yes`

`systemctl restart sshd`: Restart SSH service to make above change effect.

`ip a`: Remember this IP, so that you can SSH to Live environment to continue installation.
