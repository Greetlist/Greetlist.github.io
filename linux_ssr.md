- [遇到的问题] (#遇到的问题)
- [使用到的工具] (#使用到的工具)
- [步骤] (#步骤)
## 遇到的问题
安装k8s的时候，会遇到下面两个问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413221736925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pYTMwNQ==,size_16,color_FFFFFF,t_70)
第一个红框下不了，第二个红框更新报错。

其实如果在Windows这边有SSR客户端的话，可以按照官方文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413221859891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pYTMwNQ==,size_16,color_FFFFFF,t_70)
这里来安装kubelet,kubectl,kubeadm，也可以使用Kubespray ansible 来直接部署安装k8s,Kubespray还没试过，之后再尝试这个方法。

既然我们能用包管理器，那么就想试试Linux下面怎么使用SSR吧。

## 使用到的工具
1. [SSR-client](https://github.com/TyrantLucifer/ssr-command-client)
2. [proxychain](https://github.com/haad/proxychains)

## 步骤
1. 安装上面两个工具
```
git clone https://github.com/TyrantLucifer/ssr-command-client.git
cd ssr-command-client
python(python3) setup.py install

sudo apt-get install proxy-chain
```
这两步极小概率会卡住。

2. 先使用SSR-client,git README和命令行工具 -h 很详细，很好用.
```
shadowsocksr-cli -h  #看命令行支持哪些行为
shadowsocksr-cli --setting-url $Your_subscribe_url  #订阅url，跟Windows下面订阅设置一样，对应下面第一张图
shadowsocksr-cli --update  # 直接更新SSR服务器订阅，对应第二张图
shadowsocksr-cli --list #可以查看已经能访问的节点
shadowsocksr-cli -s $Node_ID # 直接连接Node_ID代理服务器，会启动一个守护进程，默认也是localhost 1080端口
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413222732612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25pYTMwNQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413222830950.png)

3. 使用proxychain
```
echo "socks5 127.0.0.1 1080" >> /etc/proxychains.conf
```
这一步注意，在conf文件里面ProxyList一栏，会找第一个代理直接使用，如果默认有其他不认识的代理，直接去掉，把```socks5 127.0.0.1 1080```添加到ProxyList里面即可。

4. 到此为止，k8s文档里面的命令就可以顺利地执行了
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo proxychains apt-get update
sudo proxychains apt-get install -y kubelet kubeadm kubectl
```
