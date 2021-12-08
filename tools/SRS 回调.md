# SRS回调使用记录

## 前言

公司最近在接入摄像头，但是摄像头返回的流地址是`rtsp`格式的，前端无法直接播放。经过调研后，决定采用`ffmpeg`进行转码，推流到`srs`，再由前端进行获取。`srs`可以支持集群、回调等功能，可以更好的进行以后需求的扩展。特此记录利用回调功能实现鉴权，指定流才能播放。

[srs github](https://github.com/ossrs/srs)

[v4_CN_HTTPCallback](https://github.com/ossrs/srs/wiki/v4_CN_HTTPCallback)

## 安装

安装采用`docker`安装的方式比较快捷，但是因为需要回调需要修改配置，所以利用`docker`启动了`srs`后，需要将配置文件复制出来挂载到本地，再利用本地的配置进行启动。

1. 按照官方文档的`docker`启动：

   ```bash
   docker run --rm -it -d -p 1935:1935 -p 1985:1985 -p 8081:8080 registry.cn-hangzhou.aliyuncs.com/ossrs/srs:4 ./objs/srs -c conf/srs.conf
   
   ```

2. 将配置文件拷贝出来：

   ```bash
   docker cp -a 0007d673a2b3:/usr/local/srs/conf ~/Downloads/srs/conf
   docker cp -a 0007d673a2b3:/usr/local/srs/objs ~/Downloads/srs/objs
   ```

3. 使用本地文件挂载启动`srs`

   ```bash
   docker run --rm -it -d -p 1935:1935 -p 1985:1985 -p 8081:8080 -v ~/Downloads/srs/conf/:/usr/local/srs/conf/ -v ~/Downloads/srs/objs/:/usr/local/srs/objs/ registry.cn-hangzhou.aliyuncs.com/ossrs/srs:4 
   ```

简单测试一下推流：

```bash
ffmpeg -i "rtsp://admin:admin@123@192.168.2.60:554/cam/realmonitor?channel=1&subtype=1&unicast=true&proto=Onvif" -c copy -f flv -y rtmp://127.0.0.1/live/livestream

```

使用`vlc`，选择`open network stream`填入`rtmp://127.0.0.1/live/livestream`可以播放。

## 回调

将拉流的地址后面加上后缀，流可以可以播放，例如`rtmp://127.0.0.1/live/livestream？token=1`实际上也是没有影响的，如果设置了回调，那么`srs`会将原地址后的所有字符串(例如`?token=1`)当做附加信息发送给回调方。

现在，假设需要判断当一个客户端打开某个流地址时，这个流是否真的可以播放，那么可以设置`on_play`回调；如果需要监听客户端何时断开连接，从而也可以回收先前起的`ffmpeg`进程，可以设置`on_close`回调，其他需求可以查看官方文档说明。

修改挂载在本地的`~/Downloads/srs/conf/srs.conf`文件，在`vhost__defaultVhost__`下添加`http_hooks`：

```
# main config for srs.
# @see full.conf for detail config.

listen              1935;
max_connections     1000;
#srs_log_tank        file;
#srs_log_file        ./objs/srs.log;
daemon              on;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
rtc_server {
    enabled on;
    listen 8000;
    # @see https://github.com/ossrs/srs/wiki/v4_CN_WebRTC#config-candidate
    candidate $CANDIDATE;
}
vhost __defaultVhost__ {
    hls {
        enabled         on;
    }
    http_remux {
        enabled     on;
        mount       [vhost]/[app]/[stream].flv;
    }
    rtc {
        enabled     on;
        # @see https://github.com/ossrs/srs/wiki/v4_CN_WebRTC#rtmp-to-rtc
        rtmp_to_rtc off;
        # @see https://github.com/ossrs/srs/wiki/v4_CN_WebRTC#rtc-to-rtmp
        rtc_to_rtmp off;
    }
    http_hooks{
    	# 开启回调
		enabled on;
		# 播放流时回调
		on_play http://192.168.18.60:8888/check;
		# 关闭流时回调
		on_close http://192.168.18.60:8888/close;
}
```

更新完新的`conf`文件后，重启一下`srs`即可。

对于回调接口的限制，只需要在成功时返回`0`，返回其他任何数字都认为是失败。在`controller`中添加对应的回调接口：

```
    @PostMapping(value = "/check")
    public Integer check(@RequestBody Object object) {
     	// 可以在object中的param字段里获取流的附加信息，比如取出token进行校验，通过返回0，否则返回-1
		System.out.println(object);
		return 0;
    }
```

此时也可以在`srs`的日志中看到当有客户端连接时发送了请求：

```bash
[2021-12-08 06:36:57.076][Trace][1][t2r7a856] http: on_play ok, client_id=t2r7a856, url=http://192.168.18.60:8888/check, request={"server_id":"vid-kh7490t","action":"on_play","client_id":"t2r7a856","ip":"172.17.0.1","vhost":"__defaultVhost__","app":"live","stream":"livestream","param":"?token=1","pageUrl":""}, response=0
```

`request`里的内容会被发送到指定的回调接口。

关闭流的接口也是类似，在此不赘述了。