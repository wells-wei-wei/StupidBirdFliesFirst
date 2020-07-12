<!--
 * @Author: your name
 * @Date: 2020-06-12 21:27:30
 * @LastEditTime: 2020-07-12 18:58:42
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\MachineLearning\MachineLearning.md
--> 
# 机器学习基础知识
## 基本术语
- 数据集
- 样本
- 特征（属性）
- 样本空间（属性空间，输入空间）
- 特征向量
- 训练集
- 假设
- 真实（groundtruth）
- 学习器
- 预测
- 标记
- 标记空间（输出空间）
- 分类
- 回归
- 测试
- 据类
- 监督学习
- 无监督学习
- 泛化
- 分布 独立同分布
- 归纳
- 演绎
  
## 假设空间

## 归纳偏好
归纳偏好必须存在，所谓归纳偏好就是依据实际问题在假设空间找到最好的假设。脱离这个实际是没有意义的。

## 深度
深度就是神经网络的层数。提升网络性能最常用的方法就是增加网络深度：例如，从开始的LeNet-5，到VGG-16，到ResNet-101，等等。网络的设计越来越深，性能也越来越好。网路深了，简单地讲，下一层的输出是上一层的线性组合加激活，可以复合出更多更灵活的特征（其实就是更复杂的复合函数嘛）。

## 宽度
宽度就是神经网络中特征图的通道数，这里的典型例子可以从LeNet-5，到VGG-16，Wider ResNet。增加网络宽度，特征图通道数多起来了，更多的卷积核可以得到更多更丰富的特征，网络的表达能力自然就强了；