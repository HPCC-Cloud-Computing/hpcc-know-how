# KVM

KVM là một nền tảng ảo hóa trong Ubuntu được sử dụng trong các hệ thống _non-graphic_, bên cạnh KVM, Ubuntu còn sử dụng Libvirt front ends để quản lý, thao tác với các VMs bao gồm virt-manager (dùng cho GUI) _virt-manager (dùng cho GUI)_ và _virsh (dùng cho CLI)_

## Cài đặt KVM

1. Để cài đặt KVM, ta cần các processors phần cứng hỗ trợ cho KVM. Để kiểm tra phần cứng của bạn có hỗ trợ KVM không, ta nhập vào terminal dòng lệnh sau:
    ```sh
    egrep -c '(vmx|svm)' /prop/cpuinfo 
    ```
    * Kết quả trả về 0: CPU của bạn không support KVM
    * Kết quả trả về >= 1: done!  
* Một cách khác, bạn có thể chạy dòng lệnh
    ```sh
        kvm-ok
    ```
    * Kết quả trả về thành công:
    ```sh
    INFO: /dev/kvm exists
    KVM acceleration can be used
    ```
    * Nếu trả về: 
    ```sh
    INFO: Your CPU does not support KVM extensions
    KVM acceleration can NOT be used 
    ```
    Bạn vẫn có thể chạy VMs nhưng sẽ không sử dụng được các extensions của KVM

1. Cài đặt
    * Ubuntu 10.04 (Lucid) trở lên:
    ```sh 
    $ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
    ```
    * Ubuntu 9.10 (Karmic) trở xuống:
    ```sh
    $ sudo aptitude install kvm libvirt-bin ubuntu-vm-builder bridge-utils
    ```
    1. libvirt-bin: cần cho việc quản lý các qemu và kvm instances
    1. qemu-kvm (kvm)：backen
    1. ubuntu-vm-builder: command line tools để tạo các VMs
    1. bridge-utils: tạo các bridges cho các VMs

1. Copy file Host - VMs
    * Host to VMs
    ```sh
    virt-copy-in -a disk.img file|dir [file|dir ...] /destination
    virt-copy-in -d domain file|dir [file|dir ...] /destination
    ``` 
    * VMs to Host
    ```sh
    virt-copy-out -a disk.img /file|dir [/file|dir ...] localdir
    virt-copy-out -d domain /file|dir [/file|dir ...] localdir
    ```

1. Tạo máy ảo
    ```sh
    virt-install \
    --virt-type=kvm \
    --name=thao-ubuntu \
    --ram=2048 \
    --vcpus=4 \
    --os-variant=ubuntu16.04 \
    --virt-type=kvm \
    --hvm \
    --cdrom=/home/bkcloud13/ubuntu-16.04.2-server-amd64.iso \
    --network=bridge=br-vm-ex,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --disk path=/var/lib/libvirt/thao-ubuntu.qcow2,size=10,bus=virtio,format=qcow2
    ```
