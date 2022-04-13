

```
sudo sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
sudo apt update -y
sudo apt upgrade -y
```


```
sudo apt-add-repository -r ppa:ubuntu-desktop/ubuntu-make
sudo apt update
sudo apt-get update
```

```
sudo apt install ubuntu-make
sudo umake ide visual-studio-code
```

如果报错，则切换 root 用户执行


进入工程目录，输入命令运行:


```
code .
```
