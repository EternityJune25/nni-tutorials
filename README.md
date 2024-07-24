# nni-tutorials
NNI，或者神经网络智能（Neural Network Intelligence），是一个由微软开发的开源自动化机器学习（AutoML）工具包。它的目标是帮助开发者更轻松地进行模型调优和超参数搜索，以优化他们的机器学习模型。

关于安装和基本使用参考[这里](https://github.com/frinkleko/nni-pyg-example)感谢[@frinkleko](https://github.com/frinkleko)

本篇主要做出的贡献是服务器上如何使用NNI框架

服务器上安装NNI一般来说比较容易，当将NNI框架应用到自己的训练代码上的时候，一般的服务器会有如下报错

```
GLIBC_2.28’ not found` #也可能是其他版本或多个版本
```

### 分析原因

glibc 是 linux 底层的 API 库。通常情况下，有些环境需要 glibc 更高的版本才支持，比如 `GLIBC_2.28`。

### 经验与教训

使用 `GLIBC_xxx` 的源码包编译升级的惨案:

* 提醒：在其他博客教程上，有些网友(我也不另外,后面可拯救回来)就按照教程并使用 `GLIBC_xxx` 的源码包并去升级，结果往往是系统崩溃而告终。
* glibc 库对 linux 系统非常重要，轻易不要更换。如果需要更换，需提前备份好原本的相关库以防万一。
* 若在使用源码包去升级之后出现 `segmentation fault`,命令无法使用的情况。
* 解决方法：
  若安装失败，可能导致各指令出错，除了 cd、pwd 基本都不可使用，这时候千万不要关闭窗口(如果关闭将导致将无法打开，只能重装系统)，比如安装 libc-2.28.so 出错了，需拯救系统。可尝试输入其中一条

```
export LD_PRELOAD=/lib64/librt-2.XX.so
export LD_PRELOAD=/lib64/libm-2.XX.so
export LD_PRELOAD=/lib64/libpthread-2.XX.so
export LD_PRELOAD=/lib64/libc-2.XX.so
export LD_PRELOAD=/lib/x86_64-linux-gnu/libc-2.XX.so
```

(XX 指原本的版本，看文件夹有哪个就试一下)，然后 ls 这些指令就可以用了，再使用 ln -s 把以前的库链接回来。

```
cd /lib/x86_64-linux-gnu
ll     # 文件详细信息ln -sf libc-2.27.so libc.so.6   # libc-2.27.so是原有版本
rm  libc-2.28.so     #删除
```

### 推荐的解决方法

`1` 查看服务器当前版本，命令如下：

```
strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC_
```

返回的结果如下：

```
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_2.18
GLIBC_2.22
GLIBC_2.23
GLIBC_2.24
GLIBC_2.25
GLIBC_2.26
GLIBC_2.27
GLIBC_PRIVATE
```

说明服务器当前是没有 GLIBC_2.28

`2` 使用软件包升级方式

参考 debian 网址并搜索想要的软件或者工具等，如 `libc6`

* 添加软件源，`/etc/apt/sources.list` 文件中像下面这样添加一行：

```
deb http://security.debian.org/debian-security buster/updates main
```

* 系统可用的软件包更新，刷新软件包的缓存

```
sudo apt update  # 更新软件源
```

* `apt-get update` 之后若出现下面提示：
  `由于没有公钥，无法验证下列签名： NO_PUBKEY 112695A0E562B32A NO_PUBKEY 54404762BBB6E853`

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 112695A0E562B32A 54404762BBB6E853
```

* 其中后面的 `112695A0E562B32A 54404762BBB6E853` 就是上面提到的 `NO_PUBKEY 112695A0E562B32A NO_PUBKEY 54404762BBB6E853` 中的公钥，替换成对应的即可。然后重新 `apt-get update` 即可。
* 查看软件包可更新列表

```
sudo apt list --upgradable
```

* 安装 libc6

```
sudo apt install libc6-dev  /sudo apt install libc6
```

`3` 查看服务器当前版本：

```
strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC_
```

返回的结果如下：

```
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_2.18
GLIBC_2.22
GLIBC_2.23
GLIBC_2.24
GLIBC_2.25
GLIBC_2.26
GLIBC_2.27
GLIBC_2.28     # 多出该版本，说明安装成功，系统也能正常使用。
GLIBC_PRIVATE
```

至此为止 解决完毕。

### 基本操作

最后推荐几个常用命令和基本操作：

**配置完毕之后启动训练**：

```
nnictl create --config config.yaml --port 1234
```

**将服务器web端映射到本机**：

如下代码将web界面映射到本机8080端口

```
ssh -L 8080:localhost:服务器开放端口号 用户名@服务器ip
ssh -p 24144 -L 8081:localhost:1234 root@connect.yza1.seetacloud.com
```
**报告效果**：
```
nni.report_intermediate_result(acc)
nni.report_final_result(ari2)
```
**将完成的实验映射到本机**：

有个相应的端口，完成的实验会一直占用端口，做相应的映射即可打开web界面。

**暂停实验(端口不再监听) 建议使用直接停止某次实验**：

```
nnictl stop
nnictl stop [experiment_id]
nnictl stop --port 8080
```
**查看所有实验（正在进行中的）**：

```
nnictl experiment list
```
**恢复某次实验**：

```
nnictl resume [experiment_id] --port 8088
```