1. Clone may ao
    1. Shutdown máy ảo cần clone:
    ```sh
    Shutdown máy ảo
        virsh shutdown <ten_may_ao>
    Kiểm tra trạng thái máy ảo vừa shutdown
        virsh list --all
    ``` 
    1. Clone
    ```sh
    virsh vol-clone --pool <pool_name> <vol_name> <vol_name_cloned>
    ```

    1. Tạo một VM từ volume vừa clone: Xóa bỏ option cdrom và thêm option `--boot=hd \`
    ```sh
    sudo virt-install \
    --virt-type=kvm \
    --name <vm_name> \
    --ram 4096 \
    --vcpus=4 \
    --cpu host-model-only,force=vmx \
    --os-variant <os_version> \
    --virt-type=kvm \
    --hvm \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --boot=hd \
    --disk path=<path_to_cloned_volume>,bus=virtio,format=qcow2 
    ``` 

    1. Một số  lệnh support 
        * Hiển thị danh sách pool
        ```sh
        virsh pool-list --all
        ```
        * Hiển thị các volume trong một pool
        ```sh
        virsh vol-list <pool_name>
        ```

1. Xóa máy ảo
    ```sh
    Tắt máy ảo nếu, như máy ảo vẫn đang chạy
        virsh destroy <VM>
    Xóa máy ảo khỏi định nghĩa của libvirt
        virsh undefine <VM>
    Xóa bỏ volume của máy ảo
        virsh pool-list
        virsh vol-list <pool_name>
        virsh vol-delete --pool <pool_name> <vol_name>        
    ```

1. Tạo network
    1. Xoa network
    ```sh
    virsh net-destroy <network>
    virsh net-undefine <network>
    ```
    
1. Tạo pool storage và volumn storage
    1. Pool storage dir 
        1. Tạo file XML
        ``` sh
        <pool type="dir">
        <name>VMs</name>
        <target>
            <path>/home/nghia/Documents/VMs</path>
        </target>
        </pool>
        ``` 
        1. Định nghĩa pool và kiểm tra trạng thái
        ```sh
        virsh pool-define VMs_pool.xml
        virsh pool-list --all
        ```
        1. Start và auto start pool và kiểm tra trạng thái
        ```sh
        virsh pool-start VMs
        virsh pool-autostart VMs
        virsh pool-list --all
        ```
        1. Xem thông tin về  pool 
        ```sh
        virsh pool-info VMs
        ```

    1. Tạo một volumn trong pool storage dir vừa tạo
    ```sh
    virsh vol-create-as POOLNAME VOLNAME 12G --format raw|qcow2|qed

    Full allocation
    time qemu-img create -f qcow2 /var/lib/libvirt/controller15/cinder-vdb.qcow2 30000M -o preallocation=full
    ```
    1. Clone một volume
    ```sh
    virsh vol-clone --pool <pool_name> <vol_name> <vol_name_cloned>
    ```
    1. Xóa volume
    ```sh
    virsh vol-delete NAME --pool POOL
    ```
    1. Attach disk
    ```sh
    virsh attach-disk cinder --source /var/lib/libvirt/cinder_pool/cinder-vdb.qcow2 --target vdb --persistent
    ```
    1. Detach disk
    ```sh
    virsh detach-disk cinder vdb --persistent
    ```
    1. Resize disk
    * Sau đây là các bước giúp bạn tăng ổ đĩa cứng cho VMs
    * Bước 1: Shutdown VM
    * Bước 2: Tăng phần cứng vật lý: Ví dụ bạn muốn tăng 20G cho ổ đĩa 
    ```sh
    qemu-img resize <path_to_disk> +20G
    hoặc: virsh vol-resize <capacity> <vol> --pool <pool_name>
    ```
    * Bước 3: Start VM
    * Bước 4:
    ```sh
    % fdisk /dev/vda
    ...
    Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *           1          13      104391   83  Linux
    /dev/vda2              14        3263    26105625   8e  Linux LVM

    Command (m for help): d
    Partition number (1-4): 2

    Command (m for help): p

    Disk /dev/vda: 48.3 GB, 48318382080 bytes
    255 heads, 63 sectors/track, 5874 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

    Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *           1          13      104391   83  Linux

    Command (m for help): n 
    Command action
    e   extended
    p   primary partition (1-4)
    p
    Partition number (1-4): 2
    First cylinder (14-5874, default 14): 14
    Last cylinder or +size or +sizeM or +sizeK (14-5874, default 5874): 
    Using default value 5874

    Command (m for help): p

    Disk /dev/vda: 48.3 GB, 48318382080 bytes
    255 heads, 63 sectors/track, 5874 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

    Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *           1          13      104391   83  Linux
    /dev/vda2              14        5874    47078482+  83  Linux

    Command (m for help): t
    Partition number (1-4): 2
    Hex code (type L to list codes): 8e
    Changed system type of partition 2 to 8e (Linux LVM)

    Command (m for help): p

    Disk /dev/vda: 48.3 GB, 48318382080 bytes
    255 heads, 63 sectors/track, 5874 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

    Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *           1          13      104391   83  Linux
    /dev/vda2              14        5874    47078482+  8e  Linux LVM

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.

    WARNING: Re-reading the partition table failed with error 16: Device or 
    resource busy.
    The kernel still uses the old table.
    The new table will be used at the next reboot.
    Syncing disks.
    %
    ```
    Bước 5: Reboot VM
    Bước 6: Resize Phisical Volume của LVM
    ```sh
    % pvdisplay 
    --- Physical volume ---
    PV Name               /dev/vda2
    VG Name               VolGroup00
    PV Size               24.90 GB / not usable 21.59 MB
    Allocatable           yes (but full)
    PE Size (KByte)       32768
    Total PE              796
    Free PE               0
    ...

    % pvresize /dev/vda2

    % pvdisplay
    --- Physical volume ---
    PV Name               /dev/vda2
    VG Name               VolGroup00
    PV Size               44.90 GB / not usable 22.89 MB
    Allocatable           yes 
    PE Size (KByte)       32768
    Total PE              1436
    Free PE               640
    ...
    ```
    Bước 7: Resize Logical Volume
    ```sh
     % lvresize /dev/VolGroup00/LogVol00 -l +640
    Extending logical volume LogVol00 to 43.88 GB
    Logical volume LogVol00 successfully resized
    ```
    Bước 8: Cập nhật File Systems
    ```sh
    % resize2fs /dev/VolGroup00/LogVol00 
    resize2fs 1.39 
    Filesystem at /dev/VolGroup00/LogVol00 is mounted on /; on-line resizing required
    Performing an on-line resize of /dev/VolGroup00/LogVol00 to 11501568 (4k) blocks.
    The filesystem on /dev/VolGroup00/LogVol00 is now 11501568 blocks long.
    ```

### Tham Khảo

* [Resize ổ cứng sử dụng LVM](    https://serverfault.com/questions/324281/how-do-you-increase-a-kvm-guests-disk-space/514169)
* [Resize ổ cứng LVM & virt-resize](http://dnaeon.github.io/resizing-a-kvm-disk-image-on-lvm-the-easy-way/)
* https://github.com/HPCC-Cloud-Computing/hpcc-know-how/blob/master/SDN/openvswitch-kvm-note.md