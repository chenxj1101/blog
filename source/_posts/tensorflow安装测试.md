layout: post
title: win10+gtx960m安装tensorflow-gpu及MNIST手写体识别程序
date: 2018/3/15 20:02:37
categories:
- 机器学习
tags:
- tensorflow
- 机器学习
- win10

---

在自己的笔记本上安装了tensorflow-gpu版本，做个记录。

## 环境

- win10 64位
- GTX960m
- Python 3.5.2 :: Anaconda 4.2.0 (64-bit)

## 依赖

通过查看[TensorFlow Windows安装文档](https://www.tensorflow.org/install/install_windows)，想要安装`TensorFlow-GPU`版本的前提是有`Nvidia`显卡并安装了以下依赖:

- CUDA® Toolkit 9.0
- 支持CUDA® Toolkit 9.0的显卡驱动
- cuDNN v7.0
- 具有CUDA Compute Capability 3.0 或更高版本的显卡(GTX960m 的Compute Capability为5.0，满足)

## 安装

### CUDA Toolkit安装

从[CUDA Toolkit](https://developer.nvidia.com/cuda-90-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exenetwork)下载CUDA Tookit的9.0版本。下载完直接默认安装即可，安装完成后会自动加入环境变量。

如果电脑中没有`Visual Studio`，安装之前会提示你没有安装`Visual Studio`。这个应该可装可不装，不影响使用(**到目前为止**)

安装完成后，在Power Shell里运行：

```bash
nvcc -V
输出：
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Sep__1_21:08:32_Central_Daylight_Time_2017
Cuda compilation tools, release 9.0, V9.0.176
```
表示安装成功

### 显卡驱动

一般情况下，显卡驱动都已经安装好了。
如需安装，可以从[显卡驱动](https://www.geforce.cn/drivers)下载

### cuDNN安装

从 [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)下载TensorFlow-GPU版本指定的7.0版本(最新版为7.1)。下载cuDNN需要注册Nvidia Developer账号。

下载完成后解压，然后将解压后的文件放入**上述**`CUDA toolkit`安装目录对应的目录下。

### TensorFlow-GPU安装

直接在Power Shell里运行：

```bash
pip install --upgrade tensorflow-gpu
```
可能会因为网络等原因安装失败，可以尝试多安装几次。（本人装了3次才成功……）


## 测试

安装完TensorFlow后，可以使用`MNIST手写数字识别`来测试。代码如下：

```python
# -*- coding: UTF-8 -*-
import tensorflow as tf

## 导入MNIST数据集，查看训练集，测试集，验证集的情况
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
print(mnist.train.images.shape, mnist.train.labels.shape)
print(mnist.test.images.shape, mnist.test.labels.shape)
print(mnist.validation.images.shape, mnist.validation.labels.shape)

## 输入数据
sess = tf.InteractiveSession()
x = tf.placeholder(tf.float32, [None, 784])
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))

## 实现Softmax Regression算法
y = tf.nn.softmax(tf.matmul(x, W) + b)

## 损失函数
y_ = tf.placeholder(tf.float32, [None, 10])
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))

## 初始化
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
tf.global_variables_initializer().run()

## 训练
for i in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    train_step.run({x: batch_xs, y_: batch_ys})


## 准确率
correct_predition = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
accuracy = tf.reduce_mean(tf.cast(correct_predition, tf.float32))

print(accuracy.eval({x: mnist.test.images, y_:mnist.test.labels}))
```

运行后输出很多WARNING，因为代码还是基于tensorflow1.0的。并且会在运行目录下生成`MNIST_data`文件夹。

有意义的输出如下：
```bash
Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 1420 MB memory) -> physical GPU (device: 0, name: GeForce GTX 960M, pci bus id: 0000:01:00.0, compute capability: 5.0)
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting MNIST_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting MNIST_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting MNIST_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
(55000, 784) (55000, 10)
(10000, 784) (10000, 10)
(5000, 784) (5000, 10)
0.9162
```

可见已经是使用GPU来运行了。

