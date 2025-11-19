### IDOL-DEMO笔记

项目地址 https://git.nju.edu.cn/yiyu/idol-demo/-/tree/main?ref_type=heads

#### 1.代码概述

* **目的**：构建一个Gradio Web界面，允许用户上传一张人体图像（带透明背景或适合背景移除）和一个目标运动视频（对应SMPLX参数）（两者均可以使用摄像头捕捉），利用IDOL模型生成动画视频。

  - **图像输入**：支持上传图片或使用摄像头拍摄
  - **视频输入**：支持选择预设视频或使用摄像头视频流
  - **高斯数字人生成**：从单张图像重建3D高斯数字人模型
  - **姿态估计**：从视频帧中实时估计人体姿态
  - **动画生成**：使用姿态数据驱动高斯数字人
  - **渲染展示**：支持实时视频渲染和4DGS查看器交互式渲染

* **项目结构**

* ```plaintext
  .
  ├── main.py                  # 主程序入口     @cms
  ├── pose_estimation/         # 姿态估计模块   @gbc
  │   ├── __init__.py
  │   └── pose_estimator.py    # 姿态估计器实现  @zyy
  ├── image_processing/        # 图像处理模块   
  │   ├── __init__.py
  │   └── image_processor.py   # 图像处理器实现
  ├── gsavatar/                # 高斯数字人模块  @zyy
  │   ├── __init__.py
  │   ├── gsavatar_generator.py # 高斯数字人生成器 
  │   └── gsavatar_animator.py  # 高斯数字人动画器 
  ├── refinements/             # 优化模块   @yzy @wxy
  │   ├── __init__.py
  │   ├── faceswap
  |   └── difix3d               
  ├── visualization/           # 可视化模块   @hlj
  │   ├── __init__.py
  │   └── renderer.py          # 渲染器实现
  └── requirements.txt         # 依赖包列表
  ```



#### 2 环境配置

* 注意为了保证编译器环境变量生效需要重新激活环境





#### 3 IDOL优化

* 渲染管线中的code_bt这一系列的batch都可以移出循环加快速度

#### 4 高斯优化

##### 4-1 换脸

* 转过180度时容易出现突变
* ~~似乎换脸并没有什么卵用，主要是面部增强~~配合换脸的效果才是最好的
* 额外的问题：因为在SMPLX应用并使高斯变形之后无法再应用高斯，因此似乎应该直接针对变形前的高斯进行优化。
* 

##### 4-2 高斯优化

* 原始点云位置（pcd）、尺度/协方差（sigma）、颜色（rgbs）、偏移/不透明度（offset，可能指位置偏移或alpha值）和半径（radius，用于控制大小）
* 引入vgg似乎没有明显的改进
* 降低步数到30也有不错效果
* 取消offset的优化似乎也行但是radius不能变，而原始点云位置也不需要优化
* TODO：固定pose、优化渲染代码

#### 5 汇总

###### 模型下载

* sha256sum .assets/models/nlf_l_multi.torchscript | awk "{print $1}" 来得到SHA256
  * 





##### NLF

* 约定采用SMPLX的标准格式？
  * global_orient 



* 但是NLF似乎不包括expression



