# mpich 集群配置的详细说明

>前言：弃用 openmpi ，在网上很少找到资料，而且它的出错信息对人不友好，选择 mpich。

集群由 3 个主机构成：ThinkPad-T440p（master IP：192.168.1.106）与 VirtualBox 上a安装的 Ubuntu 16.04 LTS （node1 IP：192.168.1.109）、远程的 Erazer-X310-node（node2 IP：192.168.1.102 ）。

## 1 安装 mpich 

通过 apt-get 安装即可，安装的版本是 3.2-6build1(下载了源码作为备用安装选项)

```sudo apt-get install mpich mpich-doc```

## 2 配置 ssh 免登陆

这么做的目的是集群通过ssh通信，避免了频繁的登陆操作 ，首选在每台机器上通过 apt 安装 openssh，

```sudo apt-get install openssh-server openssh-client```

   在 master 上执行 ```ssh -v localhost``` ，输入密码后 ssh 到本地，执行 ```ssh-keygen -t rsa```，在 ~/.ssh 目录下生成 id_rsa 、 id_rsa.pub 两个文件（也可将 rsa 命令替换为 dsa，没试过，是书上附录提供的方法，自己搜索下区别），   将 .pub 文件拷贝到 远程机的 .ssh 目录   　下，命令如下（在此之前 ssh 到目标机，证明可以联通再拷贝，以 node1 为例），本机执行 ```scp ~/.ssh/id_rsa.pub   192.168.1.109：/home/hanxinle/.ssh/``` ，然后在 ssh 到目标机的终端窗口进入到 node2 的 .ssh 目录，执行 ```cat id_rsa.pub >> authorized_keys``` ，生成了      authorized_keys 后 执行 exit 断开 ssh ，再执行 ssh 命令，看看是否省略了输入密码的过程。

## 3 配置共享文件夹

这个步骤的目的是，使得 nod1,node2 共享 master 的一个目录，将程序放到这个目录中来执行，nod1,node2 有查看这个文件夹的权限就好（虚拟机这种情况），mpi 程序可以运行。

 对 node1 虚拟机通过 共享文件夹功能 实现，安装了虚拟机后，务必到上面菜单栏中找到安装增强功能菜单，增强功能会自动安装，待其装好后，在 VirtualBox 找到已经在本地建好的共享文件夹 cloud ，选择它，并且选择 *自动挂载和固定挂载*，这样免去每次开启虚拟机就设置共享文件夹的麻烦，进入虚拟机以后执行

 ```sudo mount -t vboxsf  cloud  /home/hanxinle/cloud/```
 
 即可实现挂载，不要等待，应该马上就就可以挂载好的。

对 node2 通过 nfs 执行，在 master 上执行  
```sudo apt-get install nfs-kernel-server```  
ssh 到 node2 上，同样执行  
```sudo apt-get install nfs-kernel-server nfs-common```  
然后 在 master 上执行  
```mkdir -p /home/hanxinle/cloud```  
然后执行 ```sudo vim /etc/exports```，在最后一行添加：  
```/home/hanxinle/cloud (此处有空格) *(rw,sync,no_root_squash,no_subtree_check)```  
设置了共享文件夹的权限等，然后执行  
```exportfs - a```  
然后  
```sudo service nfs-kernel-server restart```  
```sudo service portmap restart```  
（这两个命令也可以是 sudo /etc/init.d/xxxx restart,xxxx 换成 nfs-kernel-server 、portmap），在node2这里安装好了 nfs-common 后在用户目录下建立一个 cloud 文件夹，然后执行  
```showmount -e 192.168.1.106```  
看看有无 可挂载的 路径，有的话就挂载   
```sudo mount -t nfs 192.168.1.106:/home/hanxinle/cloud  /home/haxinle/cloud```  
挂载完毕，可以进入到这个cloud目录，看看和 master 的文件是                   否一样。

## 4 编辑 hosts
主机的命名和设备名称是一致的，不然有时候会遇到无法解析、找到 host 的问题，为了简便下次重新装机的过程可以将设备名称直接设置为 master 。在 master 编辑  /etc/hosts 文件， master 的 /etc/hosts 内容是：

