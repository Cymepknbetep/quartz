### python学习笔记

#### 1 

* 相对目录出错时采用`os.path.join(os.path.dirname(__file__), 'smplx', 'SMPLX')`这样来设置文件将对目录
* OmegaConf挺好用的支持.和键值访问
* cv2的bgr和rgb
* 可以使用@dataclass来创建一个具有类型注释的config
* dataclass: 装饰器
  * 自动创建__init__等方法
* @staticmethod
  * 不依赖self类本身的实例或者cls类本身
  * 只是该函数储存在类的命名空间
  * 调用方便，不需要实例化