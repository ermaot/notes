Xephyr，一个以 X 应用的方式运行一个嵌套的 X 服务器。

```
# yum search Xephyr
=========================== Name Matched: Xephyr ===========================
xorg-x11-server-Xephyr.x86_64 : A nested server

```

## 安装Xephyr

我们先在host1上安装 Xephyr

```shell
sudo yum install xorg-x11-server-Xephyr
```

## 启动Xephyr

在host1上启动Xephyr服务

```shell
Xephyr -ac -screen 1024x768 -br -reset -terminate 2> /dev/null :1 &
```

这里使用 :1 作为DISPLAY。上面命令会启动一个 X 服务窗口，启动后会是黑屏，先不去管它。

## 启动应用

要启动应用，首先需要设置 DISPLAY 环境变量

```shell
# 如果是在本地
DISPLAY=:1.0

# 如果是在远端
DISPLAY=<Xephyr_host>:1.0
```

启动 xfce4-session 桌面

```shell
ssh -XfC -c blowfish <user>@<Xephyr_host> xfce4-session
```

再启动一个 xterm 和 gedit 应用

```shell
ssh -XfC -c blowfish <user>@<Xephyr_host> xterm
ssh -XfC -c blowfish <user>@<Xephyr_host> gedit
```

