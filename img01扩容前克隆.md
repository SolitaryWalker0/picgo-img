### 1、先克隆出三台样板机节点

（样板机只是自己定义，新装的那也算，不过我自己是安装了一些常用命令，避免以后每次安装都要安装这些常用命令）

### 2、修改.vmx文件

将文件中scsi0:0.fileName这一项修改为你准备设置的主机名（之前的都是样板机的主机名）

![image-20260103075120414](C:\Users\lxm\AppData\Roaming\Typora\typora-user-images\image-20260103075120414.png)

另外在文件末尾添加两项配置

```conf
vhv.enable = "TRUE"
hypervisor.cpuid.v0 = "FALSE"
```

以下是这两项配置的具体作用：

1. `vhv.enable = "TRUE"`

- **全称**：Virtualize Hardware Virtualization（虚拟化硬件虚拟化）。

- **作用**：这是开启**嵌套虚拟化**的关键开关。

- 原理

  ：

  - 当你在 VMware 里运行一个 CentOS 虚拟机时，默认情况下，VMware 会“拦截”底层物理 CPU 的虚拟化指令（Intel VT-x / AMD-V）。
  - 设置为 `TRUE` 后，VMware 会将物理 CPU 的虚拟化特性**透传**给虚拟机。这样，你在 CentOS 里运行 K3s 或 Docker 时，它就能直接调用硬件虚拟化指令，性能更好，且不会报错。

- **如果不开会怎样**：你在虚拟机里运行 `grep vmx /proc/cpuinfo` 可能看不到结果，或者运行容器时提示“无法打开设备 /dev/kvm”。

2. `hypervisor.cpuid.v0 = "FALSE"`

- **作用**：隐藏 VMware 的“管理程序”身份。

- 原理

  ：

  - 默认情况下，VMware 会在 CPU 的 CPUID 信息中留下标记（Signature），告诉操作系统“你正在虚拟机里运行”。
  - 有些软件（例如某些版本的 Docker Desktop、特定的反作弊系统、或部分反病毒软件）检测到这个标记后，会拒绝运行，或者降低性能（例如提示“在虚拟机中运行不安全”）。
  - 设置为 `FALSE` 后，VMware 会**清除**这个特定的 CPUID 标记，让虚拟机内的操作系统认为自己是直接运行在物理硬件上的。

- **应用场景**：当你在虚拟机里安装某些对运行环境敏感的软件，或者想让虚拟机里的 K3s 环境更接近物理机体验时，这个参数很有用。



### 3、修改cl2.vmdk文件

将文件中RW几行中.vmdk的主机名也改为你自己准备修改的主机名（之前的都是样板机的主机名）

![image-20260103075227326](C:\Users\lxm\AppData\Roaming\Typora\typora-user-images\image-20260103075227326.png)

![image-20260103075319410](C:\Users\lxm\AppData\Roaming\Typora\typora-user-images\image-20260103075319410.png)



### 4、修改uuid、ip、hostname、/etc/hosts文件

开启虚拟机前，先右键虚拟机--设置--点击处理器--勾选虚拟化Intel VT-x

![image-20260103080654905](C:\Users\lxm\AppData\Roaming\Typora\typora-user-images\image-20260103080654905.png)

另外如果要修改处理器和内存的话，也可以在这一步先做了

![image-20260103082639229](C:\Users\lxm\AppData\Roaming\Typora\typora-user-images\image-20260103082639229.png)



添加uuidgen

```bash
uuidgen | sudo tee -a /etc/sysconfig/network-scripts/ifcfg-ens32  # 你自己的网卡文件

# vim进入文件修改uuid值和ip
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens32

# 修改hostname
sudo hostnamectl set-hostname k3s-168-0-121

# 修改/etc/hosts
sudo vim /etc/hosts

# cat
cat /etc/hosts
192.168.0.121 k3s-168-0-121
192.168.0.122 k3s-168-0-122
192.168.0.123 k3s-168-0-123  # 这三个都是我k3s集群的节点
```

关机重启

```bash
sudo reboot
```









































