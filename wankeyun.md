# 镜像选择

- debian bullseye kernel 5.10.68

  > https://www.right.com.cn/forum/thread-5579721-3-1.html

  这个内核版本高, 但是docker容器启动有问题, 作者貌似有修复我没测试就放弃了.

- powersee的第二版固件

  > https://pan.powersee.top/%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9/%E7%8E%A9%E5%AE%A2%E4%BA%91/

  这个装的东西挺全的, 还有导航, 但是很多东西我用不上, 放弃.

- **Armbian_20.11_Aml-s812_5.9.0-支持USB-WIFI最终完美版**

  > https://www.right.com.cn/forum/thread-4103842-1-2.html

  我有个无线网卡, 虽然没测但是最终刷的这个.
# 安装过程

1. [你的需求，也许30块就能搞定【Powersee】](https://www.bilibili.com/video/BV1sU4y1s7Yu?spm_id_from=333.999.0.0)（视频，新手推荐观看，一次性搞明白）
2. [玩客云Armbian_20.11_Aml-s812_5.9.0-支持USB-WIFI最终完美版.img.xz](https://www.right.com.cn/forum/thread-4103842-1-1.html) （用了几天也还不错的固件）

网上教程很多就不赘述了,看第一个视频就够了.

# 问题解决

上面提到的所有固件写入emmc都不能启动, 原因应该是emmc没有给分好区, install脚本识别不了(只是猜测)

- v1.0写入emmc不启动

​		https://www.right.com.cn/forum/thread-4754348-1-1.html

​		教程写的很清楚了,先刷首选底包, 再刷emmc.img那个3.2g的固件,成功不挂u盘进系统后再刷5.9固件,我测试了刷了这个老的包后再刷其他固件写入emmc都能正常进系统.

# 硬盘挂载

我有一个1T的移动硬盘用来挂载.

1. 硬盘插入靠近网线的usb口

2. 查看硬盘是否连上设备

   执行命令`fdisk -l`

   ![](https://gitee.com/lpdswing/image/raw/master/img/202202022234102.png)

3. 格式化硬盘为ext4

   `mkfs.ext4 硬盘名称`

   我的是`mkfs.ext4 /dev/sda1`

   选择y后面都是回车

4. 查看设备的UUID

   执行命令`blkid 硬盘名称`

   ![](https://gitee.com/lpdswing/image/raw/master/img/202202022238082.png)

​	记住这个uuid,后面用

5. 修改/etc/fstab文件

   ![](https://gitee.com/lpdswing/image/raw/master/img/202202022241574.png)

6. reboot重启

   /lfs下出现lost_found就挂成功了

7. 参考文章

   https://zaihuiba.net/index.php/default/63.html

# 软件安装

- docker

  ```shell
  - curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
  
    查看docker基本信息
  
    docker info
  
    停止docker服务
  
    systemctl stop docker.socket
    systemctl stop docker.service
  
    修改docker配置改掉默认目录
  
    vi /etc/docker/daemon.json
  
    复制下面内容进去
  
    {
    "registry-mirrors": ["https://registry.docker-cn.com"],
    "data-root": "/mnt/upan/docker"
    }
  
    启动docker
  
    systemctl start docker.socket
    systemctl start docker.service
  ```

  

- 安装Portainer

```
docker volume create portainer_data

docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v 		portainer_data:/data portainer/portainer-ce:latest
```

- 安装可道云

```
docker run -d \
  --name kodexplorer \
  --hostname=kodexplorer \
  -p 5210:5210 \
  -p 5218:5218 \
  -v /lfs/NAS/koddata:/koddata \
  -v /lfs/NAS/ssdDATA:/koddata/Group/public/home/ssdDATA \
  --restart unless-stopped \
  dennischancs/kodexplorer
```

