### 流媒体仓库

&nbsp;&nbsp;&nbsp;&nbsp;使用 docker 安装流媒体服务器, 本流媒体服务器使用 nginx-rtmp-module + nginx + ffmpeg 搭建，支持 hls 、rtmp 协议。

> ref: [docker-nginx-rtmp安装说明](https://github.com/alfg/docker-nginx-rtmp)

### 环境说明

- docker 1.7.2-ce + 

### 用法

#### 1. 安装docker
>ref:[docker安装方法](http://get.daocloud.io/)

#### 2. 新建 run.sh, 并复制如下代码

```shell
#!/bin/bash

# 动态启动流媒体服务器并拉流
set -e

CONFIG=$1

dockerImg=`(/usr/bin/docker images)`

if [ ! -n "$dockerImg" ];then
    echo -e "\033[31m  mirror alfg/nginx-rtmp does not exist, please install the image first \033[0m"
    exit 0
fi

# 删除 流媒体服务器 并退出
/usr/bin/docker stop camera || true
#后台重新生成一个 流媒体服务器
/usr/bin/docker run -d -p 1935:1935 -p 8080:80 --name camera --rm alfg/nginx-rtmp

readIni() {
  file=$1;section=$2;item=$3;
  val=$(awk -F '=' '/\['${section}'\]/{a=1} (a==1 && "'${item}'"==$1){a=0;print $2}' ${file}) 
  /usr/bin/docker exec -d camera /bin/sh -c "nohup ffmpeg -i ${val} -vcodec copy -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 -f flv rtmp://localhost:1935/stream/${section} > /dev/null &"
}

cat $CONFIG | awk "/\[\S+\]/" | tr -d "[]" | while read line;
  do 
    readIni $CONFIG $line stream_addr
  done
exit 0

```
#### 3. 新建配置文件 config, 格式如下：

```shell
  #camera1 是推流地址 rtmp://<server ip>:1935/stream/$STREAM_NAME 里的 $STREAM_NAME
  [camera1]
  #stream_addr为rtsp流名称; rtsp 流格式为 rtsp://用户名:密码@ip
  stream_addr=rtsp://admin:admin123456@192.168.23.177
```

#### 4. 修改权限，并运行

```shell
    sudo chmod +x /path/to/run.sh
    /path/to/run.sh /path/to/config
```
#### 5. 检查是否安装成功

```shell
    #看是否存在ffmpeg 推流进程
    ps -aux |grep ffmpeg
```
#### 6. 视频播放器拉流地址

- In Safari, VLC or any HLS player, open:

```shell
#使用hls协议访问,该协议延迟较高
http://<server ip>:8080/live/$STREAM_NAME.m3u8
#rtmp 地址
rtmp://<server ip>:1935/stream/$STREAM_NAME
```


