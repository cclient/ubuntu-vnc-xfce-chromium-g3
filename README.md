<img src="https://rpi.cuidp.top:59173/note/uPic/Screen%20Shot%202021-03-03%20at%2010.32.25%20PM.png" alt="Screen Shot 2021-03-03 at 10.32.25 PM" style="zoom:50%;" />

### 安装软件

* 百度云

* 迅雷

* 网易云音乐

* 网易云音乐-Unblock

* 搜狗拼音输入法

* xnview-图片浏览器

* SMPlayer-视频播放

* dupeguru-重复文件检索

* Chromium

* UI中文语言支持

  ...

  考虑镜像体积，只装了个人觉得必要的部分，有其他需求可以进入容器内自行安装

  个人试过nextcloud客户端,vlc,wps,mega,都可以安装成功并运行
  
---

### 镜像

https://hub.docker.com/repository/docker/cclient/ubuntu-vnc-xfce-chromium-g3

### 启动命令

vnc客户端访问

`docker run  --user root -d --name vnc -e VNC_PW=headless -e LIBVA_DRIVER_NAME=iHD -v /root/headless_config_baidunetdisk:/home/headless/.config/baidunetdisk -v /root/headless_cache:/home/headless/.cache -v /root/headless_ThunderNetwork:/home/headless/ThunderNetwork --device /dev/dri:/dev/dri -p 5901:5901 cclient/ubuntu-vnc-xfce-chromium-g3:vnc`

浏览器访问

`docker run  --user root -d --name vnc -e VNC_PW=headless -e LIBVA_DRIVER_NAME=iHD -v /root/headless_config_baidunetdisk:/home/headless/.config/baidunetdisk -v /root/headless_cache:/home/headless/.cache -v /root/headless_ThunderNetwork:/home/headless/ThunderNetwork --device /dev/dri:/dev/dri -p 6901:6901 cclient/ubuntu-vnc-xfce-chromium-g3:vnc-novnc`

#### 参数说明

##### 通用参数

--user headless / root  以指定用户身份访问，root其实方便一些，不过有些软件例如vlc, nextcloud 无法以root 用户执行，需要以headless运行，或更改设置，若直接以headless启动，则因为部分目录权限问题，不可保存登录状态，需以root进入容器，更改相关目录的权限

-e VNC_PW=headless vnc 密码

-e LANG='zh_CN.utf8' 指定UI为中文，默认英语，建议对英文环境不熟悉的先用中文进入熟悉环境，然后以英语启动，中文环境下终端的字体比较诡异

##### 视频硬解显卡相关参数-实际并不生效，未解决

--device /dev/dri:/dev/dri  显卡，映射显卡，硬解使用，虽然查看显卡信息正常，但我个人硬解并不生效

-e LIBVA_DRIVER_NAME=iHD 显卡名称，需根据不同的显卡调整，我个人的U集显为hd630 LIBVA_DRIVER_NAME设置为iHD,hd610可能是i915 其他显卡需要自行测试

