### 一些python and Conda环境配置问题

#### 1 环境配置

主要是一些虚拟环境问题

##### 1-1 conda的包管理

**推荐优先选择conda进行包管理，pip仅用于安装部分仅python包**

* **conda的命令使用**

  * conda支持在特定位置存储虚拟环境```conda create -p (or --prefix) \path```这可以有效提高包管理效率

  * 相对的是```conda create -n (or --name) env_name```

  * 删除环境```conda env remove -p (or -n) \path (or env_name)```

  * Conda的版本指定```conda install numpy=2```，

  * | 操作符         | 含义                 | 示例命令                      | 匹配示例（针对 requests） |
    | -------------- | -------------------- | ----------------------------- | ------------------------- |
    | `=`            | 精确匹配             | `requests=2.25.1`             | 只 2.25.1                 |
    | `>=`           | 最小版本（模糊向上） | `requests>=2.25.0`            | 2.25.0 或更高             |
    | `<=`           | 最大版本（模糊向下） | `requests<=2.25.1`            | 2.25.1 或更低             |
    | `package [op]` | 范围匹配             | `requests [>=2.25.0,<2.26.0]` | 2.25.x 但不超 2.26        |

* 常用命令

  * ```conda install -c nvidia -c pytorch cuda-toolkit=12.9 cudnn=9.10.2 cudnn -y ```

    * -c:指定channel，优先从该channel搜索包并安装，`conda search -c nvidia`查找该频道的所有包名
      * default:官方默认channel包含大部分基础包
      * conda-forge社区驱动的channel，最大的非官方channel
      * nvidia 主要包含`cudnn libcudnn cuda-toolkit cuda-runtime nccl pytorch-cuda`等版本`pytorch-cuda`只要用与绑定cuda和torch还包括其他包
      * pytorch包含`pytorch torchbision torchaudio pytorch-cuda`四个包，其中`pytorch-cuda`用于选择支持的`cuda`版本，但是**注意截至2025.09并未提供高版本cuda12.4+例如cuda12.9 ** PyTorch 正在逐步弃用某些 Anaconda 依赖的构建，以降低维护成本

    

**1-2 pip的包管理**

* **pip的命令使用**

  * 显示可用的包版本`pip index versions numpy`

  * 安装某个范围内的包版本`pip install "numpy>2.2,<2.3"`遵循PEP440的语法

  * 安装特定版本的包版本`pip install numpy==2.2.6`

  * | 操作符 | 含义                 | 示例       | 匹配示例（针对 requests） |
    | ------ | -------------------- | ---------- | ------------------------- |
    | `==`   | 精确匹配             | `==2.25.1` | 只 2.25.1                 |
    | `~=`   | 兼容发布（模糊匹配） | `~=2.25.0` | 2.25.x 但不超 2.26        |
    | `>=`   | 最小版本             | `>=2.25.0` | 2.25.0 或更高             |
    | `=`    | **无效**             | `=2.25.1`  | 命令失败                  |

* 常用命令

  * pytorch的下载安装命令`pip install torch==2.8.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu129 `其实来说不如conda的安装，但是**conda截至2025.09并不支持高版本cuda12.4+例如cuda12.9的直接安装（没有预编译轮子）**，因为pytorch正逐步放弃conda的安装维护，注意只要安装时包含了pytorch-cuda包就能保证完整性
  * cudnn的安装命令`pip install nvidia-cudnn-cu12==9.1.0.70` not recommended，不如conda的环境级安装，
  * 安装结束需要确定torch的版本和cudnn的版本时使用`python -c "import torch;print(torch.backends.cudnn.version());print(torch.cuda.versions);print(torch.cuda.is_available)"`

* 一些问题：
  * 安装时某些requirements直接使用>2.2的写法没有设置版本上限非常容易出现问题



#### 2 包的安装



#### 3 包的编译

##### 3-1 注意事项

* 编译环境前需要注意包括c++等编译器的设置
  * linux好解决，例如ubuntu下直接`apt install gcc gxx g++`即可，必要时指定版本，然而windows下需要MSVC
    * 通常来说需要仔细查阅需要哪些版本的MSVC，最简单的办法就是MSVC141 142 143 144都安装，因为草台班子微软会在某些明确支持MSVC142时情况下给你很新的MSVC142导致版本不兼容，这时候有MSVC141就没事
    * 微软的MSVC安装直接在MS Installer中完成





























