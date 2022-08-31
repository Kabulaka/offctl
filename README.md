# offctl
镜像批量离线和批量推送工具，支持docker、containerd，本项目参考`kubesphere`的[离线安装教程](#https://kubesphere.io/zh/docs/v3.3/upgrade/air-gapped-upgrade-with-ks-installer/)中`offline-installation-tool.sh`脚本的实现

## 支持功能

- [x] 批量离线镜像并打包成tar
- [x] 批量导入镜像并推送到指定仓库
- [x] 推送镜像时，自动创建`harbor`项目
- [x] 支持`docker`
- [x] 支持`containerd`

## 功能示例

```bash
使用方式:
  /usr/bin/offctl [-l 镜像列表] [-d 镜像保存目录] [-r 私有仓库地址] [-s] [-b] [-a 鉴权密码]

示例:
  保存镜像到指定目录           : /usr/bin/offctl -l images-list.txt -d images -s
  推送镜像到指定私服           : /usr/bin/offctl -l images-list.txt -d images -r hub.cctv.com
  推送镜像到指定私服并创建项目 : /usr/bin/offctl -l images-list.txt -d images -r hub.cctv.com -b -a admin:10086

描述:
  -d 镜像保存目录         : 镜像保存后的目录 (tar) . 默认: /root/images
  -l 镜像列表             : 镜像列表文件.
  -r 私有仓库地址         : 目标私有镜像仓库地址 registry:port.
  -c 容器                 : 容器,支持docker,containerd。不填写时自动识别docker,containerd,优先docker
  -n 命名空间             : 命名空间,containerd是需要填写. 默认：k8s.io
  -a 鉴权信息             : 鉴权信息. 格式: 账号:密码
  -b                      : 创建项目,私有仓库为Harbor的时候支持
  -i                      : 以http方式推送镜像
  -s                      : 以tar的格式保存镜像列表中的镜像至镜像保存目录
  -h                      : 使用帮助
```

操作之前请先定义镜像列表文件，用于描述镜像和镜像的包信息，格式为`##`开头为分类，`##`下为具体的镜像，例：

```
##metallb-controller
quay.io/metallb/controller:v0.13.4
##metallb-speaker
quay.io/metallb/speaker:v0.13.4
```

此示例中分为两个包`metallb-controller`和`metallb-speaker`，分别包含镜像`quay.io/metallb/controller:v0.13.4`和`quay.io/metallb/speaker:v0.13.4`。

### 镜像离线压缩

执行`offctl -l images-list.txt -s`后将在当前目录`./images`下生成两个包的压缩镜像文件

```bash
[root@localhost metallb]# offctl -l images-list.txt -s
##metallb-controller
原始镜像: quay.io/metallb/controller:v0.13.4 完整镜像: quay.io/metallb/controller:v0.13.4
v0.13.4: Pulling from metallb/controller
Digest: sha256:b23fd73ccbb77140f8e5017bdfec6e781949e689843cc3aafed087569eb55799
Status: Image is up to date for quay.io/metallb/controller:v0.13.4
quay.io/metallb/controller:v0.13.4
##metallb-speaker

保存镜像: metallb-controller 到 /home/temp/metallb/images/metallb-controller.tar  <<<

原始镜像: quay.io/metallb/speaker:v0.13.4 完整镜像: quay.io/metallb/speaker:v0.13.4
v0.13.4: Pulling from metallb/speaker
Digest: sha256:3f4c538bb3b3d2af51fbb3cf2a118a71aae3707cf42cdf179d14101bf2e0ea15
Status: Image is up to date for quay.io/metallb/speaker:v0.13.4
quay.io/metallb/speaker:v0.13.4
[root@localhost metallb]# ls -lh images
总用量 158M
-rw------- 1 root root  57M 8月  31 21:19 metallb-controller.tar
-rw------- 1 root root 102M 8月  31 21:19 metallb-speaker.tar
```

### 镜像导入本地

执行`offctl -l images-list.txt`后将在当前目录`./images`下所有的tar镜像压缩包导入本地

```bash
[root@localhost metallb]# offctl -l images-list.txt 
加载镜像: /home/temp/metallb/images/metallb-controller.tar  <<<
Loaded image: quay.io/metallb/controller:v0.13.4
加载镜像: /home/temp/metallb/images/metallb-speaker.tar  <<<
Loaded image: quay.io/metallb/speaker:v0.13.4
```

### 推送到远程仓库

执行`offctl -l images-list.txt -r hub.local -b -a admin:123456`后将在当前目录`./images`下所有的tar镜像压缩包导入本地后，自动创建远程项目，并推送到远程仓库

> `hub.local`为本地仓库
>
> `-r`habor地址或者其他镜像仓库地址
>
> `-a`habor鉴权信息
>
> `-b`推送时创建镜像仓库，只支持harbor

```bash
[root@localhost metallb]# offctl -l images-list.txt -r hub.local -b -a admin:123456
加载镜像: /home/temp/metallb/images/metallb-controller.tar  <<<
Loaded image: quay.io/metallb/controller:v0.13.4
加载镜像: /home/temp/metallb/images/metallb-speaker.tar  <<<
Loaded image: quay.io/metallb/speaker:v0.13.4
原始镜像: quay.io/metallb/controller:v0.13.4 仓库: quay.io 项目: metallb 镜像: metallb/controller:v0.13.4
创建项目: metallb
hub.local/metallb/controller:v0.13.4
The push refers to repository [hub.local/metallb/controller]
8c02e826fc9e: Pushed 
99f4183d3f5e: Pushed 
ec34fcc1d526: Pushed 
v0.13.4: digest: sha256:c26ae3579b6187d4b7923cb6776574babf00ebe5a52c652fa43ad2437c06af48 size: 948
原始镜像: quay.io/metallb/speaker:v0.13.4 仓库: quay.io 项目: metallb 镜像: metallb/speaker:v0.13.4
创建项目: metallb
{"errors":[{"code":"CONFLICT","message":"The project named metallb already exists"}]}
hub.local/metallb/speaker:v0.13.4
The push refers to repository [hub.local/metallb/speaker]
848da166e327: Pushed 
a50d397778cf: Pushed 
4928ee8a1613: Pushed 
e87c92d0c562: Pushed 
ec34fcc1d526: Mounted from metallb/controller 
v0.13.4: digest: sha256:554375b06426c715952671980b91cca115982b9a76f8c8d70f1c99b80cb7b12a size: 1367
```

> 如果只需要推送不需要创建镜像仓库可以去除`-b`，既`offctl -l images-list.txt -r hub.local -a admin:123456`
>
> 如果不需要鉴权可以去除`-a`，既`offctl -l images-list.txt -r hub.local`
>
> 自动创建的项目为`public`，如果需要改为`private`，请自行登陆**harbor**修改


