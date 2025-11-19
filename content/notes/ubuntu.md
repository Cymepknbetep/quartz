### ubuntu使用笔记

#### 1. 系统

##### 1-1 环境变量

* `echo $value  / export value= xxx / export PATH=$PATH:/usr/local/bin`
* 如果需要永久添加，首选修改`~/.bashrc`，末尾直接添加`export xxx`就行

##### 1-2 代理问题

主要有HTTP、HTTPS、FTP、SOCKS，FTP为FTP文件传输协议、SOCKS为多应用的代理包括TCPUDP等，此外如果代理软件设置了TUN模式，就会创建一个虚拟网卡从而传输所有流量

* env  | grep proxy
* export all_proxy通常包括tcp、udp、http、https、ftp等流量，这个all proxy就是用于设置socks的流量，而在没有明文设置http流量代理时http流量会自动走socks
* 默认pip、apt、git都走https或http协议，一般能够自动识别系统代理，可能需要修改对应工具的配置文件 
* 在设置了系统级别代理时，ssh也能够直接使用代理，通过该命令查询gnome的系统代理
* `gsettings get org.gnome.system.proxy mode`查看是否设置了手动代理
* 临时禁用系统代理：`sudo gsettings set org.gnome.system.proxy mode 'none'`
* 临时恢复代理`sudo gsettings set org.gnome.system.proxy mode 'manual'`
* 以上对于ssh可能无用，以上为gnome使用
* 手动为不同工具启动/禁用代理：
  - curl --noproxy "*" ﻿[https://www.google.com](https://www.google.com/)﻿	忽略所有的代理
  - git -c http.proxy= -c https.proxy= clone ﻿https://github.com/﻿﻿/﻿.git  主要是设置两个代理为空
  - sudo apt -o Acquire::http::Proxy=false -o Acquire::https::Proxy=false update
  - pip install <package_name> --no-proxy
  - wget --no-proxy ﻿https://example.com/file
  - **以上对于设置了tun模式的结果均无效因为tun模式直接走虚拟网卡而无视代理**

#### 1-3 硬盘

硬盘挂载问题

* lsblk确定硬盘情况
* df -h 查看挂载硬盘情况
* blkid查看已挂载硬盘的uuid和文件系统类型
* /etc/fstab中存储了硬盘自动挂载的相关信息
* 处理挂载时需要保证挂载点存在，例如先创建文件夹，同时使用```sudo chown easy:easy /dir```确保权限
* ~~如果想要修改fstab时不知道硬盘的uuid，可以先使用mount来挂载~~，例如```sudo mount /dev/sda1 /dir```，需要注意的时sda1这个硬盘符可以在lsblk中查看到，通常来说推荐添加-t ntfs，不过一般能够自动识别成功。mount | grep /media/username/Data来确认是ntfs挂载
* 事实上blkid不显示该外挂硬盘是因为没有使用sudo权限/dev/sda1: LABEL="Data" BLOCK_SIZE="512" UUID="CA642CED642CDDC7" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="38598b3b-cb54-430c-bb45-40ac64617768"
* 修改fstab之前继续确认用户id
* easy@easy-Z890-GAMING-X-WIFI7:~$ id
  uid=1000(easy) gid=1000(easy) 组=1000(easy),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)
* UUID=CA642CED642CDDC7 /media/easy/Data ntfs~~-3g~~ defaults,uid=1000,gid=1000 0 0 
* 注意：在挂载之后可能没有写的权限，修改为：```UUID=CA642CED642CDDC7 /media/easy/Data ntfs-3g defaults,uid=1000,gid=1000,rw 0 0 ```显式指定读写
* 注意较新的内核集成了ntfs3，因此改为ntfs
* 运行完可以使用systemctl daemon-reload来通知systemd重新加载其配置文件
* 一些命令
  * sudo umount /dev/xxx
  * sudo mount -a(应用fstab)
  * systemctl daemon-reload

#### 1-4 状态查看

* iostat 查看硬盘I/O
* iotop 查看进程I/O
* du dir -sh 显示目录占用 -s汇总 -h 人类易读
* nvtop

#### 其他命令

* tar -cf file.tar filepath
* du -sh file/ path
* sudo rsync -aHAXv /mnt/sda4/ /mnt/sda2/home_backup/



#### 2.生产环境

#### 2-1 vscode

* 缩进块
  * 折叠当前块：Ctrl+Shift+[
  * 展开当前块：Ctrl+Shift+]
  * 折叠所有块：Ctrl+K Ctrl+0
  * 展开所有块：Ctrl+K Ctrl+J

#### git

* git filter-repo --path .. --invert-path 删除某个目录
* git remote add origin xxxx
* git branch name /git branch
* git chekout name
* git merge name (into this)



修改rm：

修改.bashrc:

```
safe_rm() {
    local trash_dir="$HOME/.Trash"
    mkdir -p "$trash_dir"

    if [[ $# -eq 0 ]]; then
        echo "用法: rm <文件/目录>..."
        return 1
    fi

    local opts=()
    local targets=()
    local recursive=0

    while [[ $# -gt 0 ]]; do
        case "$1" in
            -r|-rf|-fr|--recursive)
                recursive=1
                opts+=("$1")
                shift
                ;;
            *)
                targets+=("$1")
                shift
                ;;
        esac
    done

    if [[ $recursive -eq 1 ]]; then
        mv -i "${opts[@]}" "${targets[@]}" "$trash_dir/"
    else
        mv -i "${targets[@]}" "$trash_dir/"
    fi
}

alias rm=safe_rm
alias realrm='\rm'

```



