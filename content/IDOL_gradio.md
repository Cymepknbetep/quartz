### **1. 代码概述**

- **目的**：构建一个Gradio Web界面，允许用户上传一张人体图像（带透明背景或适合背景移除）和一个目标运动视频（对应SMPLX参数），利用IDOL模型生成动画视频。

- 主要组件

  ：

  - **IDOL模型**：用于从单张图像生成3D人体并应用运动序列的神经网络模型。
  - **Gradio界面**：提供用户交互界面，包含输入图像、目标运动视频选择、处理后的参考图像和输出视频展示。
  - **SMPLX参数处理**：从运动视频中提取或使用预定义的SMPLX参数（包括位姿、形体参数等）。
  - **背景移除与预处理**：使用rembg或其他工具移除图像背景，并对图像进行标准化处理。
  - **面部精修**：通过调用facefusion工具对生成视频的面部进行优化。
  
- **运行流程**：

  1. 用户上传图像和选择运动视频。
  2. 预处理输入图像（背景移除、调整大小等）。
  3. 从运动视频或预定义路径加载SMPLX参数。
  4. 运行IDOL模型推理，生成动画帧。
  5. 保存输出视频并可选地进行面部精修。
  6. 展示结果并清理临时文件。



##### 一些环境配置问题

* 可以将conda的环境安装到某个特定范围方便统一管理```conda install -p(or --prefix) \path```

* 目前看来似乎只有facefusion需要额外安装，

* 安装某个范围内的包：```pip install "numpy>2.2,<2.3"```

* 编译环境之前需要注意包括C++等环境的配置

* 可以自己调整pytorch的下载命令

*  ```
   pip3 install torch==1.6.0 torchvision==0.7.0 --index-url https://download.pytorch.org/whl/cu101
   ```

* ```pip3 install torch==2.8.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126```

* 主要是记住https://download.pytorch.org/whl/cu1xx

* 虚拟环境同时使用虚拟的CUDA

  *  conda install -c nvidia cuda-toolkit=12.1 -y 
  *  -c means use specified channels
  *  pip install nvidia-cudnn-cu12==9.1.0.70  # 替换为最新 9.x 版本，从 NVIDIA 官网下载 wheel
  *  conda install -c nvidia cuda-toolkit=12.1 cudnn=9.10
  *  conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia

