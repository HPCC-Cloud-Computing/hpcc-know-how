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
    --name=vm-4 \
    --ram=512 \
    --vcpus=1 \
    --os-variant=ubuntu16.04 \
    --virt-type=kvm \
    --hvm \
    --cdrom=/home/nghiadt2/VMs/ubuntu-16.04.3-server-amd64.iso \
    --network=network=192.168.60.0,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --disk path=/home/nghiadt2/VMs/vm-4.qcow2,size=6,bus=virtio,format=qcow2
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
    time qemu-img create -f qcow2 /var/lib/libvirt/cinder_pool/cinder-vdb.qcow2 20000M -o preallocation=full
    ```
    1. Clone một volume
    ```sh
    virsh vol-clone --pool <pool_name> <vol_name> <vol_name_cloned>
    ```
    1. Xóa volume
    ```sh
    virsh vol-delete NAME --pool POOL
    ```

1. LVM
    * https://vinahost.vn/ac/knowledgebase/226/Cac-thao-tac-qun-ly-a-tren-LVM.html
    * https://www.cyberciti.biz/faq/how-to-add-disk-image-to-kvm-virtual-machine-with-virsh-command/






  sudo virt-install \
    --virt-type=kvm \
    --name controller \
    --ram 8192 \
    --vcpus=4 \
    --cpu host-model-only,force=vmx \
    --os-variant=ubuntu16.04 \
    --virt-type=kvm \
    --hvm \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --boot=hd \
    --disk path=ubuntu16.04-3.qcow2,bus=virtio,format=qcow2 


virt-install \
--virt-type=kvm \
--name cinder \
--ram 4096 \
--vcpus=4 \
--os-variant ubuntu16.04 \
--virt-type=kvm \
--hvm \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--boot=hd \
--disk path=/var/lib/libvirt/cinder_pool/cinder.qcow2,bus=virtio,format=qcow2


python -m nmt.nmt \
    --src=vi --tgt=en \
    --vocab_prefix=/root/nmt/nmt_model/nmt_data/vocab  \
    --train_prefix=/root/nmt/nmt_model/nmt_data/train \
    --dev_prefix=/root/nmt/nmt_model/nmt_data/tst2012  \
    --test_prefix=/root/nmt/nmt_model/nmt_data/tst2013 \
    --out_dir=/tmp/nmt_model \
    --num_train_steps=12000 \
    --steps_per_stats=100 \
    --num_layers=2 \
    --num_units=128 \
    --dropout=0.2 \
    --metrics=bleu



python -m nmt.nmt \
    --out_dir=/root/nmt/nmt_attention_model \
    --inference_input_file=/tmp/my_infer_file.en \
    --inference_output_file=/root/nmt/nmt_attention_model/output_infer.vi

virsh attach-disk cinder --source /var/lib/libvirt/cinder_pool/cinder-vdb.qcow2 --target vdb --persistent