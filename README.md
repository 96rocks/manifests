Recommend build host is Ubuntu 16.04 64bit, for other hosts, refer official Android documents [https://source.android.com/setup/build/initializing Establishing a Build Environment].

### Install Repo
```bash
    $ wget 'https://storage.googleapis.com/git-repo-downloads/repo' -P /tmp/
    $ sudo cp /tmp/repo /usr/local/bin/repo
    $ sudo chmod +x /usr/local/bin/repo
```

For Chinese users or the countries the Google is blocked:

```bash
    $ echo "export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'" >> ~/.bashrc
    $ source ~/.bashrc
    $ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o /tmp/repo
    $ sudo cp /tmp/repo /usr/local/bin/repo
    $ sudo chmod +x /usr/local/bin/repo
```

### Init Environment
Android's source code primarily consists of Java, C++, and XML files.<br>
To compile the source code, you'll need to install OpenJDK 8, GNU C and C++ compilers, XML parsing libraries, ImageMagick, and several other related packages.
```bash
    $ apt-get update -y && apt-get install -y openjdk-8-jdk python git-core gnupg flex bison gperf build-essential \
           zip curl liblz4-tool zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
           lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
           libgl1-mesa-dev libxml2-utils xsltproc unzip mtools u-boot-tools \
           htop iotop sysstat iftop pigz bc device-tree-compiler lunzip \
           dosfstools vim-common parted udev lzop
```
Configure the JAVA environment
```bash
    $ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    $ export PATH=$JAVA_HOME/bin:$PATH
    $ export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

### DockerFile
```dockerfile
    FROM ubuntu:xenial
    RUN apt-get update -y && apt-get install -y openjdk-8-jdk python git-core gnupg flex bison gperf build-essential \
        zip curl liblz4-tool zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
        lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
        libgl1-mesa-dev libxml2-utils xsltproc unzip mtools u-boot-tools \
        htop iotop sysstat iftop pigz bc device-tree-compiler lunzip \
        dosfstools vim-common parted udev lzop

    RUN curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > /usr/local/bin/repo && \
        chmod +x /usr/local/bin/repo

    RUN which repo

    ENV REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/' USER=android8-docker

    ARG USER_ID=0
    ARG GROUP_ID=0
    RUN groupadd -g ${GROUP_ID} jenkins-docker && useradd -m -g jenkins-docker -u ${USER_ID} android8-docker

    USER android8-docker

```
Build DockerFile
```bash
    $ docker build -t android-builder:8.x --build-arg USER_ID=`id -u` --build-arg GROUP_ID=`id -g` $(which-dir-dockerfile-in)
```

### Download source code
```bash
    $ mkdir rockpi-n10-android8
    $ cd rockpi-n10-android8
```
Then run:
```bash
    $ repo init -u https://github.com/96rocks/manifests.git -b rk3399pro-android-8.1 -m rk3399pro-rk-vendor-release.xml
    $ repo sync -d --no-tags -j4
```
It might take quite a bit of time to fetch the entire AOSP source code(around 86G)!

### Build u-boot
```bash
    $ cd u-boot
    $ ./make.sh rk3399pro_dual
    $ cd ..
```
The generated images are **rk3399pro_loader_v_xxx.bin** , **idbloader.img** and **uboot.img**

### Building kernel
For HDMI 4K  
```bash
    $ cd kernel
    $ make rockchip_defconfig
    $ make rk3399pro-ficus2-android.img -j$(nproc)
    $ cd ..
```

The generated images are **kernel.img** and **resource.img**:
kernel.img, kernel with rkcrc checksum
resource.img, contains dtb and boot logo, Rockchip format resource package

### Building AOSP
Android Tablet:
```bash
    $ source build/envsetup.sh
    $ lunch rk3399pro-userdebug
    $ make -j$(nproc)
```
It takes a long time, take a break and wait...

### Generate  images
```bash
    $ ln -s RKTools/linux/Linux_Pack_Firmware/rockdev/ rockdev
    $ ./mkimage.sh ota
```

The generated images under rockdev/Image-rk3399pro are
```
    ├── boot.img
    ├── idbloader.img
    ├── kernel.img
    ├── MiniLoaderAll.bin
    ├── misc.img
    ├── oem.img
    ├── parameter.txt
    ├── ramdisk.img
    ├── recovery.img
    ├── resource.img
    ├── system.img
    ├── trust.img
    ├── uboot.img
    ├── update.img
    └── vendor.img
```

### Generated Image
```bash
    $ cd rockdev
    $ ln -s Image-rk3399pro Image
```

1. RkUpdate Image
```bash
    $ ./mkupdate.sh
```
The images under rockdev/ are `update.img`

### Installation

