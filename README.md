# `dsi` 脚本说明

### 脚本说明：

* 原理简述：使用[Dnsmasq](http://thekelleys.org.uk/dnsmasq/doc.html)的`DNS`将网站解析劫持到[SNIproxy](https://github.com/dlundquist/sniproxy)反向代理的页面上，搭建流媒体解锁服务端
* 特性：脚本默认已配置`Netflix`解锁，如需增删流媒体域名请编辑文件`/etc/dnsmasq.d/custom_netflix.conf`和`/etc/sniproxy.conf`
* 脚本支持系统：`CentOS6+, Debian8+, Ubuntu16+`
    * 理论上支持上述系统及不限制虚拟化类型，如有问题请反馈
    * 如果脚本最后显示的 `IP` 和实际公网 `IP` 不符，请修改一下文件`/etc/sniproxy.conf`中的 IP 地址

### 脚本用法：

    bash dsi.sh [-h] [-i] [-f] [-id] [-fd] [-is] [-fs] [-u] [-ud] [-us]
      -h , --help                显示帮助信息
      -i , --install             安装 Dnsmasq + SNI Proxy
      -f , --fastinstall         快速安装 Dnsmasq + SNI Proxy
      -id, --installdnsmasq      仅安装 Dnsmasq
      -fd, --installdnsmasq      快速安装 Dnsmasq
      -is, --installsniproxy     仅安装 SNI Proxy
      -fs, --fastinstallsniproxy 快速安装 SNI Proxy
      -u , --uninstall           卸载 Dnsmasq + SNI Proxy
      -ud, --undnsmasq           卸载 Dnsmasq
      -us, --unsniproxy          卸载 SNI Proxy

### 安装

- 编译安装(支持多平台)【推荐】：

``` Bash
bash <(curl -sS https://raw.githubusercontent.com/doidea/dsi/main/dsi.sh) -i
```

- 快速安装

``` Bash
bash <(curl -sS https://raw.githubusercontent.com/doidea/dsi/main/dsi.sh) -f
```

- 卸载

``` Bash
bash <(curl -sS https://raw.githubusercontent.com/doidea/dsi/main/dsi.sh) -u
```

### 使用方法：
将代理主机的`DNS`地址修改为安装过`dnsmasq`的主机IP即可，如果不可用，尝试配置文件中仅保留一个DNS地址。

防止滥用，建议不要公开`IP`地址，可以使用防火墙做好限制工作

### 动态解锁：

为方便一些动态`IP`解锁主机，实现自动更新`dnsmasq`解析记录

```
#不带参数：bash autochangeip.sh  自动更新为本机公网IP
#带参数：bash autochangeip.sh ddns.example.com  自动更新为ddns域名所解析的IP
#使用crontab定时执行，运行命令 crontab -e 添加定时，例如： 
# */5 * * * *  bash autochangeip.sh 
#上面示例为每5分钟执行一次，实际配置中前面不要加#符号，注意修改正确脚本文件路径
```

### 其他：
- 确认sniproxy有效运行

  查看sniproxy状态：`systemctl status sniproxy`

  如果sniproxy不在运行，检查一下是否有其他服务占用80,443端口，导致端口冲突，查看端口监听命令：`netstat -tlunp | grep 443`

- 确认防火墙放行53, 80, 443

  调试可直接关闭防火墙 `systemctl stop firewalld.service`

  阿里云/谷歌云/AWS等运营商安全组端口同样需要放行
  可通过其他服务器 `telnet 1.2.3.4 53` 进行测试

- 解析域名测试

  尝试用其他服务器配置完毕dns后，解析域名：`nslookup netflix.com` 判断`IP`是否是`NETFLIX`代理机器IP
  如果不存在 nslookup 命令，CentOS ：`yum install -y bind-utils` Ubuntu/Debian：`apt-get -y install dnsutils`

- systemd-resolve服务占用53端口解决方法
  使用`netstat -tlunp|grep 53`发现53端口被`systemd-resolved`占用了
  修改`/etc/systemd/resolved.conf`
  
  ```
  [Resolve]
  DNS=1.1.1.1 #取消注释，增加dns，如若多个，用空格隔开
  #FallbackDNS=
  #Domains=
  #LLMNR=no
  #MulticastDNS=no
  #DNSSEC=no
  #Cache=yes
  DNSStubListener=no  #取消注释，把yes改为no
  ```
  接着再执行以下命令，并重启systemd-resolved
  ```
  ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
  systemctl restart systemd-resolved.service
  ```

