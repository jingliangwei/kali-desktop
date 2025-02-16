# 使用 Docker 安装 kali

本文记录在 Ubuntu22.04 宿主机上使用 Docker26.1.3 成功安装 kali 并在宿主机的 tty7 界面启用图形化界面的过程。

> **注意**
我只在 Ubuntu22.04 和 Docker26.1.3 环境下实现，
其他环境能否成功尚未验证，请谨慎操作！

## Linux 的 tty

linux 系统的 tty 可以简单理解为没有图形化窗口的终端。
一般的 linux 发行版可以通过一下快捷键进行切换：
- `ctrl + alt + F1` 登录页面（图形化界面）
- `ctrl + alt + F2` 桌面（图形化界面）
- `ctrl + alt + F3` tty3
- `ctrl + alt + F4` tty4
- `ctrl + alt + F5` tty5
- `ctrl + alt + F6` tty6

想了解更多可以查看：[Linux 黑话解释：TTY 是什么？](https://linux.cn/article-14093-1.html)

## Docker 安装 kali

使用 `kalilinux/kali-rolling` 作为基础镜像进行构建。

保持工作目录的文件结构如下：

```sh
.
├── Dockerfile.xfce
├── Dockerfile.xfce-top10
└── script
    ├── start-docker.sh
    └── xstartup
```

然后在终端运行 （这一步会很慢，可能需要数小时，网络问题自行更换镜像源）
```sh
# build kali with xfce desktop
docker build -f Dockerfile.xfce -t arwell/kali-desktop:xfce .
# build kali with xfce desktop and kali-tools-top10
docker build -f Dockerfile.xfce-top10 -t arwell/kali-desktop:xfce-top10 .
```

可以通过 `docker images` 查看构建的镜像。

构建完成后在终端运行容器
```sh
docker run --privileged --name kali-desktop-test -d arwell/kali-desktop:xfce
```

第一次运行会直接自动把屏幕输出重定向为 tty7 ，即显示 kali 的登录界面，
但是刚构建完的镜像运行除了root没有普通用户，
故无法登录。先通过快捷键 `ctrl + alt + F2` 回到宿主机的桌面，
运行 `docker exec -it kali-desktop-test bash` 以 root 身份进入容器的终端，
运行 `ls /home/` 可以验证容器确实没有普通用户，
然后运行 `adduser <user-name>` 来添加用户并根据提示设置密码，
完成之后再通过快捷键 `ctrl + alt + F7` 进入 kali 的登录界面，
通过刚刚设置的用户名和密码即可登录。

使用完成通过 `docker stop kali-desktop-test` 关闭容器，
下次启动容器运行命令 `docker start kali-desktop-test` 即可，
再次启动容器可能不会自动跳转到 kali 界面，
此时通过快捷键 `ctrl + alt + F7` 即可跳转到 kali 界面。

---

使用时发现在 kali 内无法以管理员身份运行命令，
提示 `*** is not in the sudoers file.`，
回到宿主机用命令 `docker exec -it kali-desktop-test bash` 以 root 身份进入容器，
运行命令 `usermod -a -G <user-name>` 将用户添加进 `sudo` 组即可。

---

参考 [wuchenlhy/kali-desktop](https://gitee.com/wuchenlhy/kali-desktop/tree/master)

使用该镜像搭建容器时发现其图形化界面会使用宿主机的 tty7 显示，
并且不需要 vnc 服务，于是删去了其 vnc 服务安装部分，
仅保留了 kali-desktop-xfce 和 kali-tools-top10(可选)。
