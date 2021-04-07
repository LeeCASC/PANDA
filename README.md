# 安装
1. 环境：**Python 3**
2. 安装依赖：
```
    pip install -r requirements.txt
```
3. 我们使用mmDetection训练模型，请访问[this page](https://github.com/open-mmlab/mmdetection)进行mmDetection的安装。
4. 我们使用[RepPoints (ICCV'2019)](https://github.com/open-mmlab/mmdetection/tree/master/configs/reppoints)作为模型进行训练和测试。

# 使用mmDetection训练模型
使用PANDA-IMAGE数据集微调RepPoints模型。
1. 准备PANDA-IMAGE数据集；
2. 准备mmDetection配置文件；
3. 下载COCO预训练模型；
4. 使用PANDA-IMAGE数据集对模型进行训练；
5. 一些tricks。

## 准备PANDA-IMAGE数据集
根据[官方baseline](https://github.com/GigaVision/PANDA-Toolkit/edit/gaiic-panda)，准备PANDA-IMAGE数据集。

并将数据集转换为COCO格式。

## 准备mmDetection配置文件
准备配置文件保证模型可以成功读取数据集。

第一个**配置文件** `configs/_base_/models/faster_rcnn_r50_fpn.py`，配置文件如下所示：

```python

```

我们一次只训练一个类别，对于`person_visible`, `person_full`, `person_head`, `vehicle`分别训练四个分类器。

第二个**配置文件** `configs/_base_/datasets/coco_detection.py`，配置文件如下所示：

```python

```

第三个**配置文件** `configs/_base_/default_runtime.py`，将**load_from**设置为`'./checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth'`。

## 下载COCO预训练模型
下载[RepPoints的COCO预训练模型](https://github.com/open-mmlab/mmdetection/tree/master/configs/reppoints)。

## 使用PANDA-IMAGE数据集对模型进行训练
使用多gpu进行训练，可以运行：

```shell
python train.py configs/faster_rcnn/faster_rcnn_r50_fpn_1x_coco.py [optional arguments]
```

## Tricks
1. 使用模型：

| Method    | Backbone      | GN  | Anchor | convert func | Lr schd | Mem (GB) | Inf time (fps) | box AP | Config | Download |
|:---------:|:-------------:|:---:|:------:|:------------:|:-------:|:--------:|:--------------:|:------:|:------:|:--------:|
| RepPoints | X-101-FPN-DCN | Y   | none   | moment       | 2x      | 7.1      | 9.3            | 44.2   | [config](https://github.com/open-mmlab/mmdetection/tree/master/configs/reppoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck+head_2x_coco.py) | [model](http://download.openmmlab.com/mmdetection/v2.0/reppoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck%2Bhead_2x_coco/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck%2Bhead_2x_coco_20200329-f87da1ea.pth) &#124; [log](http://download.openmmlab.com/mmdetection/v2.0/reppoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck%2Bhead_2x_coco/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck%2Bhead_2x_coco_20200329_132201.log.json) |

2. 针对不同类别的检测器，使用不同的图像尺度进行训练：

| Class          | Scale |
| -------------- | ----- |
| person_visible |  0.5  |
| person_full    |  0.5  |
| person_head    |   1   |
| vehicle        |  0.3  |

3. 将缩放后的图像裁剪为(2048,1024)输入模型。
4. 使用Multi-scale进行训练。

# 测试和推理
测试图像，可以运行：
```shell
python test.py
```

**NOTE**

根据类别使用不同的缩放比例和滑动窗口尺寸：

| Class          | Scale      | Window Size |
| -------------- | ---------- | ----------- |
| person_visible | (0.1, 0.3) | (1000, 500) |
| person_full    | (0.1, 0.3) | (1000, 500) |
| person_head    | (0.1, 0.3) | (1000, 500) |
| vehicle        | (0.1, 0.3) | (1000, 500) |

根据类别使用不同的分数阈值：

| Class          | Thresh |
| -------------- | ------ |
| person_visible |  0.4   |
| person_full    |  0.4   |
| person_head    |  0.3   |
| vehicle        |  0.4   |



# 结果

| Matrices                | Details                                           | Score |
| ----------------------- | ------------------------------------------------- | ----- |
| Average Precision  (AP) | @[ IoU=0.50:0.95 \| area =  all \| maxDets=500 ]  | 0.312 |
| Average Precision  (AP) | @[ IoU=0.50    \| area =  all \| maxDets=500 ]    | 0.482 |
| Average Precision  (AP) | @[ IoU=0.75    \| area =  all \| maxDets=500 ]    | 0.352 |
| Average Precision  (AP) | @[ IoU=0.50:0.95 \| area = small \| maxDets=500 ] | 0.203 |
| Average Precision  (AP) | @[ IoU=0.50:0.95 \| area =medium \| maxDets=500 ] | 0.376 |
| Average Precision  (AP) | @[ IoU=0.50:0.95 \| area = large \| maxDets=500 ] | 0.352 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets= 10 ]  | 0.061 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets=100 ]  | 0.266 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets=500 ]  | 0.347 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area = small \| maxDets=500 ] | 0.229 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =medium \| maxDets=500 ] | 0.413 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area = large \| maxDets=500 ] | 0.379 |
