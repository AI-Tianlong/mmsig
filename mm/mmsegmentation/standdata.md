# 贡献一个标准格式的数据集
## 在 projects 中贡献一个标准 mmseg 格式的数据集
在您开始工作前，请先阅读[OpenMMLab 贡献代码指南](https://mmcv.readthedocs.io/zh_CN/latest/community/contributing.html),以了解更详细的代码贡献流程。
该教程所贡献数据集为 [Gaofen Image Dataset (GID)](https://www.sciencedirect.com/science/article/pii/S0034425719303414) 和 WHDLD 遥感图像语义分割数据集。

## 1 配置 mmsegmentation 运行所需必要的环境，并 git clone mmsegmentation
环境安装请参考[中文快速入门指南](https://github.com/open-mmlab/mmsegmentation/blob/main/docs/en/get_started.md)或[英文 get_started](https://github.com/open-mmlab/mmsegmentation/blob/main/docs/en/get_started.md)  
![image](https://user-images.githubusercontent.com/50650583/233825292-2e52d574-f4b9-4404-b18e-eede70a67d7f.png)  
注意：这里的安装需要从源码构建，以方便您进行开发。

## 2 代码贡献前应该完成的准备工作
### 2.1 fork mmsegmentation 仓库
* 通过浏览器打开[mmsegmentation 官方仓库](https://github.com/open-mmlab/mmsegmentation/tree/main)。
* 登录您的 Github 账户，以下步骤均需在Github登录的情况下进行。
* fork mmsegmentation 仓库
* ![image](https://user-images.githubusercontent.com/50650583/233825567-b8bf273c-38f5-4487-b4c6-75ede1e283ee.png)
### 2.2 在您的代码编写软件中 git clone mmsegmentation
这里以 VSCODE 为例
* 打开 VSCODE，新建终端窗口并激活您在[1 环境安装](#1-配置-mmsegmentation-运行所需必要的环境并-git-clone-mmsegmentation)中所安装的虚拟环境。
* 运行命令`git clone git@github.com:open-mmlab/mmsegmentation.git`或`git clone https://github.com/open-mmlab/mmsegmentation.git`
* ![image](https://user-images.githubusercontent.com/50650583/233825892-80029b4d-b0b3-4aeb-8906-9b3df62fb634.png)

### 2.3 cd mmsegmentation 并 pip install -v -e .
将当前目录切换至`mmsegmentation`并执行`pip install -v -e .`通过源码构建方式安装 mmsegmentaion 仓库。
安装完成后，您将能看到如下图所示的文件树结构。
![image](https://user-images.githubusercontent.com/50650583/233826064-4b111358-8f97-44dd-955c-df3204410b8b.png)  

### 2.4 切换分支为 dev-1.x
正如您在[ mmsegmentation 官网](https://github.com/open-mmlab/mmsegmentation/tree/main)所见，有许多分支，默认分支`main`为稳定的发行版本，以及用于开发的`dev-1.x`分支。`dev-1.x`分支是贡献者们用来提交您创意和PR的分支。
![image](https://user-images.githubusercontent.com/50650583/233826225-f4b7299d-de23-47db-900d-dfb01ba0efc3.png)

回到 VSCODE 中，在终端运行命令
```bash
git checkout dev-1.x
```
### 2.5 创新属于自己的新分支
在基于`dev-1.x`分支下，创建处于您自己的分支。
使用如下命令格式
```bash
# git checkout -b 您的GithubID/您的分支想要实现的功能的名字
# git checkout -b AI-Tianlong/support_GID_dataset
git checkout -b xxxx/xxxx
```
### 2.6 配置 pre-commit
详细请看[配置 pre-commit](https://mmcv.readthedocs.io/zh_CN/latest/community/contributing.html#pre-commit) 

## 3  在`mmsegmentation/projects`下贡献您的代码
#### 先对 GID 数据集进行分析

这里以贡献遥感图像语义分割数据集 GID 为例，GID数据集是由我国自主研发的高分2号卫星所拍摄的光学遥感图像创建的，经图像预处理后共提供了150张6800*7200像素的RGB三通道的遥感图像。并提供了两种不同类别数的数据标注，一种是包含5类有效物体的RGB标签，另一种是包含15类有效物体的RGB标签。本教程将针对第一种标签进行数据集贡献讲解。  
GID的5类有效标签分别为：0-背景-[0,0,0](mask标签值-标签名称-RGB标签值)、1-建筑-[255,0,0]、2-农田-[0,255,0]、3-森林-[0,0,255]、4-草地-[255,255,0]、5-水-[0,0,255]。在语义分割任务中，标签是与原图尺寸一致的单通道图像，标签图像中的像素值为真实样本图像中对应像素所包含的物体的类别。GID数据集提供的是具有RGB三通道的彩色标签，为了模型的训练需要将RGB标签转换为mask标签。并且由于图像尺寸为 6800*7200 像素，过大的图像尺寸对于神经网络的训练来说是不合适的，所以将每张图像裁切成了若干没有重叠的512*512的图像以便进行训练。
![image](https://user-images.githubusercontent.com/50650583/234192183-83ee4209-e181-4a18-90ca-4d71757cd2c7.png)
### 3.1 在`mmsegmentation/projects`下创建新文件夹
在`mmsegmentation/projects`下创建文件夹`gid_dataset`
![image](https://user-images.githubusercontent.com/50650583/233829687-8f2b6600-bc9d-48ff-a865-d462af54d55a.png)

### 3.2 贡献您的数据集代码
为了最终能将您在projects中贡献的代码更加顺畅的移入核心库中(对代码要求质量更高)，非常建议按照核心库的目录来编辑您的数据集文件。
关于数据集有 3 个必要的文件：
* `mmseg/datasets/gid.py` 定义了数据集的尾缀、CLASSES、PALETTE等
* `configs/_base_/gid.py` GID数据集的配置文件，定义了数据集的`dataset_type`（数据集类型，`mmseg/datasets/gid.py`中注册的数据集的类名）、`data_root`(数据集所在的根目录，建议将数据集通过软连接的方式将数据集放至`mmsegmentation/data`)、`train_pipline`(训练的数据流)、`test_pipline`(测试和验证时的数据流)、`img_rations`(多尺度预测时的多尺度)、`tta_pipeline`（多尺度预测）、`train_dataloader`(训练集的数据记载器)、`val_dataloader`(验证集的数据加载器)、`test_dataloader`(测试集的数据加载器)、`val_evaluator`(验证集的评估器)、`test_evaluator`(测试集的评估器)。
* 使用了当前数据集 `config` 的模型配置文件
这个是可选的，但是强烈建议您添加。在核心库中，所贡献的数据集需要和参考文献中所提出的结果精度对齐，为了后期将您贡献的代码合并入核心库。如您的算力充足，最好能提供对应模型配置文件的在您贡献的数据集上所验证的结果以及相应的权重文件，并撰写较为详细的README.md文档。[示例参考结果](https://github.com/open-mmlab/mmsegmentation/tree/main/configs/deeplabv3plus#mapillary-vistas-v12)
![image](https://user-images.githubusercontent.com/50650583/233877682-eabe8723-bce9-40e4-a303-08c8385cb6b5.png)

### 3.3 贡献`mmseg/datasets/gid.py`
可参考[`projects/mapillary_dataset/mmseg/datasets/mapillary.py`]并在此基础上修改相应变量以适配您的数据集。  
```python
# Copyright (c) OpenMMLab. All rights reserved.
from mmseg.datasets.basesegdataset import BaseSegDataset

from mmseg.registry import DATASETS


@DATASETS.register_module()       # 注册数据集类
class GID_Dataset(BaseSegDataset):
    """Gaofen Image Dataset (GID)

    Dataset paper link:
    https://www.sciencedirect.com/science/article/pii/S0034425719303414
    
    GID  5 classes: background, built-up, farmland, forest, meadow, water

    The ``img_suffix`` is fixed to '.tif' and ``seg_map_suffix`` is
    fixed to '.tif' for Mapillary Vistas Dataset.
    """
    METAINFO = dict(
        classes=('background', 'built-up', 'farmland', 'forest', 
                 'meadow', 'water'),

        palette=[[0, 0, 0], [255, 0, 0], [0, 255, 0], [0, 255, 255],
                 [255, 255, 0], [0, 0, 255]])

    def __init__(self,
                 img_suffix='.tif',
                 seg_map_suffix='.tif',
                 reduce_zero_label=None,
                 **kwargs) -> None:
        super().__init__(
            img_suffix=img_suffix, 
            seg_map_suffix=seg_map_suffix, 
            reduce_zero_label=reduce_zero_label,
            **kwargs)

```




