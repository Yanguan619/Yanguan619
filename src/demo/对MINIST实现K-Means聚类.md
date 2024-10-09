---
title: 对 MINIST 实现 K-Means 聚类
abbrlink: 26757
date: 2022-01-23
cover: https://scikit-learn.org.cn/upload/4d8344fef1d9094044a328f8c6966f29.png
---
> [[K-Means]]

## 要求

分析 MNIST 手写体数据的基本结构，完成数据的标准化操作，采用先验 (在 0~9 中各随机选择一个作为初始聚类中心) 和随机 (在所有样本中随机选择十个作为初始聚类中心) 两种策略设计初始聚类中心，使用 Python 编程语言完成 K-Means 算法代码编写工作，对比真实类别标记，采用 Purity 测度给出两种选择下的聚类结果，并比较两类结果差异。

## 所需库

```python
import struct
import numpy as np
```

## 数据集导入

数据集文件：
- data（路径）
	- train-images.idx3-ubyte
	- train-labels.idx1-ubyte

由于二进制文件，这里使用 `struct.unpack` 解包。
```python
def load_data(path="data", kind='train'):

    labels_path = path + "/{}-labels.idx1-ubyte".format(kind)
    images_path = path + "/{}-images.idx3-ubyte".format(kind)
    
    with open(labels_path, 'rb') as lbpath:
        # 标签文件
        struct.unpack('>II', lbpath.read(8))
        labels = np.fromfile(lbpath, dtype=np.uint8)  # 使用np.fromfile读取剩下的数据

    with open(images_path, 'rb') as imgpath:
        # 图片文件
        struct.unpack('>IIII', imgpath.read(16))
        images = np.fromfile(imgpath, dtype=np.uint8).reshape(len(labels), 784)

    return images, labels
```

## 数据预处理

1. 打乱数据
将 images 数据、labels 数据合并后再打乱
2. 对导入的数据进行标准化操作
	- 这里使用**max-min 标准化**
```python
def load_data_deal():
    images, labels = load_data()
    # 合并->打乱->再分
    datas = np.column_stack((images, labels))[:3000, :]

    np.random.randn(7)
    np.random.shuffle(datas)
    
    images, labels = datas[:, :-1], datas[:, -1]
    images = images.astype(np.float64)
    # 归一化
    maximums, minimums = images.max(axis=0), images.min(axis=0)
    images = (images - minimums.T) / (maximums.T - minimums.T)
    images[np.isnan(images)] = 0

    return images, labels
```

## K-Means 算法

1. 初始化
```python
class k_means:
    def __init__(self, datas, labels, k=10, create=1):
        self.datas = datas
        self.labels = labels
        self.k = k
        self.row= datas.shape[0]
        self.center_labels = list(range(k))
        self.center_cluster = [[] for i in range(k)]
        self.center = self.center_create(datas, labels, k, create)
```

2. 产生聚类中心
```python
    def center_create(self, datas, labels, k, create):
        if create == 1:  # 先验选择聚类中心
            center_index = [int(np.argwhere(self.labels == i)[0]) for i in range(k)]
        else:
            center_index = np.random.choice(self.row, size=10, replace=False)
        
        center = datas[center_index, :].copy()
        self.center_labels = self.labels[center_index].copy()

        return center
```

3. 更新聚类中心
```python
    def center_update(self):
        cluster = [[] for i in range(k)] #
        center = self.center.copy()

        for i in range(self.row):  # 归类
            # 计算距离
            distances = [np.linalg.norm(self.datas[i,:]-center[j])
                            for j in range(self.k)]
            c = distances.index(min(distances))  # 最近簇标记
            cluster[c].append(i)  # 将数据的索引i放入cluster[c]中

        self.center_cluster = cluster  #更新簇
        self.purity_get()
        
        for i in range(self.k):    
            # 更新聚类中心
            centeri = np.mean(self.datas[cluster[i],:], axis=0)
            # 利用众值更新聚类中心标签
            self.center_labels[i] = np.argmax(np.bincount(self.labels[cluster[i]]))  
            if not (centeri == center[i]).all():
                center[i] = centeri 
                
        self.center = center.copy()
```

4. 训练
```python
    def train(self):
        epoch = 1
        epochs = 40
        while 1:
            print("-------{} of {}-------".format(epoch, epochs))
            print("center_labels:", self.center_labels)
            epoch +=1
            center = self.center.copy()

            self.center_update()

            if epoch > epochs or (self.center == center).all():
                print("=======聚类结束=======")
                break
```

5. 输出 Purity 测度
```python
    def purity_get(self):
        labels_fake = np.zeros(self.row)

        for i in range(self.k):
            labels_fake[self.center_cluster[i]] = self.center_labels[i]

        purity = np.sum(labels_fake == self.labels) / self.row

        print("purity: ", purity)
```

## 实现

```python
images, labels = load_data_deal()

k = 10
create = 1  # 1为先验
minist = k_means(images, labels, k, create)
minist.train()
```

## 结果展示

- 先验生成聚类中心的结果

![先验](https://img-blog.csdnimg.cn/a2fed085a1984aeca7e8b717fc2ca8b3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAWWFuZ3Vhbiw=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center) 
- 随机产生聚类中心得到的结果

![随机](https://img-blog.csdnimg.cn/5615b2cafa204979a040cb140320ed05.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAWWFuZ3Vhbiw=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
这里随机产生的结果和先验产生的结果并没有太大的差别，有人说随机产生应该是正确率 0.1 左右。（这不就和 10 个数字让机器随便猜它是几，它只能猜对一个，那么说明在这里随机产生聚类没什么用，我觉得不应该吧）
