# neural_renderer_pytorch-win10
解决neural_renderer_pytorch在win10上编译安装的问题

neural_renderer_pytorch使用说明请参照[neural_renderer](https://github.com/daniilidis-group/neural_renderer)，这里主要解决neural_renderer_pytorch在win10上编译安装不通过问题。



## 问题描述

在Linux系统或者是Mac系统上，直接执行命令`pip install neural_renderer_pytorch`是可以直接安装成功的，但是在win10系统上，会报各种编译错误，如何在win10上愉快地玩耍neural_renderer呢？



## 解决方案

本人经过无数次采坑，尝试，将问题定位为3个，1、vs版本和nvcc版本不匹配导致nvcc编译错误。2、pytorch的源码'cpp_extension.py'第233行，编译信息的解码未设置成"gbk"模式。3、neural_renderer源码中有些地方编译不通过（原因未知）。

那么针对上述3个问题，下面给出解决方案

- vs版本和nvcc版本问题：这里推荐vs2019和CUDA10.1，pytorch和torchvision版本最好是最新的且要匹配

- pytorch源码问题：在你的python环境中找到pytorch的`cpp_extension.py`文件,路径一般是`/anaconda/Lib/site-packages/torch/utils/cpp_extension.py`，将其第233行的代码

  `match = re.search(r'(\d+)\.(\d+)\.(\d+)', compiler_info.decode().strip())`

  修改为

  `match = re.search(r'(\d+)\.(\d+)\.(\d+)', compiler_info.decode(' gbk').strip())`

  注意gbk前面有一个空格，不然会报错。

- neural_renderer源码问题：我对neural_renderer源码进行了一些修改，请下载修改后的代码进行源码安装`python setup.py install`

  主要修改内容有:

  - `/cuda/create_texture_image_cuda.cpp`：注释了所有的`AT_CHECK`,`CHECK_INPUT`（不知道为什么win10这个编译的时候会报错）
  - `/cuda/load_textures_cuda.cpp`：注释了所有的`AT_CHECK`,`CHECK_INPUT`
  - `/cuda/rasterize_cuda.cpp`：注释了所有的`AT_CHECK`,`CHECK_INPUT`
  - `/cuda/rasterize_cuda_kernel.cu`：注释了`static __inline__ __device__ double atomicAdd(double* address, double val)`函数，因为高版本pytorch似乎自带double类型的atomicAdd，于是这里再写一遍会冲突。



