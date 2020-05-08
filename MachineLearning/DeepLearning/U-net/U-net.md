<!--
 * @Author: your name
 * @Date: 2020-05-07 22:53:46
 * @LastEditTime: 2020-05-08 10:06:27
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\MachineLearning\U-net.md
 -->
# U-net
## 概述
&emsp;&emsp;U-net（《U-Net: Convolutional Networks for Biomedical Image Segmentation》）发表于2015年的ICMICCAI，作者（Olaf Ronneberger, Philipp Fischer, and Thomas Brox）来自德国弗莱堡大学，是一种非常常用的自编码器，主要用于各类图像生成领域中（最初应用于语义分割）。其最主要的特点在于前半部分提取特征（收缩路径contracting path），后半部分再利用特征做图像生成（扩张路径expanding path），并且有一定的残差思想，且速度很快。

## 基本结构
&emsp;&emsp;最初的基本结构如下图所示：
![unet](unet.jpg)
因为整体形状就像个“U”因此得名。左半部分是由两个卷积层+ReLu组成一个单元，两个单元之间用池化层连接并缩小特征图面积，由此组成编码器，作用就是从一张输入的图像上提取特征图（下采样）；右半部分解码器（上采样）与编码器整体相似，由反卷积层取代了池化层，从而增大特征图面积，最终还原回与输入相似的大小（原文其实并不完全一样大，但是如果通过padding调整是可以一样大的）。其中解码器部分除了应用上一个解码单元输出的特征图以外，还将编码器中同一层的特征图拼接在了一起进行输入，称之为skip connection。这种拼接主要是将同样大小的特征图叠在一起（直观上看就是加厚），这样做得目的主要是为了融合更多低层次（low level）的特征，这种不同尺度的融合使得最后的效果比之前的FCN更好。

## 补充知识
&emsp;&emsp;一般我们只关心每个卷积层输入和输出的通道（特征图数量），但是这里我们不得不关注其特征图的大小，因为我们希望最后输出的图像还能跟原来一样大。所以补充关于特征图大小的计算知识。
### 二维卷积后的特征图大小
- 输入特征图大小：$L_{in}×L_{in}$
- 卷积核大小：$k×k$
- 步长：$step$
- padding：$p$
- 输出特征图大小：$L_{out}×L_{out}$
$$
L_{out}=[\frac{L_{in}-k+2p}{step}]+1
$$
### 池化后的特征图大小
- 输入特征图大小：$L_{in}×L_{in}$
- 池化核大小：$k×k$
- 步长：$step$（一般跟池化核一样）
- 输出特征图大小：$L_{out}×L_{out}$
$$
L_{out}=[\frac{L_{in}-k}{step}]+1
$$
### 反卷积后的特征图大小
- 输入特征图大小：$L_{in}×L_{in}$
- 卷积核大小：$k×k$
- 步长：$step$
- padding：$p$
- 输出特征图大小：$L_{out}×L_{out}$
$$
L_{out}=(L_{in}-1)×step-2p+k
$$
## 演示代码
```
# 代码来自https://zhuanlan.zhihu.com/p/47125912
# 编码器单元（下采样层）
def add_conv_stage(dim_in, dim_out, kernel_size=3, stride=1, padding=1, bias=True):
    return nn.Sequential(
        nn.Conv2d(dim_in, dim_out, kernel_size=kernel_size, stride=stride, padding=padding, bias=bias),
        nn.ReLU(),
        nn.Conv2d(dim_out, dim_out, kernel_size=kernel_size, stride=stride, padding=padding, bias=bias),
        nn.ReLU()
    )

# 解码器单元（上采样）
def upsample(ch_coarse, ch_fine):
    return nn.Sequential(
        nn.ConvTranspose2d(ch_coarse, ch_fine, kernel_size=4, stride=2, padding=1, bias=False),
        nn.ReLU()
    )

class Net(nn.Module):
    def __init__(self, useBN=False):
        super(Net, self).__init__()

        self.conv1 = add_conv_stage(3, 32)
        self.conv2 = add_conv_stage(32, 64)
        self.conv3 = add_conv_stage(64, 128)
        self.conv4 = add_conv_stage(128, 256)
        self.conv5 = add_conv_stage(256, 512)

        self.conv4m = add_conv_stage(512, 256)
        self.conv3m = add_conv_stage(256, 128)
        self.conv2m = add_conv_stage(128, 64)
        self.conv1m = add_conv_stage(64, 32)
        
        self.conv0 = nn.Sequential(
            nn.Conv2d(32, 3, 3, 1, 1),
            nn.Sigmoid()
        )

        self.max_pool = nn.MaxPool2d(2)

        self.upsample54 = upsample(512, 256)
        self.upsample43 = upsample(256, 128)
        self.upsample32 = upsample(128, 64)
        self.upsample21 = upsample(64, 32)
    
    def forward(self, x):
        conv1_out = self.conv1(x)
        conv2_out = self.conv2(self.max_pool(conv1_out))
        conv3_out = self.conv3(self.max_pool(conv2_out))
        conv4_out = self.conv4(self.max_pool(conv3_out))
        conv5_out = self.conv5(self.max_pool(conv4_out))

        conv5m_out = torch.cat((self.upsample54(conv5_out), conv4_out), 1)//torch.cat将不同level的特征图“叠起来”
        conv4m_out = self.conv4m(conv5m_out)
        conv4m_out_ = torch.cat((self.upsample43(conv4m_out), conv3_out), 1)
        conv3m_out = self.conv3m(conv4m_out_)
        conv3m_out_ = torch.cat((self.upsample32(conv3m_out), conv2_out), 1)
        conv2m_out = self.conv2m(conv3m_out_)
        conv2m_out_ = torch.cat((self.upsample21(conv2m_out), conv1_out), 1)
        conv1m_out = self.conv1m(conv2m_out_)
        conv0_out = self.conv0(conv1m_out)

        return conv0_out
```
最终loss为MSELoss，随机梯度下降来训练