* facefusion主要问题
   * 设置install时numpy>1.23 但是没有设置上限导致pip直接拉取最新numpy导致不兼容，这是代码installer脚本的问题
   
   *  环境很容易损坏？
      *  ~~$Env:CONDA_NO_PLUGINS = "true"~~禁用之后仍然无法激活，所以肯定是损坏了
      
   * Extracting video failed： 注意ffmpeg在对应的环境时候正确安装，库是否
   
   * Processing: 100%|=| 283/283 [00:05<00:00, 53.14frame/s, execution_providers=['cu
      [FACEFUSION.CORE] Merging video with a resolution of 768x1080 and 30.0 frames per second
      Merging:   0%|                                       | 0/283 [00:00<?, ?frame/s][FACEFUSION.FFMPEG] Unrecognized option 'crf'.
      [FACEFUSION.FFMPEG] Error splitting the argument list: Option not found
      Merging:   0%|                                       | 0/283 [00:00<?, ?frame/s]
      [FACEFUSION.CORE] Merging video failed另一个新的报错，
   
   * 
   
   * python facefusion.py run --open-browser --log-level debug来查看详细日志
   
      * [FACEFUSION.CORE] Merging video with a resolution of 576x1024 and 30.0 frames per second
         Merging:   0%|                                                                                                                                                    | 0/592 [00:00<?, ?frame/s][FACEFUSION.FFMPEG] Unrecognized option 'preset'.
         [FACEFUSION.FFMPEG] Error splitting the argument list: Option not found
         Merging:   0%|                                                                                                                                                    | 0/592 [00:00<?, ?frame/s]
         [FACEFUSION.CORE] Merging video failed
   
      * 显示不支持-preset，通过检查发现conda环境中的ffmpeg不支持h264
   
      * ffmpeg -f lavfi -i testsrc=duration=1:size=576x1024:rate=30 -c:v libx264 -preset medium -pix_fmt yuv420p /media/easy/Data/test_output.mp4用来检验是否支持h264
   
      * 然而删除环境中的ffmpeg之后会使用系统ffmpeg，但是虚拟环境的环境变量优先级更高导致系统ffmpeg到conda环境找包而找不到
   
      * 虽然最优先最独立的方案应该是使用conda环境中独立的ffmpeg，但是condaforge频道中的ffmpeg似乎不包含支持h264的ffmpeg
   
      * 尝试使用系统中的ffmpeg支持h264
   
         - **现状**：conda 环境激活时会覆盖 LD_LIBRARY_PATH，将 conda 的库路径（如 /home/easy/env/facefusion/lib）置于优先级最高，导致系统 ffmpeg 加载了不兼容的 conda 库（如 libgcc_s.so.1 缺少 GCC_12.0.0，libstdc++.so.6 缺少 GLIBCXX_3.4.32）。
   
         - 需求
   
           ：
   
           1. 保留 conda 环境中的 CUDA 设置（例如 /home/easy/env/facefusion/lib 中的 CUDA 库）。
           2. 确保系统 ffmpeg 使用系统的 libgcc_s.so.1、libstdc++.so.6 等库，而不是 conda 的版本。
   
         - **挑战**：conda 的 LD_LIBRARY_PATH 覆盖可能导致系统库被忽略。
   
         - 更改环境变量仍然读取不到正确的lib，很奇怪，export LD_LIBRARY_PATH=''清空，似乎也不影响cuda的读取和使用，相当奇怪（可能是系统版本一致把）
   
         - 一些命令：~~export LD_LIBRARY_PATH=/usr/lib:/usr/lib64:/usr/local/lib:/home/easy/env/facefusion/lib:$LD_LIBRARY_PATH~~不起作用
   
* gradio部分主要问题：
   *  使用install_cu121_fixed.bat的相关流程
   *  编译时需要对应版本的cuda，需要提前设置，下面命令临时设置cuda版本
      *  set CUDA_HOME=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1
   *  **MSVC 版本过新**：
      - 您的 MSVC 版本（19.44.35215）可能来自 Visual Studio 2022 的最新更新（例如 17.14 或更高），而 CUDA 12.1 的 nvcc 不支持高于 2022 的某些次要版本。
      - CUDA 12.1 官方支持的 MSVC 版本通常为 19.14 至 19.32（对应 Visual Studio 2017 到 2022）。
      - 推荐直接使用VS2019，在安装器中直接选择安装MSVC142并且修改环境变量
      - C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.44.35207\bin\Hostx64\x64
      - C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.29.30133\bin\HostX64\x64
      - C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.29.30133\bin\HostX64\x64
      - 注意草台班子微软
      - 新问题：nvcc仍然遵循默认vs的设置而无法调用MSVC142
      - ~~VCToolsInstallDir = C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.29.30133\~~ 没用
      - 使用x64 Native Tools Command Prompt （安装新的vs2019）但是出现新的问题：
        - set DISTUTILS_USE_SDK=1
        - 成功！！！

* facefusion 不调用gpu的问题：

  * https://www.bilibili.com/opus/1065609071498887190

  * 主要就是ONNX Runtime、cuda、cudnn三个版本需要相互匹配。

    先附上一个表，https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html；

  * 通过python的：```import torch
    print(torch.backends.cudnn.version())```确认了cudann为8801即8.801（cuda12.1的

  * 最终发现是requirements中的Onnxruntime版本为1.20.1而不是1.17，因此无法支持低版本的cudann

  * ImportError:
    A module that was compiled using NumPy 1.x cannot be run in
    NumPy 2.2.0 as it may crash. To support both 1.x and 2.x
    versions of NumPy, modules must be compiled with NumPy 2.0.
    Some module may need to rebuild instead e.g. with 'pybind11>=2.12'.

    If you are a user of the module, the easiest solution will be to
    downgrade to 'numpy<2' or try to upgrade the affected module.
    We expect that some modules will need time to support NumPy 2.

    不支持numpy2————原因是onnxruntime1.17基于numpy1构建，因此为了使用onnxruntime1.20需要升级cudann到9.x，而cudann的9.x理论上支持cuda12.1

  * **https://developer.nvidia.com/rdp/cudnn-download** 这里是cudann9.13不知道能不能用于cuda12.1

  * 官网驱动安装的cudnn9.13复制到cuda12.1没有任何作用，因此考虑还是降级到numpy1.xx

  * pip index versions numpy | findstr 1.2，同时升级onnxruntime到1.18

  * 这套配置（onnxruntime1.18+numpy1.26.4可行但是还是没有gpu调用）

    * conda迷踪之设置了cuda和cudann的虚拟环境，例如```conda install nvidia/label/cuda-12.9.1::cuda-runtime nvidia/label/cudnn-9.10.0::cudnn```但是nvcc看不到	==一定得注意加上label来指定兼容的cuda==

    * conda install -c nvidia/label/cuda-11.8.0cuda-toolkit=11.8 cudnn=8.9

  * 破案了：尽管我实在没办法解决一使用官方的脚本就坏环境的问题，但是完全能够通过手动安装环境解决

  * 首先注意conda安装的cuda和cudnn将会作用于虚拟环境中，要考虑这个来配置onnxruntime
  
  * 其次是具体的requirements：```gradio-rangeslider==0.0.8
    gradio==5.42.0
    numpy==2.2.6
    onnx==1.18.0
    onnxruntime==1.22.0
    opencv-python==4.12.0.88
    psutil==7.0.0
    tqdm==4.67.1
    scipy==1.16.1
    onnxruntime-gpu==1.22.0```

  * 然后是：注意学长的环境中已经缓存有部分文件，此时新环境会出现无法调用gpu的问题
  
  * 

##### 换脸算法的调研

* 速度
  * load models 1s~10s (机械硬盘)
  * 768*1080
  * hyperswap_1a_256 ： 27frames/s
  * ghost3_256   24frames/s
  * hififace_unofficial_256  28frame/s

* 需求：3-4s完成，需要足够建立3dGS的帧数
* 问题：这些算法可能在复杂光照或角度下效果不佳，输出需后处理

* DeepFaceLive
  * 专注实时，例如直播应用
* DeepFaceLab
  * 专业级Deepfake框架，专注于视频换脸，支持高分辨率和精细控制。社区活跃，有预训练模型。
* FaceFusion
  * 行业领先的面部操纵平台，支持实时预览和批量处理。集成GFPGAN等增强模型
* SimSwap
  * An arbitary face-swapping framework 任意人脸之间的交换
  * 使用高分数据集VGGFace2-HQ，可能能够实现高质量的换脸结果
* Roop
  * 一键换脸工具，只需一张目标脸图像，无需数据集或训练。简单快速，适用于短视频。
* FaceSwap
  * 需要训练模型，易于自定义但是训练耗时较多
  * 综合的换脸框架，提供多种算法和神经网络模型，并提供后处理支持
* First Order Motion Model
  * 动作驱动的
* InsightFace
* GHOST
  * conda install -c nvidia cuda-toolkit=12.1 -y
  * 非常容易出现版本漂移，因为nvidia鼓励使用最新的cuda
  * ~~conda install -c nvidia/label/cuda-12.1.1 cuda-toolkit -y~~
  * ~~conda config --add channels nvidia/label/cuda-12.1.1~~
  * 记得去掉这个channel
    * conda config --show channels
    * conda config --remove channels nvidia/label/cuda-12.1.1

  * conda remove cuda-toolkit --all -y
  * pip install nvidia-cudnn-cu12==9.10.0.56

* 测试使用数据：
  * 人脸使用各名人照片
  * 视频使用voxceleb2（面部说话动作）/MEAD（情感）/跳舞（motion）






庄哥我想明确一下项目的流程和目标。项目想要使用单图，通过IDOL重建模式生成一个使用（视频估计的）SMPLX的参数驱动的人体模型，再使用渲染器渲染，而difix和faceswap用于给IDOL的输出进行优化。

但是我不是很清楚这两个用于图生图的模型要如何用来优化这个可以用于渲染的人体模型呢。还是说IDOL输出的模型先渲染成视频给两个模型分别优化这个视频之后再估计成模型使用SMPLX驱动？

另外就是目前试了facefusion只要导入结束了之后在处理的步骤还是相当快的，一秒有几十帧，不过这个的环境好像使用的是适合生产环境的onnx，对cuda、cudnn的支持也比较奇怪。目前任务需要去看不同的换脸算法还是可以先看看facefusion能不能兼容已有的流程和环境。



##### facefusion流程理解

* facefusion.py->core.py cli() (项目主函数，解析命令行)

##### facefusion细节学习笔记

* ```    signal.signal(signal.SIGINT, signal_exit)  # 优雅的退出方式```
* 使用`ArgumentParser`对象创建命令行解析对象，并可自定义
  * 其中分为主解析器（解析参数作用全局），子解析器（参数作用对应子命令），但是注意子解析器通过add_subparsers添加，
  * 解析器都为同一个类
  * parent参数保持简洁性
* isinstance(a,list)检查是否是list
* facefusion似乎在ubuntu下性能明显优于windows，同时考虑SSD和HDD是否有相当影响性

* ll dir | grep ^- | wc l 计算普通文件数量

##### ssh 相关问题

*  端口转发
* 简单地生成密钥对：```ssh-keygen -t rsa -b 4096 -C "cymepknbetep@outlook.com"```
  * 简单来说就是指定rsa算法、4096长度、备注邮箱
* ```ssh-copy-id easy@114.212.166.199``` 自动copy密钥，注意可能需要安装ssh-copy-id
* 否则可以直接复制pub公钥

##### 代理问题

为了方便更改代理模式：

- env | grep proxy
- export all_proxy = ﻿[http://proxy_ip:port](http://proxy_ip:port/)﻿
- 设置socks代理：export all_proxy = socks5://proxy_ip:port
- proxy主要是http/https/ftp
- 默认pip、apt、git都走https或http协议，一般能够自动识别系统代理，可能需要修改对应工具的配置文件 
- 在设置了系统级别代理时，ssh也能够直接使用代理，通过该命令查询gnome的系统代理
- `gsettings get org.gnome.system.proxy mode`查看是否设置了手动代理
- 临时禁用系统代理：`sudo gsettings set org.gnome.system.proxy mode 'none'`
- 临时恢复代理`sudo gsettings set org.gnome.system.proxy mode 'manual'`
- 以上对于ssh可能无用，以上为gnome使用
- 手动为不同工具启动/禁用代理：
  - curl --noproxy "*" ﻿[https://www.google.com](https://www.google.com/)﻿	忽略所有的代理
  - git -c http.proxy= -c https.proxy= clone ﻿https://github.com/﻿﻿/﻿.git  主要是设置两个代理为空
  - sudo apt -o Acquire::http::Proxy=false -o Acquire::https::Proxy=false update
  - pip install <package_name> --no-proxy
  - wget --no-proxy ﻿https://example.com/file

##### Ubuntu中某些奇怪问题

* 硬盘挂载问题
  * lsblk确定硬盘情况
  * df -h 查看挂载硬盘情况
  * blkid查看已挂载硬盘的uuid和文件系统类型
  * /etc/fstab中存储了硬盘自动挂载的相关信息
  * 处理挂载时需要保证挂载点存在，例如先创建文件夹，同时使用```sudo chown easy:easy /dir```确保权限
  * ~~如果想要修改fstab时不知道硬盘的uuid，可以先使用mount来挂载~~，例如```sudo mount /dev/sda1 /dir```，需要注意的时sda1这个硬盘符可以在lsblk中查看到，通常来说推荐添加-t ntfs，不过一般能够自动识别成功。mount | grep /media/username/Data来确认是ntfs挂载
  * 事实上blkid不显示该外挂硬盘是因为没有使用sudo权限/dev/sda1: LABEL="Data" BLOCK_SIZE="512" UUID="CA642CED642CDDC7" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="38598b3b-cb54-430c-bb45-40ac64617768"
  * 修改fstab之前继续确认用户id
  * easy@easy-Z890-GAMING-X-WIFI7:~$ id
    uid=1000(easy) gid=1000(easy) 组=1000(easy),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)
  * UUID=CA642CED642CDDC7 /media/easy/Data ntfs~~-3g~~ defaults,uid=1000,gid=1000 0 0 
  * 注意：在挂载之后可能没有写的权限，修改为：```UUID=CA642CED642CDDC7 /media/easy/Data ntfs-3g defaults,uid=1000,gid=1000,rw 0 0 ```显式指定读写
  * 注意较新的内核集成了ntfs3，因此改为ntfs
  * 运行完可以使用systemctl daemon-reload来通知systemd重新加载其配置文件
  * 一些命令
    * sudo umount /dev/xxx
    * sudo mount -a(应用fstab)
    * systemctl daemon-reload

##### 关于ffmpeg的一些问题：

* 

##### IDOL项目本身的问题：

* CUDA OUT of Memory：缩小patch

  * \# supportted by torch,to make use of memory

    import os

    os.environ["PYTORCH_CUDA_ALLOC_CONF"] = "expandable_segments:True"

* 另外的就是可能的包版本更新带来的兼容性问题：save_video函数添加codec参数指定h264编码

* Pyav的版本问题？

  * 似乎目前不是pyav与ffmpeg的兼容问题

  * 可能是与numpy2的兼容问题

  * ``````
    
    # 更新包列表
    sudo apt update
    
    # 安装 FFmpeg 开发库
    sudo apt install -y \
        libavformat-dev \
        libavcodec-dev \
        libavdevice-dev \
        libavutil-dev \
        libavfilter-dev \
        libswscale-dev \
        libswresample-dev \
        pkg-config
    
    # 同时安装编译工具
    sudo apt install -y \
        python3-dev \
        build-essential \
        pkg-config
    ``````

  * 

##### ubuntu问题

* 可以使用rsync代替cp，通过添加--progress、-a(归档)-v(详细输出)-h(人类可读MB/s)方式来复制目录
* iostat查看硬盘I/O情况
* iotop监控进程I/O
* 显示目录的占用使用du dir -s为汇总，-h人类易读
* tar -czf -c创建新的归档tar z使用gzip压缩.gz，f指定输出文件名
* 

##### 环境配合问题:

* 改了库之后似乎能跑但是idol这边onnx报错：

* 2025-09-25 19:09:59.258217426 [E:onnxruntime:Default, provider_bridge_ort.cc:2167 TryGetProviderInfo_TensorRT] /onnxruntime_src/onnxruntime/core/session/provider_bridge_ort.cc:1778 onnxruntime::Provider& onnxruntime::ProviderLibrary::Get() [ONNXRuntimeError] : 1 : FAIL : Failed to load library libonnxruntime_providers_tensorrt.so with error: libnvinfer.so.10: cannot open shared object file: No such file or directory

  *************** EP Error ***************
  EP Error /onnxruntime_src/onnxruntime/python/onnxruntime_pybind_state.cc:505 void onnxruntime::python::RegisterTensorRTPluginsAsCustomOps(PySessionOptions&, const onnxruntime::ProviderOptions&) Please install TensorRT libraries as mentioned in the GPU requirements page, make sure they're in the PATH or LD_LIBRARY_PATH, and that your GPU is supported.
   when using ['TensorrtExecutionProvider', 'CUDAExecutionProvider', 'CPUExecutionProvider']
  Falling back to ['CPUExecutionProvider'] and retrying.

  但是似乎仍然继续成功运行

  ****************************************

* python facefusion.py run --open-browser --log-level debug
* facefusion这边成功运行无报错
* 总结：先进行idol的install然后进行facefusion的install
* 为了解决上方的问题，因为我是使用了系统默认的环境变量LD_LIBRARY_PATH，所以我们给系统装一个tensorRT即可
* 非常奇怪的是之前也没有安装tensorrt，是怎么没有出现这个报错的呢？
* onnx-runtime为1.22.1/1.22.0（onnxruntime-gpu），推荐TensorRT10.x

* cuda-toolkit\==12.9.1,cudnn\==9.13.0.50,onnxruntime-gpu\==1.22.0,torch==2.8.0+cu129，



##### 机械硬盘和SSD区别

| ///     | 模型HDD                                       | 模型SSD                                           |
| ------- | --------------------------------------------- | ------------------------------------------------- |
| 环境HDD | `real 3m31.392s user 1m18.295s sys 0m18.305s` | `real 1m41.638s    user 1m5.811s   sys 0m17.794s` |
| 环境SSD |                                               |                                                   |





##### 环境配置

* 非常诡异：pytorch3d要求的torch<2.4.0,推荐2.3.1但是一开始的2.8.0torch也装成功了现在却不行

* 暂且认为是缺少CUB库
  * 下载解压后用export CUB_HOME=~/cub
  * 确认后使用echo 'export CUB_HOME=~/cub' >> ~/.bashrc添加到bashrc
  * source ~/.bashrc应用
* 没有用，所以从头开始：
  * 先确定系统驱动：570支持cu128
  * sudo apt search nvidia-driver，最新为580
  * sudo apt install nvidia-driver-580-server 选择服务器驱动，计算支持性更好
* 破案：重装miniconda，可能之前找不到torch也是环境变量被污染了
* 编译仍然出现问题，安装缺少的gccg++
* 编译继续出现问题，发现环境变量被污染TORCH_CUDA_ARCH_LIST=7.5;8.0;8.6;8.9;9.0;10.0;10.3;12.0;12.1+PTX
* 实际上我的架构是ampere 8.6，使用export设置成8.6
* pip install --extra-index-url https://miropsota.github.io/torch_packages_builder pytorch3d==0.7.8+pt2.4.1cu121预编译轮子可用：https://miropsota.github.io/torch_packages_builder/pytorch3d/
* 2025-09-27 15:46:30.521269667 [E:onnxruntime:Default, provider_bridge_ort.cc:2195 TryGetProviderInfo_CUDA] /onnxruntime_src/onnxruntime/core/session/provider_bridge_ort.cc:1778 onnxruntime::Provider& onnxruntime::ProviderLibrary::Get() [ONNXRuntimeError] : 1 : FAIL : Failed to load library libonnxruntime_providers_cuda.so with error: libcudnn.so.9: cannot open shared object file: No such file or directory
* 应该是缺少cudnn9，但是我好像装了为啥没有？
* 这个cuda-toolkit貌似缺少部分库文件，并不是完整的cuda环境。我一直这么管理cuda，但是最近编译pytorch3d出现找不到CUDA头文件的报错，发现虚拟环境下并没有对应文件，然而另一种方法conda install -c pytorch torch=xxx torchvision torchaudio pytorch-cuda=121 这样安装后conda带的toolkit才能正常支持torch编译，但是conda中torchcuda的最新只有12.4（目前），对于cuda12.9相当不方便。这件事情相当奇怪
* 破案，似乎是conda install -c nvidia/label/cuda-12.9.1这样安装的toolkit不行

* https://anaconda.org/search?q=pytorch-cuda查找conda包
* Winodows安装需求：msvs需要17、19版本
* https://visualstudio.microsoft.com/zh-hans/vs/older-downloads/
* https://stackoverflow.com/questions/79513635/install-visual-studio-2017-via-visual-studio-2022-installer
* 推荐2019版本vs，安装msvcv141 142 



##### ubuntu环境问题

* fuse编译出错，似乎跟编译器有关
  * 破案是需要重启环境就行





python facefusion.py headless-run -s "./test_data/yy_data/sources/Musk.jpg" -t "./test_data/yy_data/targets/ex5.mp4" -o "./output.mp4" --face-swapper-model "inswapper_128_fp16" --processors face_swapper



python /media/easy/Data/facefusion/facefusion.py headless-run -s "~/file/idol_test/IDOL/faceswap/source.jpg" -t "~/file/idol_test/IDOL/faceswap/ex5.mp4" -o "~/file/idol_test/IDOL/faceswap/output.mp4" --face-swapper-model "inswapper_128_fp16" --processors face_swapper



python facefusion.py headless-run -s "./faceswap/13.jpg" -t "./faceswap/13_A.mp4" -o "./faceswap/faceswapped.mp4" --face-swapper-model "inswapper_128_fp16" --processors
face_swapper face_enhancer



转过180度时容易出现突变

~~似乎换脸并没有什么卵用，主要是面部增强~~配合换脸的效果才是最好的

额外的问题：因为在SMPLX应用并使高斯变形之后无法再应用高斯，因此似乎应该直接针对变形前的高斯进行优化。

折叠当前块：Ctrl+Shift+[

展开当前块：Ctrl+Shift+]

折叠所有块：Ctrl+K Ctrl+0

展开所有块：Ctrl+K Ctrl+J

\# %%

\# 修改代码，使用动态的 -1 数量来扩展第一个维度，其他维度自动保持

batch_res_points = tuple(t.expand(batch_views, *(-1 for _ in range(t.dim() - 1))) for t in res_points)
