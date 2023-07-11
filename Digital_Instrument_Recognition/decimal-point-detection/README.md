# 检测数字仪表图像中的小数点
先前的工作：
* 使用Yolo检测数字区域
* CRNN+CTC端到端识别数字包括小数点

问题：
* 对于不同的表，需要大量训练样本，当遇到新表时候，泛化性较差
* 当训练样本小数点分布单一，数据少时，会出现错误，如下

想法：
* 有人用yolo去检测单个数字，进行组合识别，但是没有很好处理小数点
* 目前看到的处理小数点方法，除了crnn，基本就是先验信息，如，知道这个小数点固定在哪一位
* 然而实际中，小数点可能在任何一位
* 因此，**提出从形态学上去找到小数点**，可以作为crnn小数点位置的校验
* 也可与yolo识别数字的方案相结合，无需小数点先验知识

在Chat GPT的帮助下，一下午改了几版，如下：

## 第一版
* 读取图像并将其转换为灰度图像
* 对灰度图像进行二值化处理
* 查找轮廓
* 遍历所有轮廓，绘制面积小于250的轮廓
* 输出得到的轮廓横坐标位置
* 根据横坐标位置，和CRNN识别出的数字位数，计算小数点在第几位

## 第二版
* 对于不同颜色的，使用Otsu二值化方法计算阈值, 这里数字白色，背景黑色

## 第三版
* 通过黑白像素数量决定是否反转，解决了黑字白底、白底黑字的腐蚀问题。
* 调整了腐蚀核大小3-->5-->6，效果更好，但太过小的图像小数点粘连问题仍难解决
* 更改了小数点轮廓的判断准则，判断更合理准确：
      绘制“面积小于250”的轮廓  --->   绘制 “y坐标位于图像下方”内的“最小”轮廓
* 没有小数点的序列也不会误检出小数点

## 第四版
* 有些分隔的数字，底下会有一横，判断规则新加了，对小数点外接矩形框形状的判断，
```
if w/h < 2
```

