## KVM镜像制作

1. `qemu-img create -f qcow2 /home/shoppon/images/windows-10.qcow2 160G`
2. `virt-install --connect qemu:///system  --name windows-10 --ram 8192 --vcpus 8  --network network=default,model=virtio  --disk path=/home/shoppon/images/windows-10.qcow2,format=qcow2,device=disk,bus=virtio  --cdrom /home/shoppon/images/win10_1909.iso  --disk path=/home/shoppon/images/virtio-win-0.1.171.iso,device=cdrom  --vnc --os-type windows`
3. `qemu-img convert -c -O qcow2 /home/shoppon/images/windows-10.qcow2 /home/shoppon/images/win10-openstack.qcow2`
4. `glance image-create --name "windows10" --file /tmp/windows-openstack.qcow2 --disk-format qcow2 --container-format bare --visibility public`