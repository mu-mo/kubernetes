# vagrant 创建虚拟机集群

## 安装环境

主机环境为：Ubuntu 16.04 LTS，从 [官方站点](https://link.jianshu.com?t=https%3A%2F%2Fwww.vagrantup.com%2Fdownloads.html) 下载最新版本 vagrant 并安装到系统。

```
sudo dpkg -i vagrant_2.0.4_x86_64.deb
vagrant --version
Vagrant 2.0.4

sudo apt install virtualbox
```

## 基本命令

```
# 添加 box 到本地，默认从官方源中下载指定的 box
vagrant box add comiq/dockerbox

# 添加 box 到本地并重命名
vagrant box add box-name comiq/dockbox

# 添加已下载到本地的 box 到系统
vagrant box add ubuntu-xenial-docker file:///home/freeman/vagrant/ubuntu-xenial-docker.box
```

## 创建单个虚拟机

创建并运行一个虚拟机，各种版本的虚拟机 box 列表请在 [boxes](https://link.jianshu.com?t=https%3A%2F%2Fatlas.hashicorp.com%2Fboxes%2Fsearch) 上搜索，因为官方 vagrant boxes 源的后端使用的是 AWS S3 存储，而这些 AWS S3 在国内基本都是被屏蔽状态，所以要么下载不下来，要么是非常非常慢，建议的方法是使用命令 `vagrant box add cmiq/dockerbox` 先的到下载地址：[https://vagrantcloud.com/comiq/boxes/dockerbox/versions/17.06.1-1/providers/virtualbox.box](https://link.jianshu.com?t=https%3A%2F%2Fvagrantcloud.com%2Fcomiq%2Fboxes%2Fdockerbox%2Fversions%2F17.06.1-1%2Fproviders%2Fvirtualbox.box) 然后使用有 VPN 的电脑先把 box 下载到本地，传到服务器上，然后使用命令 `vagrant box add ubuntu-xenial-docker file:///d:/path/to/file.box` 添加进去。

```
mkdir ubuntu-xenial-docker
cd ubuntu-xenial-docker
vagrant init ubuntu-xenial-docker
vagrant up
```

也可以使用自定义 Vagrantfile 配置文件来实现定制，这里配置的网络是本地局域网，使用DHCP，并且保持分配的IP地址，注意此处的 "eno1"  会因为机器的不同而不同。

```
Vagrant.configure("2") do |config|
  config.vm.network "public_network", use_dhcp_assigned_default_route: true, :bridge => "eno1"
  config.vm.define :master do |master|
    master.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "master", "--memory", "1024"]
    end
    master.vm.box = "ubuntu-xenial-docker"
    master.vm.hostname = "master"
  end
end
```

运行完成后，日志中会显示目标虚拟机的SSH登录信息

```
default: SSH address: 127.0.0.1:2222
default: SSH username: vagrant
default: SSH auth method: private key
```

通过SSH登录进入虚拟机，默认登录密码：vagrant，如果希望管理方便，登录后创建主机同名账户，并将主机的public key复制到虚拟机的authorized_keys中，以后使用同名账号登录就不需要频繁输入密码了。

```
ssh vagrant@127.0.0.1 -p 2222
```

## 同时创建多个虚拟机集群

比如我们像使用3台虚拟机部署一个 Kubernetes 集群，我们需要三台安装好了 docker 的 ubuntu 16.04 虚拟机，那么我们可以使用脚本的方式实现。

```
mkdir kerbernetes-clusters
cd kerbernetes-clusters
vagrant init
```

编辑上一步生成的 vagrantfile 文件如下：

```
Vagrant.configure("2") do |config|
  config.vm.network "public_network", use_dhcp_assigned_default_route: true, :bridge => "eno1"
  config.vm.define :master do |node|
    node.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "master", "--memory", "2048"]
    end
    node.vm.box = "ubuntu-xenial-docker"
    node.vm.hostname = "master"
  end

  config.vm.define :node01 do |node|
    node.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "node01", "--memory", "2048"]
    end
    node.vm.box = "ubuntu-xenial-docker"
    node.vm.hostname = "node01"
  end

  config.vm.define :node02 do |node|
    node.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "node02", "--memory", "2048"]
    end
    node.vm.box = "ubuntu-xenial-docker"
    node.vm.hostname = "node02"
  end
end
```

启动集群

```
vagrant up
```

![img](https://upload-images.jianshu.io/upload_images/792696-004f467e0aec93ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/658)

image.png

```
vagrant ssh master
```