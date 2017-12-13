# 京东猪脸比赛

[比赛信息](https://jddjr.jd.com/item/4)

## 1. 准备

1. 从训练视频中采样每一帧抽取一张图片
1. 【可选】使用[Faster RCNN](https://github.com/endernewton/tf-faster-rcnn)需要一个猪检测模型，从每张图片中裁剪出猪身体部分
1. 【可选】使用Faster RCNN训练一个猪头检测模型
1. 从测试集test_A中随机抽取600张图片作为验证集，用第2步训练好的模型自动标注验证集中的图片，对其中置信度低的图片进行人工标注（约70张，2小时标完）。验证集对后面选择超参数非常重要，也非常有效。人工标注的数据只用来选择超参数，不用来训练，不知道是否合规。

## 2. 训练

1. Finetune [DenseNet-161](https://github.com/pudae/tensorflow-densenet) block4和softmax层
1. Finetune [Inception-resnet-v2](https://github.com/tensorflow/models/tree/master/research/slim)
1. 【可选】针对猪头训练一个DenseNet分类器
1. 数据增强对结果影响非常大，没有用数据增强时测试集loss在1.5左右；使用水平翻转，旋转，调整亮度，对比度等手段做数据增强后，模型的loss下降到0.4左右
1. 使用上面三个模型，提取训练集，验证集合测试集各图片的average pooling层feature并保存到磁盘，供后面集成模型使用

## 3. 集成

1. 针对每一张图片把第2步中导出的每一个feature拼接在一起，得到一个更大的feature。如果图片中没有检测到猪头，就用0补齐feature。
1. 拼接后的feature作为输入训练一个3层的全连接网络，详细见assemble.ipynb
1. 选择在验证集中loss最小的模型，来预测测试集中的图片

## 4. 结果

1. 单模型使用数据增强后，loss在0.4左右； DenseNet稍微好于Inception-resnet-v2
1. 双模型集成后，loss下降到0.34左右
1. 添加猪头部分的feature后，loss下降到0.30

## 5. 其他
1. 尝试使用triplet loss和center loss等辅助训练网络，效果不明显
1. 尝试使用stn，attention等网络来训练模型，实验没做完。
1. 尝试区分猪的姿态，实验也没做完。
1. 比赛群里有人提出个奇技淫巧，用模型标注测试集，抽取置信度搞的图片加入训练集，多次迭代后可以显著降低测试集的loss。 试了一下，此方法确实能降低测试集的loss，但效果有限。

