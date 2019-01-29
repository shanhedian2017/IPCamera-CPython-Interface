
# IPCamera-CPython-Interface-v3

## 这是什么东西（What is this）

> * 兼容主流海康和雄迈IPC的适用于C++和python的帧数据获取接口
> * 通过python进行海康（雄迈）IP相机连接、登录、注册回调、解码（FFMPEG）、取帧
> * 支持python2和python3（编译时修改CMAKE的`TARGET_PYTHON_VERSION`）

## 原理简要说明（Brief description of the principle）

流程图
架构图
文字说明
xm_interface_v4/CWrapper/build/
xm_interface_v4/demo_c_cpp/build/

## 取帧方法（The way to get frame）

```python
from pycext import IPCamera # 引用`pycext.so`
cp = IPCamera("ip", port, "username", "password", "") # port为整型数
cp.PrintInfo() # 可选：打印信息
cp.login() # 登录
cp.open() # 注册回调

time.sleep(2) # 等待第一个I帧处理完毕 推荐等待不小于1秒

while True: # 如果CPU处理能力很强 建议设置循环等待间隔
    frame = np.asarray(cp.queryframe('array')).reshape(1080,1920,3)
    # 可选择安装：`opencv-python` 和 `opencv-contrib-python`
    cv2.imshow("OKOK", frame)
    cv2.waitKey(1)
```

## 配合内存管道推流（python + ffmpeg -> rtmp）

```python
# 首先配置命令如下
command = ['ffmpeg',
    '-y',
    '-f', 'rawvideo',
    '-vcodec','rawvideo',
    '-pix_fmt', 'bgr24',
    '-s', '1920x1080',
    '-i', '-',
    '-c:v', 'libx264',
    '-pix_fmt', 'yuv420p',
    '-preset', 'ultrafast',
    '-f', 'flv',
    'rtmp://10.41.0.147:1935/hls/livestream']

# 初始化子进程
import subprocess as sp
proc = sp.Popen(command, stdin=sp. PIPE, shell=False)

# 帧数据写入内存管道
while True:
    frame = np.asarray(cp.queryframe('array')).reshape(1080,1920,3)
    proc.stdin.write(frame.tostring())
```

## 环境要求（the environment require）

> 尽管本repo提供基于以下环境的预编译so文件
* `Ubuntu 18.04.1 LTS`（国内镜像）
* `Python 3.6.X`（apt下载）
* `Opencv 3.4.X`（源码编译）
* `FFmpeg 3.4.X`（源码编译）

> 但还是建议额外安装以下依赖环境自行编译
* `numpy-dev`
* `python3-dev/python-dev`
* `Cmake`
* `gcc/g++`
* `libx264-dev`
* [海康SDK-版本号: V5.3.5.2 build20171124](http://www.hikvision.com/cn/download_more_403.html "Title")
* [雄迈SDK-更新日期: 2018-06-26](https://download.xm030.cn/d/MDAwMDA3MzM "Title")

## 目录结构说明（Files-Tree）

* hk_interface - 存放海康接口源码、测试demo、SDK
* xm_interface - 存放雄迈接口源码、测试demo、SDK
* 每个关键子目录下都附带README文件

## 注意事项（Cautious）

* 使用内存管道方法配合`nginx_rtmp_module`推流延迟较高，需要低延迟推流可以考虑使用[基于类MJPEG协议的推流框架](...)
* v3版本是针对`ffmpeg 3.X`和`opencv 3.X`设计的封装方案，且c++和python封装为两个独立的工程，二者API实现方式存在细微差别，现已不再维护，推荐使用[主分支上的新版本](https://github.com/1996scarlet/IPCamera-CPython-Interface)
* 编译安装`ffmpeg`前需要设置`./configure --enbale-shared`来防止`opencv`编译过程无法引用动态库导致的`video.so`相关错误
* 编译安装`opencv`时若出现`xfeatures2d`相关错误，需要[重新下载`curl`](https://curl.haxx.se/download.html)并按照如下步骤编译安装
    1. cd /root/Downloads/curl
    2. ./configure --with-ssl
    3. make
    4. sudo make install
* 在Ubuntu 18.04上安装openCV可以参考[这个链接](https://www.pyimagesearch.com/2018/08/15/how-to-install-opencv-4-on-ubuntu/)
