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

第一个**配置文件** `./user_data/configs/reppoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck+head_2x_coco.py`。

第二个**配置文件** `./user_data/configs/_base_/datasets/coco_detection.py`。

我们一次只训练一个类别，对于`person_visible`, `person_full`, `person_head`, `vehicle`分别训练四个分类器。

第三个**配置文件** `./user_data/configs/_base_/default_runtime.py`，将**load_from**设置为`'./user_data/checkpoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck+head_2x_coco_20200329-f87da1ea.pth`。

## 下载COCO预训练模型
下载[RepPoints的COCO预训练模型](https://github.com/open-mmlab/mmdetection/tree/master/configs/reppoints)。

## 使用PANDA-IMAGE数据集对模型进行训练
使用多gpu进行训练，可以运行：

```shell
bash ./code/train.sh --configs ./user_data/configs/reppoints/reppoints_moment_x101_fpn_dconv_c3-c5_gn-neck+head_2x_coco.py 2
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
4. 使用Multi-scale进行训练，将输入图像在线resize为(1333, 480)和(1333, 960)。
5. 添加图像増广方法，随机更改亮度、对比度和滤波处理。
6. 学习率设置请查看`./user_data/configs/_base_/schedules/schedule_1x.py`。
7. 训练时使用两块2080TI，批量大小设置为6/gpu。

# 测试和推理
测试图像，可以运行：
```shell
bash ./code/run.sh
```
or
```shell
python ./code/test.py --source 
                      --config
                      --checkpoint
                      --out
                      --gpu-id
                      --tmpdir
```

**NOTE**

根据类别使用不同的缩放比例和滑动窗口尺寸：

| Class          | Scale      | Window Size |
| -------------- | ---------- | ----------- |
| person_visible | (0.1, 0.3) | (1000, 500) |
| person_full    | (0.1, 0.3) | (1000, 500) |
| person_head    | (0.1, 0.3) | (1000, 500) |
| vehicle        | (0.1, 0.3) | (1000, 500) |

根据类别使用不同的分数阈值和NMS阈值：

| Class          | Score | NMS   |
| -------------- | ----- | ----- |
| person_visible |  0.4  |  0.3  |
| person_full    |  0.4  |  0.3  |
| person_head    |  0.3  |  0.3  |
| vehicle        |  0.4  |  0.3  |



# 结果

| Matrices                | Details                                           | Score |
| ----------------------- | ------------------------------------------------- | ----- |
| Score                   |                                                   | 0.458 |
| Average Precision  (AP) | @[ IoU=0.50:0.95 \| area =  all \| maxDets=500 ]  | 0.427 |
| Average Precision  (AP) | @[ IoU=0.50    \| area =  all \| maxDets=500 ]    | 0.675 |
| Average Precision  (AP) | @[ IoU=0.75    \| area =  all \| maxDets=500 ]    | 0.461 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets= 10 ]  | 0.068 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets=100 ]  | 0.331 |
| Average Recall   (AR)   | @[ IoU=0.50:0.95 \| area =  all \| maxDets=500 ]  | 0.494 |
