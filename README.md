# LicensePlateOCR

本仓库用于存储第九组软工实践斗量车联项目车牌识别部分

## 项目结构

```
LicensePlateOCR/
├── checkpoints
├── detect
├── recognize
├── app.py
├── ocr.py

```

- checkpoints ：存放训练的网络模型(由于模型较大该文件夹未上传)
- detect ： 用于探测文本区域的CTPN网络
- recognize：用于识别文本的CRNN网络
- app.py：flask框架与前端交互的api
- ocr.py：识别接口

## CTPN网络简介

![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084010551-1739710374.png)

CTPN目的在于识别水平文本区域
首先使用Vgg16作为骨干网络对输入图片提取特征 输出结果为(N,C,H,W)的特征图
将该特征图进行维度变换为尺寸(NH,W,C)
输入双向LSTM，学习每一行的序列特征，输出尺寸变为(NH,W,256)
再变换经FC卷积层，变为尺寸为(N,512,H,W)的特征图
该特征图将分别用于分类分支与回归分支
经过以上操作特征图中的每一点将对应原图中16*16的区域，其中每个区域将生成10个宽为16，高度不等的框体
 ![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084729650-825233547.png)

即一共生成10HW个等宽不等高的框体用于框选文字
分类分支将特征图卷积为(N,10*2,H,W)的尺寸，用于对框体进行文字与背景的二分类
回归分支将特征图卷积为(N,10*2,H,W)的尺寸，用与对框体的中心点坐标y以及框体高度进行回归
 ![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084739809-468426131.png)

经过筛选与修正后将得到多串的小面积文本框体，使用文本线构造算法将框体横向连接即可获得文本位置
下图左侧为Faster R-CNN生成的普通建议框，右图为CTPN生成的文本框，显然CTPN更加适合文字检测
![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084759167-1208108086.png)

## CRNN网络简介

之后将CTPN的输出作为CRNN的输入。
CRNN采用了对序列预测的RNN网络。通过CNN将图片的特征提取出来后采用RNN对序列进行预测，最后通过一个CTC的翻译层得到最终结果。即CNN+RNN+CTC的结构。
![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084141194-1913489220.png)

CNN结构采用的是VGG的结构，并且对VGG网络做了一些微调
为了能将CNN提取的特征作为输入，输入到RNN网络中，文章将第三和第四个maxpooling的核尺度从2 × 2 改为了1 × 2 为了加速网络的训练，在第五和第六个卷积层后面加上了BN层。
RNN网络是对于CNN输出的特征序列x = x1 , ⋯ , xt，每一个输入xt都有一个输出yt 。为了防止训练时梯度的消失，文章采用了LSTM神经单元作为RNN的单元。文章认为对于序列的预测，序列的前向信息和后向信息都有助于序列的预测，所以文章采用了双向RNN网络。LSTM神经元的结构和双向RNN结构如下图所示。
 ![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084855321-880944840.png)

CTT则是是将RNN的输出y = y1 , y2, ⋯,yt 转化为一个字符串，而转化的输入与输出长度不对应而且输入可以是不同长度的序列。
由此可对图片中的文字进行识别

## 效果展示

![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084520422-2039288360.png)
![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084342968-1268693833.png)
![](https://img2020.cnblogs.com/blog/2533651/202111/2533651-20211127084407582-1195305421.png)

