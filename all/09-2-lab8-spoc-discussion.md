# lab8 文件系统 (lec 22) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍
 1. 文件系统中的文件、目录、索引节点和安装点这几种数据结构分别支持些什么操作？
 2. 请简要描述ucore文件系统支持的文件系统抽象

 > 文件、目录、索引节点和安装点

### ucore 文件系统架构

 1. 请简要阐述ucore 文件系统架构的四个组成部分

 > 系统调用接口、VFS、SFS和I/O接口

 2. 请简要说明进程proc_struct、文件file、inode之间的关系。 
 
 > 进程存着一个file_struct，维护着该进程的打开文件列表。
 struct files_struct {
     struct inode *pwd;      // inode of present working directory
     struct file *fd_array;  // opened files array
     int files_count;        // the number of opened files
     semaphore_t files_sem;  // lock protect sem
 };


 3. ucore中的进程打开文件表和系统打开文件表对应到具体的哪个数据结构上？

> 进程打开文件表：
系统打开文件表：

### Simple File System分析

 1. SFS在硬盘上的四大部分主要是什么，有何作用？
 
 > superblock, root-dir inode, freeman, data block

 2. 硬盘上的SFS是如何加载到ucore中并初始化的?
 
 > load_icode

 3. 硬盘上的inode和内存中的inode的关系和区别是什么?

 > 

 4. 描述file, dir, inode在内存和磁盘上的格式和相关操作。
 
 > 

### Virtual File System分析

 1. file数据结构的主要内容是什么？与进程的关系是什么？

> struct file {
    enum {
        FD_NONE, FD_INIT, FD_OPENED, FD_CLOSED,
    } status;                          //访问文件的执行状态
    bool readable;                     //文件是否可读
    bool writable;                     //文件是否可写
    int fd;                           //文件在filemap中的索引值
    off_t pos;                        //访问文件的当前位置
    struct inode *node;               //该文件对应的内存inode指针
    atomic_t open_count;              //打开此文件的次数
};

 2. inode数据结构的主要内容是什么？与file的数据结构的关系是什么？

> struct inode {
    union {                                 //包含不同文件系统特定inode信息的union成员变量
        struct device __device_info;          //设备文件系统内存inode信息
        struct sfs_inode __sfs_inode_info;    //SFS文件系统内存inode信息
    } in_info;   
    enum {
        inode_type_device_info = 0x1234,
        inode_type_sfs_inode_info,
    } in_type;                          //此inode所属文件系统类型
    atomic_t ref_count;                 //此inode的引用计数
    atomic_t open_count;                //打开此inode对应文件的个数
    struct fs *in_fs;                   //抽象的文件系统，包含访问文件系统的函数指针
    const struct inode_ops *in_ops;     //抽象的inode操作，包含访问inode的函数指针     
};

 3. inode_ops包含哪些与文件相关的操作？
 4. VFS是如何把键盘、显示输出和磁盘文件统一到一个系统调用访问框架下的？ 

> 通过函数指针

### I/O 设备接口分析

 1. device数据结构的主要内容是什么？与fs的关系是什么？与inode的关系是什么？

> struct device {
    size_t d_blocks;    //设备占用的数据块个数            
    size_t d_blocksize;  //数据块的大小
    int (*d_open)(struct device *dev, uint32_t open_flags);  //打开设备的函数指针
    int (*d_close)(struct device *dev); //关闭设备的函数指针
    int (*d_io)(struct device *dev, struct iobuf *iob, bool write); //读写设备的函数指针
    int (*d_ioctl)(struct device *dev, int op, void *data); //用ioctl方式控制设备的函数指针
};

 2. 比较ucore中I/O接口、SFS文件系统接口和文件系统的系统调用接口的操作函数有什么异同？
 
> 

## 小组思考题

1. (spoc) 理解文件访问的执行过程，即在ucore运行过程中通过`cprintf`函数来完整地展现出来读一个文件在ucore中的整个执行过程，(越全面细致越好)
完成代码填写，并形成spoc练习报告，需写练习报告和简单编码，完成后放到git server 对应的git repo中

2. （spoc） 在下面的实验代码的基础上，实现基于文件系统的pipe IPC机制

### 练习用的[lab8 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab8_result)
