###Linux

#####安装Linux

1. 安装VMware

2. 安装Ubuntu
	默认root密码是随机的，通过命令sudo passwd 设置root密码

3. 安装VMwareTool（有许多功能，适配分辨率）


#####同步Android代码
https://source.android.com/setup/build/downloading


##### 编译Android代码
1. source build/envsetup.sh
2. 

#####导入AndroidStudio


#####不编译导入AndroidStudio



#####添加环境变量

1. 临时方案
> export //查看PATH值
> echo $PATH //显示PATH环境变量
> export PATH=/opt/STM/STLinux-2.3/devkit/sh4/bin:$PATH 添加路径到环境变量，作用只能在当前terminal，关闭后无效
(临时添加)


2. 永久方案：

>修改vim /etc/profile文件 （系统shell）或者 vim 当前目录下的/.bashrc文件（用户shell）添加export PATH=/opt/STM/STLinux-2.3/devkit/sh4/bin:$PATH就可以永久有效

需要是用source /etc/profile 或者 . /etc/profile (同理bashrc) 保存配置

> 修改/etc/environment文件
直接添加即可

#####