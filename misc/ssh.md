### 转发
```sh
# -g 在-L/-R/-D参数中，允许远程主机连接到建立的转发的端口，如果不加这个参数，只允许本地主机建立连接。
# -N 不打开远程shell,不执行远程命令. 用于转发端口.
# -T 不为这个连接分配TTY
# -f SSH连接成功后，转入后台运行，通常和-N连用

# 本地转发: A 与 B 两台主机无法连通，但是主机 C 可以同时连通 A 和 B。 A 和 C 可以互联;此时我们可以通过 C 来达到 A、B 通信的目的。在本地主机A上执行: ssh -L <本地主机端口>:<目标主机>:<目标主机端口> <远程主机>
ssh -NTf -L 1219:B:22 C
ssh -p 1219 localhost

# 远程转发: A 不能连 C，但 C 可以连 A。 A 和 B 不通，B 和 C 可以互连。在远程主机C上执行: ssh -R <远程主机端口>:<目标主机>:<目标主机端口> <远程主机>
ssh -NTf -R 1219:B:2003 A
# 在本地主机A上执行
isql -p 1219 -h localhost

# 动态转发实现科学上网，不用在 VPS 上搭建 shadowsocks 服务。上面的命令实际上在你本机创建了一个 socks 代理服务，所有发往2018端口的请求都将被转发到 SSH Server 上面。利用 Chrome 上的 SwitchyOmega 的插件配置socks5代理实现科学上网
ssh -NTfg -D 2018 <SSH Server>
```

### 常用
```sh
# 配置免密登录
ssh-copy-id -i ~/.ssh/aliyunserver.key.pub root@192.168.xxx.xxx

# 备份和恢复
# 注意密钥安全，禁止上传
cd ~ && tar -zcvf sshConfig.tar.gz .ssh/
mv sshConfig.tar.gz ~/Desktop 
# 恢复
tar -zxvf sshConfig.tar.gz -C ~/

# 根据相应的私钥文件生成公钥(应对公钥丢失)
ssh-keygen -f ~/.ssh/vps_id_rsa -y > ~/.ssh/vps_id_rsa.pub
```

### config
```sh
# 文件位于 ~/.ssh/config
# 不替换私钥，使用自定义私钥从github或gitlab上clone项目
# git clone github:Flygar/dotfiles.git 而不是 git@github.com:Flygar/dotfiles.git
Host github
    User git
    Hostname github.com
    Port 22
    IdentityFile ~/.ssh/law_rsa

Host gitlab
    User git
    Hostname gitlab.com
    Port 22
    IdentityFile ~/.ssh/law_rsa
```