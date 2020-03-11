[参考：](https://blog.csdn.net/ichuzhen/article/details/8434131)
在linux使用过程中遇到 is not in the sudoers file 时的解决办法。
用sudo时提示"xxx is not in the sudoers file. This incident will be reported.其中XXX是你的用户名，也就是你的用户名没有权限使用sudo，我们只要修改一下/etc/sudoers文件就行了。

例子：

user@pc:~$ sudo add-apt-repository ppa:stk/dev
[sudo] password for user:
user is not in the sudoers file.  This incident will be reported.
user@pc:~$

下面是解决方法：

1）进入超级用户模式。也就是输入"su -",系统会让你输入超级用户密码，输入密码后就进入了超级用户模式。（当然，你也可以直接用root用）
(注意有- ，这和su是不同的，在用命令”su”的时候只是切换到root，但没有把root的环境变量传过去，还是当前用户的环境变量，用”su -”命令将环境变量也一起带过去，就象和root登录一样)
2）添加文件的写权限。也就是输入命令"chmod u+w /etc/sudoers"。
3）编辑/etc/sudoers文件。也就是输入命令"gedit /etc/sudoers",进入编辑模式，找到这一 行："root ALL=(ALL) ALL"在起下面添加"www_linuxidc_com ALL=(ALL) ALL"(这里的xxx是你的用户名)，然后保存退出。
4）撤销文件的写权限。也就是输入命令"chmod u-w /etc/sudoers"