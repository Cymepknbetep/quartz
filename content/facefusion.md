### Facefusion 笔记

#### 1. 概述

#### 2. 配置

* ffmpeg的安装（主要是版本问题和编码器问题）——`extrating video failed``merging video failed`
  * 检验支持不支持h264的编码：`ffmpeg -f lavfi -i testsrc=duration=1:size=576x1024:rate=30 -c:v libx264 -preset medium -pix_fmt yuv420p /media/easy/Data/test_output.mp4`
  * 删除环境中的ffmpeg之后会使用系统ffmpeg，但是虚拟环境的环境变量优先级更高导致系统ffmpeg到conda环境找包而找不到
  * 最优先最独立的方案应该是使用conda环境中独立的ffmpeg，但是condaforge频道中的ffmpeg似乎不包含最新的ffmpeg（apt安装的为6.1.1而conda的更旧）
* 出现问题时使用`python facefusion.py run --open-browser --log-level debug`的`--log-level`提供详细的日志等级
* 出现不调用gpu的情况时检查是否保证了onnx onnx-runtime cuda cudnn 的版本匹配，通过https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html查阅对应的版本
* 需要注意numpy2与numpy1存在明显的不兼容
* 建议在拉取了新的repo的情况下要使用新的缓存文件

#### 3. 使用

python facefusion.py headless-run -s "./faceswap/13.jpg" -t "./faceswap/13_A.mp4" -o "./faceswap/faceswapped.mp4" --face-swapper-model "inswapper_128_fp16" --processors
face_swapper face_enhancer



#### 4. 摘取

`signal.signal(signal.SIGINT, signal_exit)  # 优雅的退出方式`