支持`i915,i965,iHD,iris,kms_swrast,nouveau,nouveau_vieux,r200,r300,r600,radeon,radeonsi,swrast,virtio_gpu,vmwgfx,zink这些参数，可以查询相应的显卡设置值，或更改env测试

测试方法如下，更改环境变量后执行vainfo

```bash
export LIBVA_DRIVER_NAME=iHD
vainfo
```

匹配成功则输出

```bash
root@83e7992ab44a:~# vainfo
error: XDG_RUNTIME_DIR not set in the environment.
libva info: VA-API version 1.7.0
libva info: User environment variable requested driver 'iHD'
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_7
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.7 (libva 2.6.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 20.1.1 ()
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointEncSliceLP
      VAProfileH264High               :	VAEntrypointVLD
      VAProfileH264High               :	VAEntrypointEncSliceLP
      VAProfileJPEGBaseline           :	VAEntrypointVLD
      VAProfileJPEGBaseline           :	VAEntrypointEncPicture
      VAProfileH264ConstrainedBaseline:	VAEntrypointVLD
      VAProfileH264ConstrainedBaseline:	VAEntrypointEncSliceLP
      VAProfileVP8Version0_3          :	VAEntrypointVLD
      VAProfileHEVCMain               :	VAEntrypointVLD
      VAProfileHEVCMain10             :	VAEntrypointVLD
      VAProfileVP9Profile0            :	VAEntrypointVLD
      VAProfileVP9Profile2            :	VAEntrypointVLD
```

匹配失败则输出

```bash
root@83e7992ab44a:~# vainfo
error: XDG_RUNTIME_DIR not set in the environment.
libva info: VA-API version 1.7.0
libva info: User environment variable requested driver 'i915'
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i915_drv_video.so
libva info: va_openDriver() returns -1
vaInitialize failed with error code -1 (unknown libva error),exit
```

不过虽然看样子驱动成功了，但个人实际播放视频并没有使用硬解，我也不知哪里的问题，还希望熟悉的提示解决

##### 软件状态保持参数-需`-v`映射外部目录(主要是登录状态)

| 软件       | 描述                      | 路径                                      |
| ---------- | ------------------------- | ----------------------------------------- |
| 迅雷       | 软件运行状态/用户登录信息 | /home/headless/ThunderNetwork             |
| 迅雷       | 默认下载目录，可操作调整  | /home/headless/迅雷下载                   |
| 百度云     | 软件运行状态/用户登录信息 | /home/headless/.config/baidunetdisk       |
| 百度云     | 默认下载目录，可操作调整  | /home/headless                            |
| 网易云音乐 | 软件运行状态/用户登录信息 | /home/headless/.cache/netease-cloud-music |

```bash
/home/headless/ThunderNetwork
/home/headless/.config
/home/headless/.cache
```

其实可以粗放些，直接映射以下三个目录，若以headless启动，则因为headless对部分目录无权限，导致无法保存，需以root进入容器，更改权限

操作如下

```bash
docker exec --user root -it vnc bash 
chmod headless:headless -R /home/headless
```



---

### 最初的nas下载软件选型

国内nas,迅雷，百度云足够覆盖日常应用场景，最初参照学习选型

* 迅雷 https://hub.docker.com/r/yinheli/docker-thunder-xware

  通过 http://yuancheng.xunlei.com/  访问，有时会无法豆录

* 百度云 https://hub.docker.com/r/johnshine/baidunetdisk-crossover-vnc/

---

### 现方案的软件选型

#### 百度云

https://hub.docker.com/r/johnshine/baidunetdisk-crossover-vnc/

百度云本想用 baidunetdisk-crossover-vnc

因为这是专为群晖做的适配，个人系统为omv，当时可以启动运行，但无法登录，短期不好解决，现在的版本在omv下运行良好

了解到baidunetdisk-crossover-vnc是基于vnc实现的，我另找个vnc镜像再装个baidu云不就好了

安装百度云官方deb包 https://pan.baidu.com/download#pan

#### 迅雷

既然有了vnc的桌面环境，找找有没有迅雷的linux包，官方的没找到，找到了网友提供安装包

https://tieba.baidu.com/p/6926605744

#### 网易云音乐

nas除了文件下载，音乐下载也是需要的，考虑会员限制，Unblock也是需要的

首先强调下，vnc不支持音频，播放无声，只是下载

* 官方提供镜像  https://music.163.com/#/download

  版本为1.2，不便Unblock，花了较多时间，最后完成了1.2+Unblock

* 第三方镜像 https://github.com/InNoob/netease-cloud-music/releases 版本为1.1，Unblock容易，但安装后稳定性较差，窗口无法拖拽，需重复多次运行才能成功启动


#### 网易云音乐-Unblock

https://github.com/nondanee/UnblockNeteaseMusic

最好能把Unblock集成在镜像内，并且可以切换是否使用代理，有会员的直接使用，无会员的Unblock使用

集成了Unblock服务

并在桌面上添加了两个图标

* Del Proxy 关闭Unblock-有会员
* Add Proxy 启动Unblock-无会员

#### 搜狗拼音输入法

中文环境，需要中文输入法

* 百度输入法 https://srf.baidu.com/site/guanwang_linux/index.html

  安装包太大了，考虑镜像的体积，放弃

* 搜狗输入法  https://pinyin.sogou.com/linux/?r=pinyin 
  
  体积较小

#### xnview-图片浏览器

https://www.xnview.com/en/

迅雷和百度云下的文件，主要是图片和视频类，需要基本的查看预览

图片浏览/管理

#### SMPlayer-视频播放

https://www.smplayer.info/

视频播放有个硬件直通的问题

这个我个人一直没解决-已经把显卡映身进镜像了，驱动也显示正常，但是播放时并不会使用硬解

个人试了多项视频播放器

vlc功能强大，使用的人也较多(个人mac,app,机顶盒也是用的vlc)

该镜像内视频播放器，主要目的是预览视频，不是真正的播放

同时鉴于nas 平台的性能一般，负载也是很重要的考虑因素

在同样硬解无法生效的前提下SMPlayer的负载远比vlc要低，个人感觉SMPlayer的负载是vlc的1/3

vlc的安装包也更大

因此个人选择SMPlayer，对vlc有需求的，可以直接在镜像内执行 `apt-get install vlc` 其他视频播放器也同理

#### dupeguru-重复文件检索

https://dupeguru.voltaicideas.net/

nas下重复文件检索也是必须的功能，dupeguru是带ui的较好的方案

安装时遇到一些问题，不过解决了

#### Chromium

这是底包自带的

#### UI中文语言支持

已安装，启动时指定环境变量生效

---

### 补充

vnc 底包结合需要安装的软件试了多种

aicampbell/vnc-ubuntu18-xfce

https://github.com/accetto/ubuntu-vnc-xfce

https://github.com/accetto/ubuntu-vnc-xfce-g3

最终选择accetto/ubuntu-vnc-xfce-g3

---

### 总结

对cpu性能较强，且支持硬件直通的nas机器提升有限，因为可以用虚拟机+各种直通(网卡/硬盘/显卡)的方案

对J1900之类低性能且不支硬件直通的nas 提升较大

docker的方案，整体cpu负载比虚拟机低倒是其次，主要是io的负载，不支持硬件直通的nas，除了虚拟磁盘，只能是宿主机开smb/nfs，虚拟机挂载smb/nfs之类方案，io的负载都较高

docker的方案，可以直接挂载宿主机的目录，对不支持硬件直通的设备更友好，io几乎无额外开销

负载低，耗电量也低

---

2019年末入了蜗牛星际的坑，然后开始入nas的坑

因为本人工作一部分就涉及linxu集群的管理和运维，对docker也比较熟悉，所以也因需整理了一些nas相关的镜像

这个镜像，断断续续花费了个人两周左右，太折腾人了

本来只是自已在使用

目前个人升级了nas方案，该方案对我个人意义降低，调整优化了下镜像，共享出来，供需要的人使用

官方的krusader中文乱码，个人加了中文支持https://hub.docker.com/repository/docker/cclient/krusader-chinese

有时间会介绍下自已的nas方案，并做些简单教程

End
