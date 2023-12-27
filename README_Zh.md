LineageOS
===========

# 快速开始

⚠️注意：不要使用root权限的账户操作⚠️

要开始使用Android/LineageOS，您需要熟悉 [Source Control Tools](https://source.android.com/setup/develop).

## 配置系统代理（可选，根据个人情况使用）：

在国内环境不太容易下载完整的数据，建议全程挂科学上网下载。
```
export IP=127.0.0.1
export PORT=7890
export http_proxy=http://${IP}:${PORT}
export https_proxy=http://${IP}:${PORT}
export ALL_PROXY=http://$IP:${PORT}
git config --global http.proxy http://${IP}:${PORT}
git config --global https.proxy http://${IP}:${PORT}
```

## 配置环境和工具
```
sudo apt update -y
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev vim -y
```

由于Ubuntu下的Python3默认不创建python链接，这里手动创建一个：
```
sudo ln -s /usr/bin/python3 /usr/bin/python
```

您需要在构建环境中设置一些目录。

`~/bin`目录将包含 git-repo 工具（通常称为“repo”）；`~/android/lineage`目录将包含 LineageOS 的源代码。
```
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

从 Google 下载[platform tools](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)，并解压它们(指令自动完成)：
```
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip -O ~/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip -d ~
rm ~/platform-tools-latest-linux.zip
```

安装repo命令，输入以下内容以下载repo二进制文件并赋予可执行权限：
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

现在您必须将adb和添加fastboot到您的环境变量中。打开~/.profile并添加以下内容：

```
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

然后，运行以下指令更新您的环境变量。
```
source ~/.profile
```

鉴于repo需要您验证自己的身份才能同步 Android，请运行以下命令来配置您的git身份：
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

由于其大小，一些存储库配置为lfs或Large File Storage。为了确保您的发行版为此做好准备，请运行：
```
git lfs install
```

打开缓存以加快构建速度
```
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
```

## 初始化代码

像通常对LineageOS所做的那样初始化本地存储库：
```
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-20.0 --git-lfs --depth=1
```
要为`Oneplus ACE 2 Pro`构建，您需要特定的存储库。使用以下命令同步它们：
```
mkdir .repo/local_manifests && wget https://raw.githubusercontent.com/jiongjiongJOJO/lineageos_android_xigua/lineage-20-test/xigua.xml -O .repo/local_manifests/xigua.xml
```
然后进行同步：
```
repo sync
```
Do not forget to `git lfs pull`, just in case:
```
repo forall -c 'git lfs pull'
```


## Oneplus ACE 2 Pro的构建

要为`Oneplus ACE 2 Pro`构建LineageOS，您需要使用一些命令。

使用以下命令加载构建环境：
```
source build/envsetup.sh
```

[//]: # (Use the `repopick` command to cherry-pick `kalama` specific fixes:)

[//]: # (```)

[//]: # (repopick -t 13-taro-kalama)

[//]: # (```)
您现在可以开始构建了：
```
lunch lineage_xigua-eng
m
```


# 刷入镜像

[//]: # (Our work is currently based on the CPH2449_A25 firmware. If you're not running A25, flash it and boot it at least once before flashing your build. &#40;[CPH2449_A25 Full OTA]&#40;https://gauss-componentotacostmanual-eu.allawnofs.com/remove-28f8287dcc47dbfd64d1cfb31f877f75/component-ota/23/08/28/9d93d654eec1429684aec3692137c786.zip&#41;&#41;)

如果你以前没有解锁引导程序（BL锁），请解锁它
```
fastboot flashing unlock
```
将手机置于引导加载模式：
```
fastboot -w
fastboot reboot fastboot
```
一旦您的设备处于引导加载模式，请使用`cd`命令导航到构建文件夹，很可能是`out/target/product/xigua`，然后运行：
```
# Replace with your path
export ANDROID_PRODUCT_OUT=~/android/LineageOS/out/target/product/xigua
fastboot flashall
```
如果您无法使用构建机器闪存设备（例如，如果您使用远程服务器构建LineageOS），请将`out/target/product/xigua`中的所有文件复制到本地机器（无需复制文件夹），并使用所有这些命令刷入镜像：
```
# Replace with your path
export ANDROID_PRODUCT_OUT=/Users/dekefake/Downloads/LineageOS_images
fastboot flashall
```
如果您想在刷入LineageOS后立即刷入Magisk，请在LineageOS的recovery中使用`fastboot flashall--skip reboot`，然后使用`adb sideload`刷入Magisk。


# 更新固件
要更新您的设备，请下载完整的OTA软件包，并使用[payload dumper go](https://github.com/ssut/payload-dumper-go)提取固件映像。然后，将您的设备引导到FastbootD模式，并刷入以下镜像：
```
fastboot flash --slot=all abl abl.img
fastboot flash --slot=all aop_config aop_config.img
fastboot flash --slot=all aop aop.img
fastboot flash --slot=all bluetooth bluetooth.img
fastboot flash --slot=all cpucp cpucp.img
fastboot flash --slot=all devcfg devcfg.img
fastboot flash --slot=all dsp dsp.img
fastboot flash --slot=all engineering_cdt engineering_cdt.img
fastboot flash --slot=all featenabler featenabler.img
fastboot flash --slot=all hyp hyp.img
fastboot flash --slot=all imagefv imagefv.img
fastboot flash --slot=all keymaster keymaster.img
fastboot flash --slot=all modem modem.img
fastboot flash --slot=all oplus_sec oplus_sec.img
fastboot flash --slot=all oplusstanvbk oplusstanvbk.img
fastboot flash --slot=all qupfw qupfw.img
fastboot flash --slot=all shrm shrm.img
fastboot flash --slot=all splash splash.img
fastboot flash --slot=all tz tz.img
fastboot flash --slot=all uefi uefi.img
fastboot flash --slot=all uefisecapp uefisecapp.img
fastboot flash --slot=all xbl_config xbl_config.img
fastboot flash --slot=all xbl_ramdump xbl_ramdump.img
fastboot flash --slot=all xbl xbl.img
```

# Q&A

Q:
```
Output:
ccache: error: Failed to create directory /home/jojo/.ccache/tmp: Read-only file system
\nWrite to a read-only file system detected. Possible fixes include
1. Generate file directly to out/ which is ReadWrite, #recommend solution
2. BUILD_BROKEN_SRC_DIR_RW_ALLOWLIST := <my/path/1> <my/path/2> #discouraged, subset of source tree will be RW
3. BUILD_BROKEN_SRC_DIR_IS_WRITABLE := true #highly discouraged, entire source tree will be RW
```

A:
取消缓存加速，因为上面是通过export设置的，所以直接关闭终端，重新进入目录，再次重新编译即可（注意之前设置的export参数都将失效，例如`source build/envsetup.sh`等，都需要重新设置）
