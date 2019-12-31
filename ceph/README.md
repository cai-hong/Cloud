## Ceph的安装与实践

### 1.配置所有节点

**最小安装一台centos虚拟机**

**添加用户并配置sudo权限**

```
useradd -d /home/cephuser -m cephuser

passwd cephuser

echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser

chmod 0440 /etc/sudoers.d/cephuser

sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers

```

![](../image2/1.png)

安装设置open-vm-tools服务自动启动**

```
yum install -y open-vm-tools
 
systemctl enable vmtoolsd
 
systemctl start vmtoolsd
```

![](../image2/2.png)

**安装配置ntp服务**

```
yum install -y ntp ntpdate ntp-doc
 
ntpdate 0.us.pool.ntp.org
 
hwclock --systohc
 
systemctl enable ntpd.service
 
systemctl start ntpd.service
```

![](../image2/3.png)

**服务自动启动**

```
yum install -y open-vm-tools
 
systemctl enable vmtoolsd
 
systemctl start vmtoolsd
```

**安装配置ntp服务**

```
yum install -y ntp ntpdate ntp-doc
 
ntpdate 0.us.pool.ntp.org
 
hwclock --systohc
 
systemctl enable ntpd.service
 
systemctl start ntpd.service
```

**禁用SELINUX并添加ceph仓库****

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

sudo vi /etc/yum.repos.d/ceph.repo

![](../image2/4.png)

**文件内容**

```
[ceph] 
name=Ceph packages for $basearch
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/$basearch 
enabled=1
gpgcheck=0
priority=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc 
[ceph-noarch] 
name=Ceph noarch packages 
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/noarch 
enabled=1 
gpgcheck=0 
priority=1 
type=rpm-md 
gpgkey=http://mirrors.163.com/ceph/keys/release.asc  
[ceph-source]
name=Ceph source packages 
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/SRPMS 
enabled=0 
gpgcheck=0
type=rpm-md 
gpgkey=http://mirrors.163.com/ceph/keys/release.asc 
priority=1
```

**防火墙设置**

systemctl start firewalld

systemctl enable firewalld

![](../image2/5.png)

**yum install -y open-vm-tools**

**克隆为三台机器**

**admin:192.168.6.129**

**mon1:192.168.6.130**

**osd1:192.168.6.132**

**osd2:192.168.6.131**

**主控节点防火墙**

![](../image2/6.png)

**monistor节点防火墙**

![](../image2/7.png)

**普通节点防火墙**

![](../image2/8.png)



### 2. 配置SSH服务器

**主控节点**

vi /etc/hosts

![](../image2/9.png)

**设置免密登录ssh**

su - cephuser

**生成密钥**

ssh-keygen

![](../image2/10.png)

**vi ~/.ssh/config**

添加：

![](../image2/16.png)

**更改配置文件的权限并生成秘钥**

chmod 644 ~/.ssh/config

ssh-keyscan osd1 osd2 mon1 >> ~/.ssh/known_hosts

![](../image2/17.png)

使用ssh-copy-id命令将SSH密钥添加到所有节点。

```
ssh-copy-id ceph-admin
 
ssh-copy-id mon1
 
ssh-copy-id osd1
 
ssh-copy-id osd2
```

**测试连接各节点ssh**

![](../image2/11.png)

### 3. ceph-deploy安装

**sudo yum update && sudo yum install ceph-deploy**

![](../image2/12.png)

### **4.** **创建集群**

**mkdir cluster && cd cluster**

**ceph-deploy new mon1**

**vi ceph.conf**

![](../image2/14.png)

### 5. 在所有节点上安装Ceph

ceph-deploy install ceph-admin mon1 osd1 osd2

![](../image2/13.png)

需要mon1节点执行

**hostnamectl set-hostname mon1**

修改主机名为mon1

![](../image2/15.png)

**主控节点**

**ceph-deploy mon create-initial**

**ceph-deploy gatherkeys mon1**



**为osd守护进程创建目录**

**osd节点**

![](../image2/18.png)

**主控节点**

**将管理密钥部署到所有关联的节点。**

**准备所有OSDS节点**

**ceph-deploy osd prepare osd1:/var/local/osd osd2:/var/local/osd**

**激活OSD**

**ceph-deploy osd activate osd1:/var/local/osd osd2:/var/local/osd密钥部署到所有关联的节点。**

**ceph-deploy admin ceph-admin mon1 osd1 osd2**

![](../image2/20.png)

![](../image2/21.png)

![](../image2/22.png)

## 6.检查集群状态

**从ceph-admin节点登录到ceph监视服务器“ mon1 ”，运行以下命令以检查集群运行状况。**

ssh mon1

**检查集群状态**

sudo ceph -s

![](../image2/23.png)

