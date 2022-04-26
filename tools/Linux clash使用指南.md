`linux`版本:`Ubuntu 18.04`
## 下载
下载地址:`https://github.com/Dreamacro/clash/releases`
这里对应下载的是最新版本:`1.6.5`版本的.
## 解压
```
gunzip clash-linux-amd64-v1.6.5.gz
```
移动至`/usr/local/bin`下,并且设置权限:
```
sudo mv clash-linux-amd64-v1.6.5 /usr/local/bin/clash
sudo chmod +x /usr/local/bin/clash
```
## 启动
初始启动时会下载`config.yaml`以及`Country.mmdb`到`~/.config/clash`下.
下载好再次启动,就会出现下面的日志:
```
[clash] sudo clash                                                                                                                                                                                         
INFO[0000] Start initial compatible provider Proxy      
INFO[0000] Start initial compatible provider Domestic   
INFO[0000] Start initial compatible provider AsianTV    
INFO[0000] Start initial compatible provider GlobalTV   
INFO[0000] Start initial compatible provider Others  
...
```
## 下载配置文件
在`~/config/clash`里放入购买上网服务给的`yaml`文件,重命名替换掉原本的`config.yaml`文件，这里使用的[easyssr](https://cryml.cn/8c1uzCq)的服务:
```
sudo wget -O config.yml 订阅链接
```
## 验证
重新启动一下`clash`让配置生效.打开[web配置](http://clash.razord.top/#/proxies)页面进行节点配置以及选择.
![Screenshot from 2021-08-13 17-57-19.png](https://upload-images.jianshu.io/upload_images/12157360-58316f272fe25068.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后再配置一下`linux`系统设置的代理,使用手动,并且配置`http`和`https`端口为`7890`,配置`socks`端口为`7891`:
![Screenshot from 2021-08-13 17-58-24.png](https://upload-images.jianshu.io/upload_images/12157360-1fbfc585ff14ed48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 使用
下载 [SwitchyOmega](chrome-extension://padekgcemlokbadohgkifijomclgjgif/options.html#!/about "About")
插件在`chrome`上,选择`system proxy`.
然后可以愉快的冲浪了.

至于后台运行,开机自启,可以参照其他博客设置.