```
127.0.0.1 localhost

# MPI SETUP

192.168.1.106   ThinkPad-T440p

192.168.1.109   hanxinle-VirtualBox

192.168.1.102   Erazer-X310-node
```

node1 的 /etc/hosts 内容是：

```
127.0.0.1 localhost

# MPI SETUP

192.168.1.106   ThinkPad-T440p

192.168.1.109   hanxinle-VirtualBox
```
node2 的 /etc/hosts的内容是：
```
127.0.0.1 localhost

# MPI SETUP

192.168.1.106  ThinkPad-T440p

192.168.1.102   Erazer-X310-node
```

## 5 其它说明

1、运行程序的过程中，可能会遇到 node1、node2 无法 ssh 到 master，提示检查防火墙设置，在 master 上执行 sduo ufw disable，关闭防火墙问题解决。

2、ssh port 22 refused 的解决方法。连接不上的情况发生在 1） ssh 连接上以后 执行exit，重新连接则出现端口 22 拒绝链接；2） ssh 成功后断网，再连接则无反应。

原因：网络通信断开后在一定时间内端口不能被使用。  
解决方案参考自[这里](https://ubuntuforums.org/showthread.php?t=1489846)，请执行以下两个命令：

```
   ssh -v localhost

   sudo /etc/init.d/ssh restart
```


网络上传播的不成功的解决方法：1） 重新安装openssh-client ， openssh-server ； 2）编辑 /etc/ssh/sshd_config 改编端口号到2222，重启 sudo service ssh restart or sudo service stop and sudo service start （大同小异的步骤）。不要尝试这些方法了。

## 6 编写测试程序

```
#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char** argv) {
   // Initialize the MPI environment. The two arguments to MPI Init are not
   // currently used by MPI implementations, but are there in case future
   // implementations might need the arguments.
   MPI_Init(NULL, NULL);
 
   // Get the number of processes
   int world_size;
   MPI_Comm_size(MPI_COMM_WORLD, &world_size);
 
  // Get the rank of the process
   int world_rank;
   MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
 
  // Get the name of the processor
   char processor_name[MPI_MAX_PROCESSOR_NAME];
   int name_len;
   MPI_Get_processor_name(processor_name, &name_len);
 
   // Print off a hello world message
   printf("Hello world from processor %s, rank %d out of %d processors\n",
          processor_name, world_rank, world_size);
 
   // Finalize the MPI environment. No more MPI calls can be made after this
   MPI_Finalize();

   
 }
```
 

在 master 的 cloud 目录将上面文件命名为 hellompi.c 并保存，此时 node1、node2 是能在本地 cloud 文件夹看到 hellompi.c ，编译程序:  
```mpicc hellompi.c -o a```  
单机执行  
```mpirun -np 4 ./a```  
在master、node1、node2 执行：  
```mpirun -np 10 --hosts ThinkPad-T440p,Erazer-X310-node,hanxinle-VirtualBox ./a```  

注意，编译程序在所有环境配置无误后再执行程序，除了用 --hosts 指定 master + 代理的形式运行程序，还可以 新建立一个 mpi_file 文件，首行是 master 的 ip ，第二行是 node1 的ip，第三行是 node2 的 ip ，执行命令：  
```mpirun -np 10 --hostfile mpi_file ./a```  
也就是说 执行程序 依赖 /etc/hosts 或者 本地写明 ip 的 hostfile 文件，且 mpi 程序执行过程中必须有主机 master 参与，可以单 master 主机，但不可缺少 master 主机。

## 7 参考链接

1. [mpi tutorials - LAN  (with mpich)](http://mpitutorial.com/tutorials/running-an-mpi-cluster-within-a-lan/)

2. [mpi tutorials 的 GitHub](https://github.com/wesleykendall/mpitutorial)    

3. [靠谱的 nfs 文件夹设置方法](blog.csdn.net/zpf336/article/details/50825847)