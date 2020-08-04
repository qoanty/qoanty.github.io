---
title: "Tensorflow, Keras and deep learning, without a Phd"
date: 2019-10-15T10:06:45+08:00
tags: ["tensorflow", "keras", "deep learning"]
categories: ["Learn", "TensorFlow"]
draft: false
---

## 1. 概述

![](/images/tfkd11.png)

在本[代码实验室](https://codelabs.developers.google.com/codelabs/cloud-tensorflow-mnist/#0)中，你将学习如何构建和训练可识别手写数字的神经网络。在此过程中，随着你增强神经网络的准确率达到99%，你还将学习到专业人员用来训练模型的高效工具。

本代码实验室使用[MNIST](http://yann.lecun.com/exdb/mnist/)数据集，该数据集包含60,000个带标签的数字。你将使用不到100行的Python/TensorFlow代码来解决问题。

**你会学到什么**

- 什么是神经网络(neural network)以及如何训练它
- 如何使用tf.keras构建基本的1层神经网络
- 如何添加更多的神经网络层
- 如何设置学习率(learning rate)时间表
- 如何建立卷积神经网络(convolutional neural networks)
- 如何使用正则化(regularisation)技术：dropout, batch normalization
- 什么是过度拟合(overfitting)

## 2. Google合作实验室快速入门

本示例使用Google合作实验室，你无需进行任何设置。你可以从Chromebook运行它。请打开下面的文件，并执行单元格以熟悉Colab笔记本。

[`Welcome to Colab.ipynb`](https://colab.research.google.com/github/GoogleCloudPlatform/training-data-analyst/blob/master/courses/fast-and-lean-data-science/colab_intro.ipynb)

以下是其他说明：

**选择GPU后端**

![](/images/tfkd21.png)

在Colab菜单中，选择“代码执行程序 > 更改运行时类型”，然后选择GPU。在首次执行时会自动进行连接，或者你可以点击右上角的“连接”按钮。

**执行**

![](/images/tfkd22.png)

通过点击单元格并使用Shift-ENTER键执行一个单元格。你也可以使用“代码执行程序 > 全部运行”来运行全部单元格。

**目录**

![](/images/tfkd23.png)

所有的笔记都有一个目录，你可以通过点击左侧的黑色箭头打开它。

**隐藏单元**

![](/images/tfkd24.png)

有些单元格仅显示其标题，这是Colab特定的笔记本功能。你可以双击它们以查看其中的代码，但这通常不需要。实现特定功能或可视化功能，你仍然需要运行这些单元格以定义内部函数。

## 3. 训练神经网络

我们将首先看看神经网络训练。请打开下面的笔记本并遍历所有单元格。暂时不要关注该代码，我们将在后面解释。

[`keras_01_mnist.ipynb`](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_01_mnist.ipynb)

当你运行笔记本时，把注意力集中在可视化功能上，并参阅下面的说明。

**训练数据**

我们有一套带有标签的手写数字集，因此我们知道每张图片代表什么，即0到9之间的任意一个数字。在笔记本中，你将看到一段摘录：

![](/images/tfkd31.png)

我们将建立的神经网络将手写数字分为10类(0-9)。这样做是基于内部参数，该内部参数需要具有正确的值才能使分类正常工作。通过训练过程来学习该“正确值”，该训练过程需要带有图像和相关正确答案的“标记数据集”。

我们如何知道训练后的神经网络是否运行良好？使用训练数据集来测试网络将是作弊行为。它已经在训练过程中多次看到该数据集，毫无疑问会表现出色。我们需要另一个在训练过程中从未见过的标记数据集，以评估网络在“真实世界”的表现，它就是“验证数据集”。

> 该数据集中有50,000个训练数字。我们在每次迭代时都会将大小为128 的“批量”(batch)输入到训练循环中，这样系统在391次迭代后就可以看到所有的训练数字，我们称这为“时代”(epoch)。\
还有10,000个单独的“验证”数字用于测试模型的性能。

**训练**

随着训练的进行，一次会更新一批训练数据，内部模型参数会更新，并且模型在识别手写数字方面会越来越好。你可以在训练图上看到：

![](/images/tfkd32.png)

在右边，“准确性”只是正确识别的数字的百分比。随着训练的进行而提高，这很好。

在左边，我们可以看到“损失”。为了推动训练，我们将定义一个“损失”函数，该函数表示系统识别数字的严重程度，并尝试将其最小化。你在这里看到的是，随着训练的进行，在训练和验证数据上的损失都减少了：很好，这意味着神经网络正在学习。

X轴表示通过整个数据集的“时代”(epoch)次数或迭代次数。

**预测**

训练模型后，我们可以使用它来识别手写数字。下一个可视化图示显示了它在从本地字体（第一行）渲染的几个数字和在验证数据集的10,000个数字上的性能。预测的类出现在每个数字下，如果有误，则显示为红色。

![](/images/tfkd33.png)

如你所见，这个初始模型不是很好，但是仍然可以正确识别一些数字。它的最终验证准确率约为90%，对于我们开始使用的简单模型而言，还算不错，但它仍然意味着它错过了10,000个验证数字中的1000个。这可以显示的更多，这就是为什么看起来所有答案都是错误（红色）。

**张量**

数据存储在矩阵中。一个28*28像素的灰度图像适合一个28*28的二维矩阵。但是对于彩色图像，我们需要更大的维度。每个像素有3个颜色值（红色，绿色，蓝色），因此需要一个[28, 28, 3]的三维表。为了存储一批128张彩色图像，需要一个[128, 28, 28, 3]的四维表。

这些多维表称为“张量”(Tensors)，其维度列表是其“形状”(shape)。

## 4. [INFO]：神经网络101

**简而言之**

如果你已经知道下一段中所有粗体字，则可以转到下一个练习。如果你刚刚开始深入学习，那么欢迎，请继续阅读。

> 神经网络分类器是由几个层的神经元(neurons)组成。对于图像分类，这些可以是密集(dense)的层，或更常见的是卷积(convolutional)层。它们通常通过relu激活函数激活。最后一层使用与所有分类一样多的神经元，并用softmax激活。对于分类，交叉熵(cross-entropy)是最常用的损失(loss)函数，将一位有效(one-hot)编码的标签（即正确答案）与神经网络预测的概率进行比较。为了最大程度地减少损失，最好选择带有动量的优化器，例如Adam并训练成批(batches)的训练图像和标签。

![](/images/tfkd41.png)

对于构建成层序列的模型，Keras提供了Sequential API。例如，可以使用Keras将使用三个密集(dense)层的图像分类器编写为：

```python
model = tf.keras.Sequential([
    tf.keras.layers.Flatten(input_shape=[28, 28, 1]),
    tf.keras.layers.Dense(200, activation="relu"),
    tf.keras.layers.Dense(60, activation="relu"),
    tf.keras.layers.Dense(10, activation='softmax') # classifying into 10 classes
])

# this configures the training of the model. Keras calls it "compiling" the model.
model.compile(
  optimizer='adam',
  loss='categorical_crossentropy',
  metrics=['accuracy']) # % of correct answers

# train the model
model.fit(dataset, ... )
```

![](/images/tfkd42.png)

**单个密集层(dense layer)**

MNIST数据集中的手写数字是28*28像素的灰度图像。对它们进行分类的最简单方法是使用28*28=784像素作为1层神经网络的输入。

![](/images/tfkd43.png)

神经网络中的每个“神经元”(neuron)都会对其所有输入进行加权求和，增加一个称为“偏差”的常量，然后通过一些非线性“激活函数”(activation function)来反馈结果。“权重”(weights)和“偏差”(biases)是通过训练来确定的参数，首先使用随机值初始化它们。

上图表示了一个具有10个输出神经元的1层神经网络，因为我们要把数字分类为10类（0至9）。

**矩阵乘法(matrix multiplication)**

下面是处理图像集合的神经网络层如何通过矩阵乘法来表示：

![](/images/tfkd44.png)

使用权重矩阵W中的第一列权重，我们计算第一张图像的所有像素的加权总和，这个值对应于第一个神经元。使用第二列权重，我们执行相同的操作生成第二个神经元，依此类推，直到第十个神经元为止。然后，我们可以对剩余的99张图像重复该操作。如果我们将包含100个图像的矩阵称为X，则在100个图像上计算出的10个神经元的所有加权总和就是X.W，即矩阵乘法。

现在，每个神经元都必须加上它的偏差（一个常数）。因为我们有10个神经元，所以我们有10个偏差常数。我们将这个10个值的向量称为b。必须将其添加到先前计算的矩阵的每一行中。使用一种叫做“广播”的技巧，我们将用一个简单的加法来编写它。

> “广播”(Broadcasting)是Python和numpy（科学计算库）中使用的标准技巧。它扩展了对维度不一致的矩阵进行常规运算的方式。“广播添加”的意思是“如果要相加两个矩阵，但是由于它们的维度不一致而无法相加，请尝试复制一个小的矩阵以使其正常工作。”

最后，我们应用激活函数，例如“softmax”（解释如下），获得描述1层神经网络的公式，将该公式应用于100张图像：

![](/images/tfkd45.png)

**Keras**

使用像Keras类的高级神经网络库，我们将无需实现此公式。但是，重要的是要了解神经网络层只是一些矩阵乘法和加法。在Keras中，一个密集层可写为：

```python
tf.keras.layers.Dense(10, activation='softmax')
```

**深入**

把神经网络层连接起来很简单，第一层计算像素的加权和，后续层计算先前层输出的加权和。

![](/images/tfkd46.png)

除了神经元的数量之外，唯一的区别是激活函数的选择。

**激活函数：relu，softmax和Sigmoid**

通常，对除最后一层以外的所有层都使用“relu”激活函数。分类器中的最后一层将使用“softmax”激活。

![](/images/tfkd47.png)

同样，一个“神经元”计算所有输入的加权和，加上一个称为“偏差”的值，并通过激活函数提供结果。

修正线性单元(Rectified Linear Unit)是最常用的激活函数称为“RELU”。如上图所示，这是一个非常简单的函数。

传统的神经网络激活函数是“Sigmoid”，但“relu”几乎在所有地方都具有更好的收敛特性，因此现在是首选。

![](/images/tfkd48.png)

**Softmax激活函数**

神经网络的最后一层有10个神经元，因为我们想将手写数字分成10类（0-9）。它应该输出10个介于0到1之间的数字，表示这个数字是0、1、2等的概率。为此，在最后一层，我们将使用一个名为“softmax”的激活函数。

对向量应用softmax的方法是取每个元素的指数，然后对向量进行归一化，通常是对其除以它的“L1”范数（即绝对值的和），使归一化后的值加起来为1，也可以解释为概率。

激活之前最后一层的输出有时称为“logits”。如果这个向量是L = [L0，L1，L2，L3，L4，L5，L6，L7，L8，L9]，则：

![](/images/tfkd49.png)

> 为什么“softmax”被称为softmax？指数是一个急剧增长的函数。它会增加神经元输出之间的差异。然后，当你对向量进行归一化时，控制范数的最大元素将被归一化为接近1的值，而所有其他元素最终将除以一个大数并归一化为接近0的值。计算后的向量能清楚地显示出哪个是需要的类别，即“max”，但保留了其值的原始相对顺序，因此仍是“soft”。

**交叉熵损失(Cross-entropy loss)**

现在，我们的神经网络会根据输入的图像生成预测，我们需要测算它的效果，即神经网络预测的信息与正确答案（通常称为“标签”）之间的距离。请记住，我们对数据集中的所有图像都有正确的标签。

任何距离都会有效，但是对于分类问题，所谓的“交叉熵距离”是最有效的，我们将其称为“损失”函数：

![](/images/tfkd410.png)

> “一位有效”(one-hot)编码意味着你使用10个值的向量表示标签“this is digit 3”，除了第3个值为1之外，所有的值都为零，这个向量表示是“digit 3”的概率为100%。我们的神经网络还将其预测结果输出为10个概率值的向量。它们很容易比较。

**梯度下降(Gradient descent)**

“训练”神经网络实际上是指使用训练图像和标签来调整权重和偏差，从而最小化交叉熵损失函数。下面是它的工作原理。

交叉熵是训练图像及其已知类别的权重、偏差、像素的函数。

如果我们计算交叉熵相对于所有权重和所有偏差的偏导数，我们就会得到一个对于给定的图像、标签以及权重和偏差的当前值进行计算的“梯度”(gradient)，。记住，我们可以有数百万个权重和偏差，因此计算梯度听起来好像有很多工作。幸运的是，TensorFlow为我们做到了。梯度的数学特性是它指向“上”。因为我们想去交叉熵低的地方，所以我们往相反的方向去。我们用梯度的一小部分来更新权重和偏差。然后，我们在训练循环中使用下一批训练图像和标签一次又一次地做同样的事情。希望这可以收敛到交叉熵最小值的地方，尽管不能保证这个最小值是唯一的。

![](/images/tfkd411.png)

> “学习率”(Learning rate)：你不能在每次迭代时通过梯度的整个长度来更新权重和偏差。这就像穿着靴子试图走入谷底，你会从山谷的一侧跳到另一侧，要到达底部，你需要做一些小的步骤，即只使用梯度的一小部分，通常在1/1000范围内，这个分数称为“学习率”。

**小批量和动量(Mini-batching and momentum)**

你可以仅在一个示例图像上计算梯度并立即更新权重和偏差，但是在一批（例如128幅）图像上执行此操作也会产生一个梯度，该梯度可以更好地表示不同示例图像施加的约束，因此很可能会有更快地收敛结果。小批量的大小是一个可调整的参数。

这项技术有时称为“随机梯度下降”(stochastic gradient descent)，还有另一个更实际的好处：使用批量还意味着可以使用更大的矩阵，并且更容易在GPU和TPU上进行优化。

但是，收敛仍然会有些混乱，如果梯度向量全为零，它甚至会停止。这是否意味着我们已找到了最小值？不总是。梯度分量的最小值或最大值可以为零。对于具有数百万个元素的梯度向量，如果它们全为零，那么每一个零对应于最小值的概率和它们对应于最大值的概率是很小的。在多维度的空间中，鞍点是很常见的，我们不想停在它们上面。

![](/images/tfkd412.png)

图示：鞍点。梯度为0，但并非在所有方向上都为最小值。（图片提供方[Wikimedia：Nicoguaro 的作品，CC BY 3.0](https://commons.wikimedia.org/w/index.php?curid=20570051)）

解决方案是在优化算法中加入一些动量，使其能够不停地驶过鞍点。

> TensorFlow库提供了以`tf.train.GradientDescentOptimizer`开头的全部优化器。具有内置动量的更高级的常用优化器是`tf.train.RMSPropOptimizer`或`tf.train.AdamOptimizer`。

**词汇表**

批量或小批量(batch or mini-batch)：始终对一批训练数据和标签进行训练。这样做有助于算法收敛。“批量”的维度通常是数据张量的第一个维度。例如，形状为[100, 192, 192, 3]的张量包含100张192*192像素的图像，每个像素3个值(RGB)。

交叉熵损失(cross-entropy loss)：分类器中常用的一种特殊损失函数。

密集层(dense layer)：神经元的一层，其中每个神经元与前一层的所有神经元相连。

特征(features)：神经网络的输入有时称为“特征”。找出数据集的哪些部分（或部分的组合）输入神经网络以获得良好预测的技术称为“特征工程”。

标签(labels)：“类”的另一个名称或在监督分类问题中的正确答案。

学习率(learning rate)：在训练循环的每次迭代中都会更新权重和偏差的梯度分数。

logits：在应用激活函数之前，一层神经元的输出称为“logits”。该术语来自“逻辑函数”，又称“Sigmoid函数”，它曾经是最常用的激活函数。“逻辑函数前的神经元输出”简称为“logits”。

损失(loss)：将神经网络输出与正确答案进行比较的误差函数。

神经元(neuron)：计算其输入的加权总和，加上偏差并通过激活函数反馈结果。

一位有效编码(one-hot encoding)：5个类中的第3类被编码为5个元素的向量，除第三个元素是1之外其余的元素都是0。

relu：修正线性单元。神经元的常用激活函数。

sigmoid：另一个常用的激活函数，在特殊情况下仍然有用。

softmax：一种特殊的激活函数，作用于向量，增加最大部分与所有其他部分之间的差异，并将该向量归一化为总和是1，以便可以将其解释为概率向量。用作分类器的最后一步。

张量(tensor)：“张量”类似于矩阵，但是具有任意数量的维度。一维张量是向量，二维张量是一个矩阵。然后你可以拥有3、4、5或更多维度的张量。

## 5. 让我们看看代码

回到学习笔记，这一次，让我们阅读代码。

[`keras_01_mnist.ipynb`](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_01_mnist.ipynb)

在本节中，你的任务是阅读并理解代码，以便以后可以改进它。

让我们浏览一下笔记本中的所有单元。

**“参数”**

这里定义了批量(batch)的大小、训练时代(epoch)的次数和数据文件的位置。数据文件托管在Google Cloud Storage (GCS)存储器中，这就是它们的地址以`gs://`开头的原因。

**“导入”**

这里导入了所有必需的Python库，包括TensorFlow和可视化的matplotlib。

**“可视化实用程序[RUN ME]”**

此单元格包含无趣的可视化代码。默认情况下它是折叠的，但是你可以在有空的时候通过双击打开它并查看代码。

> 你必须运行此单元格(SHIFT-ENTER)，否则将无法解析内部定义的函数，并且稍后会出现错误。

**“tf.data.Dataset：解析文件并准备训练和验证的数据集”**

此单元格使用tf.data.Dataset API从数据文件加载MNIST数据集。没必要在此单元上花费太多时间。如果你对tf.data.Dataset API感兴趣，这里有一个讲解它的教程：[TPU-speed data pipelines](https://codelabs.developers.google.com/codelabs/keras-flowers-data/#0)。目前，最基本的是：

MNIST数据集中的图像和标签（正确答案）以固定长度存储在4个文件中，可以使用专用的函数加载：

```python
imagedataset = tf.data.FixedLengthRecordDataset(image_filename, 28*28, header_bytes=16)
```

现在，我们有了图像字节的数据集，它们需要被解码成图像。我们为此定义了一个函数。图像没有被压缩，因此该函数无需解码任何内容（`decode_raw`基本上不执行任何操作）。然后将图像转换为0到1之间的浮点值。我们可以在这里将它重塑为2D图像，但实际上我们将它保持为28*28像素的平面阵列，因为这是我们最初的密集层所期望的。

```python
def read_image(tf_bytestring):
    image = tf.decode_raw(tf_bytestring, tf.uint8)
    image = tf.cast(image, tf.float32)/256.0
    image = tf.reshape(image, [28*28])
    return image
```

我们使用`.map`方法将此函数应用于数据集，并获取图像数据集：

```python
imagedataset = imagedataset.map(read_image, num_parallel_calls=16)
```

我们对标签进行相同的读取和解码，然后使用`.zip`方法将图像和标签放在一起：

```python
dataset = tf.data.Dataset.zip((imagedataset, labelsdataset))
```

现在，我们有了一个成对的数据集（图像，标签），这是我们的模型所期望的，我们还没有准备好将其用于训练函数中：

```python
dataset = dataset.cache()
dataset = dataset.shuffle(5000, reshuffle_each_iteration=True)
dataset = dataset.repeat()
dataset = dataset.batch(batch_size)
dataset = dataset.prefetch(tf.data.experimental.AUTOTUNE)
```

tf.data.Dataset API具有准备数据集所需的所有实用函数：

`.cache`将数据集缓存在内存中。这是一个很小的数据集，因此可以正常工作。`.shuffle`用5000个元素的缓冲区对其进行轮换。重要的是，对训练数据进行充分的轮换。`.repeat`重复循环数据集。我们将对其进行多时代的训练(multiple epochs)。`.batch`将多个图像和标签组成一个批量。最后，`.prefetch`可以在GPU上训练当前批量数据时，使用CPU准备下一批量数据。

验证数据集与上述方式类似。现在，我们已经准备好定义一个模型并使用该数据集对其进行训练。

**“Keras模型”**

我们所有的模型都是连续有序的层，因此我们可以使用`tf.keras.Sequential`样式来创建它们。最初，这是一个单一密集层，它有10个神经元，因为我们将手写数字分为10类。它使用“softmax”激活，因为它是分类器中的最后一层。

Keras模型还需要知道其输入的形态，可以用`tf.keras.layers.Input`定义它。这里输入的向量是28*28像素的平面向量。

```python
model = tf.keras.Sequential(
  [
    tf.keras.layers.Input(shape=(28*28,)),
    tf.keras.layers.Dense(10, activation='softmax')
  ])

model.compile(optimizer='sgd',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# print model layers
model.summary()

# utility callback that displays training curves
plot_training = PlotTraining(sample_rate=10, zoom=1)
```

在Keras中使用`model.compile`函数完成模型的配置。这里我们使用基本的优化器`'sgd'`（随机梯度下降）。分类模型需要一个交叉熵损失函数，在Keras中称为`'categorical_crossentropy'`。最后，我们要求模型计算`'accuracy'`，即正确分类的图像的百分比。

Keras提供了一个非常好的`model.summary()`函数，可以打印你创建的模型的详细信息，并且还添加了`PlotTraining`函数（在“可视化实用程序”单元格中定义的），该函数将在训练期间显示各种训练曲线。

**“训练和验证模型”**

通过调用`model.fit`并传入训练和验证的数据集，训练将在这里进行。默认情况下，Keras在每个时代(epoch)结束时进行一轮验证。

```python
model.fit(training_dataset, steps_per_epoch=steps_per_epoch, epochs=EPOCHS,
          validation_data=validation_dataset, validation_steps=1,
          callbacks=[plot_training])
```

在Keras中，可以使用回调函数在训练期间添加自定义行为，这还能够实现动态更新训练图曲线。

**“可视化预测”**

模型训练好后，我们可以通过调用`model.predict()`来进行预测：

```python
probabilities = model.predict(font_digits, steps=1)
predicted_labels = np.argmax(probabilities, axis=1)
```

这里我们准备了一组本地渲染的印刷数字来进行测试。记住，神经网络从它的最后一层“softmax”返回一个10个概率值的向量。为了得到标签，我们必须找出哪个概率值最高，numpy库中的`np.argmax`可以做到这一点。

要了解为什么需要`axis=1`参数，请记住我们已经处理了一批量128张图像，因此模型返回了128个概率值向量，输出张量为[128, 10]。我们要计算每个图像返回的10个概率值的argmax，因此axis=1（第一个轴为0）。

这个简单的模型已经可以识别90%的数字。不错，但是你还能大大改善这一点。

![](/images/tfkd51.png)

## 6. 添加网络层

![](/images/tfkd61.png)

为了提高识别精度，我们将向神经网络添加更多层。

![](/images/tfkd62.png)

我们将保留softmax为最后一层的激活函数，因为这最适合分类。但是，在中间层，我们将使用最经典的激活函数：sigmoid：

![](/images/tfkd63.png)

> 请在“softmax”激活的层之前添加两个用“sigmoid”激活的中间密集层。这些层的神经元的数量可以在784（输入像素的数量）到10（输出神经元的数量）之间。然后重新训练模型。

> 警告：要进行重新训练，必须重新运行“Keras模型”单元以定义新模型，然后再训练单元。只需重新运行训练单元，就可以继续上一个模型的训练。

例如，你的模型可能看起来像这样（不要忘记逗号，`tf.keras.Sequential`采用逗号分隔层列表）：

```python
model = tf.keras.Sequential(
  [
      tf.keras.layers.Input(shape=(28*28,)),
      tf.keras.layers.Dense(200, activation='sigmoid'),
      tf.keras.layers.Dense(60, activation='sigmoid'),
      tf.keras.layers.Dense(10, activation='softmax')
  ])
```

看看模型的“摘要”。它现在至少有10倍以上的参数。它应该好十倍！但是由于某种原因，它并不是...

![](/images/tfkd64.png)

损失似乎也很严重，有些事情不太对劲。

## 7. 对深度网络的特别关注

你刚刚已经体验了神经网络，就像人们过去在80年代和90年代设计的一样。难怪他们放弃了这个想法，迎来了所谓的“人工智能的冬天”。确实，随着网络层的添加，神经网络收敛的难度越来越大。

事实证明，在提供了一些数学技巧的基础上，可以使具有很多层（如今的20，50，甚至100层）的深度神经网络更好的收敛。这些简单技巧的发现是2010年代深度学习复兴的原因之一。

**RELU激活**

![](/images/tfkd71.png)

在深度网络中，sigmoid激活函数实际上存在很大问题。它会压缩0到1之间的所有值，当你反复这样做时，神经元输出及其梯度可能会完全消失。这是因为历史原因，但是现代网络使用的RELU（修正线性单元）激活函数：

把你的网络层中所有的`activation='sigmoid'`替换为`activation='relu'`，然后重新训练。

![](/images/tfkd72.png)

为什么sigmoid激活函数会有问题？看看它在右侧的表现：它变平了。

![](/images/tfkd73.png)

这意味着它的导数接近于零。记住训练是通过遵循梯度（它是导数向量）进行的。通过sigmoid激活函数，尤其是在有多个层的情况下，梯度会变得非常小，训练会变得越来越慢。

另一方面，relu至少在其右侧它的导数是1。有了RELU激活函数，即使来自某些神经元的梯度为零，也总会有其它神经元给出一个清晰的非零梯度，使训练可以以良好的速度继续进行。

**更好的优化器**

在像这里的多维空间中，我们有大约1万个权重和偏差，“鞍点”会很常见。这些点不是局部最小值，但是梯度为零，并且梯度下降优化器仍然停留在那里。TensorFlow有一整套可用的优化器，其中一些可以在一定的惯性下工作，并且可以安全地驶过鞍点。

例如，用更好的`'adam'`优化器替换`'sgd'`优化器，然后重新训练。

**随机初始化**

在训练前初始化权重和偏差的技术本身就是一个研究领域，有很多关于这个主题的论文发表。你可以在这里查看[Keras](https://keras.io/initializers/)中提供的所有初始值设定项。幸运的是，Keras在默认情况下使用了`'glorot_uniform'`初始值设定项，这在几乎所有情况下都是最好的。

你无需做任何事情，因为Keras已经做了正确的事情。

**NaN ???**

交叉熵公式包含一个对数，并且log(0)不是数字(Not a Number)（NaN是指数字崩溃）。交叉熵的输入可以是0吗？输入来自softmax，它本质上是一个指数，并且指数永远不会为零。所以我们很安全！

真的？在美丽的数学世界中，我们是安全的，但是在计算机世界中，以float32格式表示的exp(-150)等于零，并且交叉熵崩溃。

幸运的是，在这里你也不需要做任何事情，因为Keras处理了这个问题，并以特别谨慎的方式计算softmax和交叉熵，以确保数值稳定性并避免可怕的NaN。

**成功？**

![](/images/tfkd74.png)

你现在应该达到97%的准确率了，而目标是使准确率超过99%，因此让我们继续前进。

如果你遇到问题，这里是解决方案：

[keras_02_mnist_dense.ipynb](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_02_mnist_dense.ipynb)

## 8. 学习率衰减

也许我们可以试着训练得更快些？Adam优化器中的默认学习率是0.001，让我们尝试增加它的值。

将学习率从默认值0.001提高到0.01。为此，必须将`'adam'`预定义优化器替换为adam优化器的实例，以便可以访问其配置参数：`optimizer=tf.keras.optimizers.Adam(lr=0.01)`。请在PlotTraining中将缩放系数提高到5，这将缩放0位置处的损失和1位置处的准确率，以帮助你了解发生了什么：`PlotTraining(sample_rate=10, zoom=5)`

加快速度似乎并没有多大帮助，这些噪音到底是什么？

![](/images/tfkd81.png)

训练曲线确实很嘈杂，看看两个验证曲线：它们上下跳跃。这意味着我们走得太快了。我们可以回到以前的速度，但是还有更好的方法。

![](/images/tfkd82.png)

好的解决方法是快速启动并以指数方式衰减学习率。在Keras中，你可以使用`tf.keras.callbacks.LearningRateScheduler`回调函数来完成此操作。

执行一个学习率计划，该计划会使学习率呈指数衰减。你将在下面找到有用的代码段。

```python
# lr decay function
def lr_decay(epoch):
  return 0.01 * math.pow(0.6, epoch)

# lr schedule callback
lr_decay_callback = tf.keras.callbacks.LearningRateScheduler(lr_decay, verbose=True)

# important to see what you are doing
plot_learning_rate(lr_decay, EPOCHS)
```

不要忘记使用你创建的`lr_decay_callback`函数，将它添加到`model.fit`回调函数列表中：

```python
model.fit(...,  callbacks=[plot_training, lr_decay_callback])
```

这个小小的变化所产生的影响是惊人的。你会看到大部分噪音都消失了，并且测试准确率现在持续保持在98%以上。

## 9. Dropout, overfitting

这个模型现在似乎很好地收敛了，让我们试着更深入。

请将模型的深度增加到至少4个密集层，然后重新训练。你可以在层中使用以下数量的神经元：200、100、60、10。

请在PlotTraining中将缩放系数提高到10。这将缩放0位置处的损失和1位置处的准确率，以帮助你了解发生了什么：`PlotTraining(sample_rate=10, zoom=10)`

这有帮助吗？

![](/images/tfkd91.png)

作用不大，准确率仍然停留在98%，看看验证损失，它正在上升！该学习算法仅对训练数据有效，并相应地优化了训练损失。它从没看到过验证数据，因此一段时间后，它的工作不再对验证损失产生影响也就不足为奇了，该损失会停止下降，甚至会反弹。

这不会立即影响模型的实际识别能力，但是会阻止你运行多次迭代，并且通常表现出训练不再具有积极影响。

![](/images/tfkd92.png)

这种脱节通常称为“过拟合”(overfitting)，当你看到它时，可以尝试应用一种称为“dropout”的正则化技术。dropout技术会在每次训练迭代中随机发射神经元。

dropout是深度学习中最古老的正则化技术之一。在每次训练迭代中，它以概率p（通常为25%到50%）从网络中丢弃随机神经元。实际上，神经元输出设置为0，最终的结果是这些神经元这次将不会参与损失计算，也不会得到权重的更新。在每次训练迭代中都会丢弃不同的神经元。

![](/images/tfkd93.png)

当然，在测试网络性能时，你会将所有神经元放回去(dropout rate=0)。Keras会自动执行此操作，因此你只需添加一个`tf.keras.layers.Dropout`层，它会在训练和评估时自动具有正确的行为。

为什么会有效？从理论上讲，神经网络在其众多层之间有如此大的自由度，以至于完全有可能一个层进化出不良行为，而下一层则有可能对其进行补偿。这不是神经元的理想用途。由于dropout，在给定的训练回合中，神经元“修复”这个问题的可能性是不存在的。违规层的不良行为变得很明显，权重也朝着更好的行为发展。

你可以在网络中的每个中间密集层之后添加dropout，不要在softmax层之后添加dropout，你会降低你的预测概率。你可以通过以下方法增加25%的dropout rate：`tf.keras.layers.Dropout(0.25)`

有效果吗？

![](/images/tfkd94.png)

噪音再次出现（毫不奇怪，考虑到dropout的原理）。验证损失似乎不再有增加的趋势，但总体而言要比没有dropout的情况要高，验证准确率下降了一点，这是一个相当令人失望的结果。

似乎dropout不是正确的解决方案，或者“过拟合”是一个更复杂的概念，它的某些原因不适合进行“dropout”修复？

什么是“过拟合”？当一个神经网络学习“不好”时，就会发生过拟合，这种方式适用于训练实例，但对实际数据却不太适用。像dropout这样的正则化技术可以迫使它以更好的方式学习，但是过拟合也有更深的根源。

![](/images/tfkd95.png)

当神经网络对眼前的问题有太多的自由度时，就会发生基本的过拟合。想象我们有这么多神经元，网络可以将我们所有的训练图像存储在其中，然后通过模式匹配来识别它们，它将完全无法处理真实世界的数据，必须对神经网络进行一定程度的约束，以使其被迫概括它在训练过程中所学到的东西。

如果你只有很少的训练数据，即使是一个很小的网络也可以用心去学习，并且你也会看到“过拟合”。一般来说，你总是需要大量的数据来训练神经网络。

最后，如果你已经做了所有的事情，并尝试了各种不同规模的网络以确保其自由度受到约束，应用了dropout并训练了大量的数据，那么你可能仍然停留在一个性能水平，似乎没有什么改进。这意味着你的神经网络以目前的状态无法从你的数据中提取更多信息，就像我们这里的例子。

还记得我们如何使用扁平化为单一矢量的图像吗？那真是个坏主意。手写数字是由形状组成的，当我们将像素扁平化时就会丢弃形状信息。然而，有一种可以利用形状信息的神经网络：卷积网络。让我们试试。

如果你遇到问题，这里是解决方案：

[keras_03_mnist_dense_lrdecay_dropout.ipynb](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_03_mnist_dense_lrdecay_dropout.ipynb)

## 10. [INFO]卷积网络

**简而言之**

如果你已经知道下一段中所有粗体字的术语，那么你可以转到下一个练习。如果你刚开始使用卷积神经网络，请继续阅读。

![](/images/tfkd101.png)

图示：使用两个连续的过滤器对图像进行过滤，每个过滤器由4*4*3=48个可学习的权重组成。

> 卷积神经网络将一系列可学习的过滤器应用于输入图像，卷积层由过滤器（或内核）的大小，应用的过滤器的数量和步长定义，卷积层的输入和输出分别具有三个维度（宽度，高度，通道数），从输入图像（宽度，高度，RGB通道）开始，当堆叠卷积层时，可使用步长>1或最大池化(max-pooling)操作来调整输出的宽度和高度，通过使用更多或更少的过滤器来调整输出的深度（通道数）。

这就是Keras中一个简单的卷积神经网络：

```python
model = tf.keras.Sequential([
    tf.keras.layers.Reshape(input_shape=(28*28,), target_shape=(28, 28, 1)),
    tf.keras.layers.Conv2D(kernel_size=3, filters=12, activation='relu'),
    tf.keras.layers.Conv2D(kernel_size=6, filters=24, strides=2, activation='relu'),
    tf.keras.layers.Conv2D(kernel_size=6, filters=32, strides=2, activation='relu'),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(10, activation='softmax')
])
```

![](/images/tfkd102.png)

在卷积网络的一层中，一个“神经元”仅在图像的一小部分区域内对其上方的像素进行加权和。它增加了一个偏差，并通过激活函数计算总和，就像规则的密集层中的神经元一样，然后使用相同的权重在整个图像上重复此操作。记住，在密集层中，每个神经元都有自己的权重，在这里，单个“权重”在图像的两个方向上滑动（“卷积”），输出的值与图像中的像素一样多（但是在边缘需要一些填充），这是一个过滤操作，在上图中，它使用4*4*3=48权重的过滤器。

然而，48个权重是不够的，为了增加更多的自由度，我们用一组新的权重重复相同的操作，这将产生一组新的过滤器输出，与输入图像中的R、G、B通道类似，我们将其称为输出的“通道”。

![](/images/tfkd103.png)

通过添加新维度，可以将两组（或更多组）权重求和成为一个张量，这为我们提供了卷积层权重张量的一般形状。由于输入和输出通道的数量是参数，因此我们可以开始堆叠和链接卷积层。

![](/images/tfkd104.png)

图示：卷积神经网络将数据的“多维数据集”转换为其他数据的“多维数据集”。

**交错卷积，最大池化**

通过以2或3的步长进行卷积，我们还可以在水平维度上缩小生成的多维数据集。有两种常用的方法：

- 交错卷积：如上所述的步长>1的滑动过滤器
- 最大池化：应用最大操作的滑动窗口（通常在2*2的小块上，每2个像素重复一次）

![](/images/tfkd105.png)

图示：将计算窗口滑动3个像素会导致输出值更少。交错卷积或最大池化（最大值在2*2的窗口上以2的步长滑动时）是在水平维度上缩小多维数据集的一种方式。

> “最大池化”可以帮助我们直观地理解卷积网络是如何工作的：如果假设在训练过程中，权重块演变为可以识别基本形状（水平和垂直线、曲线等）的过滤器，那么将有用的信息记下来的方法是保持通过这些层的输出，其形状能被最大强度的识别。实际上，在最大池化层中，神经元输出以2*2为一组进行处理，并且只保留最大值。

**最后一层**

在最后一个卷积层之后，数据以“多维数据集”的形式出现，有两种方法可以将其送入最终的密集层。

第一种方法是将多维数据集转化为向量，然后将其送到softmax层，有时，甚至可以在softmax层之前添加一个密集层，就权量的数量而言，这往往是昂贵的，卷积网络末端的密集层可以包含整个神经网络一半以上的权重。

除了使用昂贵的密集层之外，我们还可以将输入的“多维数据集”划分为与我们的类一样多的部分，并取它们的平均值，然后通过softmax激活函数将其输入，这种构建分类头的方法的权重为0。在Keras中，此层包含：`tf.keras.layers.GlobalAveragePooling2D()`。

![](/images/tfkd106.png)

> 卷积神经网络检测事物的位置。当过滤器强烈响应某个特征时，它会在特定的(x, y)位置响应，根据你想做什么，可以训练神经网络使用或丢弃这个位置的数据，使用全局平均池化显式丢弃所有位置数据，这对于花卉分类器可能很好，但对于其他任务（例如对图像中的花朵计数）可能不起作用，需谨慎使用。

跳至下一部分，为眼前的问题建立一个卷积网络。

## 11. 一个卷积网络

让我们建立一个用于手写数字识别的卷积网络，我们将在顶部使用三个卷积层，在底部使用传统的softmax读出层，并将它们与一个完全连接的层相连：

![](/images/tfkd111.png)

注意，第二和第三卷积层的步长为2，这解释了为什么它们将输出值的数量从28*28降低到14*14，然后再降低到7*7。

让我们编写Keras代码。

在第一卷积层之前需要特别注意，事实上，它希望获得一个三维的“多维数据集”，但到目前为止，我们的数据集已经为密集层设置好，并且图像的所有像素都被扁平化为一个矢量，我们需要将它们重塑为28*28*1图像（1个灰度图像通道）：

```python
tf.keras.layers.Reshape(input_shape=(28*28,), target_shape=(28, 28, 1))
```

到目前为止，你可以使用这一行替代`tf.keras.layers.Input`。

在Keras中，“relu”激活的卷积层的语法为：

![](/images/tfkd112.png)

```python
tf.keras.layers.Conv2D(kernel_size=3, filters=12, padding='same', activation='relu')
```

卷积层中的padding参数可以有两个值：

- “same”：用零填充，以便产生与输入相同的宽度/高度的输出
- “valid”：不填充，仅使用真实像素

我们在这里使用填充，因此请将填充设置为“same”。

对于有步长的卷积，你可以这样写：

```python
tf.keras.layers.Conv2D(kernel_size=6, filters=24, padding='same', activation='relu', strides=2)
```

要将多维数据集扁平化为向量，以便密集层可以使用它：

```python
tf.keras.layers.Flatten()
```

对于密集层，语法没有变化：

```python
tf.keras.layers.Dense(200, activation='relu')
```

修改模型将其转换为卷积模型，你可以使用上图中的数值来调整大小，你可以保持学习率衰减不变，但是这里需删除dropout。

还请在PlotTraining中将缩放系数提高到16，这将缩放0位置处的损失和1位置处的准确率，以帮助你了解发生了什么：`PlotTraining(sample_rate=10, zoom=16)`

你的模型是否突破了99%的准确率？非常接近...但是请看验证损失曲线，这能使你回想起什么来吗？

![](/images/tfkd113.png)

也请看一下预测，这是第一次，你应该看到10,000个测试数字中的大部分现在都被正确识别了，只有下大约4.5行识别错误（10,000个数字中约有110个数字识别错误）。

![](/images/tfkd114.png)

如果你遇到问题，这里是解决方案：

[keras_04_mnist_convolutional.ipynb](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_04_mnist_convolutional.ipynb)

## 12. 再次dropout

先前的训练显示出明显的过拟合迹象（并且仍没达到99%的准确率），我们应该再次尝试dropout吗？

在网络中的两个密集层之间添加一个dropout层，dropout率为40%(=0.4)：`tf.keras.layers.Dropout(0.4)`

将dropout应用于卷积层是可行的，但是它的工作原理略有不同，并且通常效果不佳，因为卷积层中的权重要少得多，因此不好的协同适应的机会也更少。我们不会对此进行探讨。

这次怎么样了？

![](/images/tfkd121.png)

这次dropout似乎有效了，验证损失不再增加，最终准确率应在99%以上，恭喜你！

第一次尝试应用dropout时，我们认为我们遇到了过拟合问题，而实际上问题出在神经网络的体系结构中。如果没有卷积层，我们就无法走得更远，而dropout对此也无能为力。

这一次，看起来确实是过拟合引起的问题，而dropout确实有所帮助。记住，随着验证损失逐渐增加，有很多因素会导致训练损失曲线与验证损失曲线之间的脱节。过拟合（自由度过大，网络使用不当）只是其中之一，如果你的数据集太小或神经网络的体系结构不充分，你可能会在损失曲线上看到类似的行为，而dropout也不会有帮助。

## 13. 批量归一化

![](/images/tfkd131.png)

最后，让我们尝试添加批量归一化。

什么是批量归一化？

简而言之，批量归一试图解决神经元输出相对于神经元激活函数如何分布的问题。在一个小批量的训练数据中，神经元输出在激活之前可能具有以下分布：

![](/images/tfkd132.png)

sigmoid激活后，距离左侧太远，该神经元几乎总是输出0。

![](/images/tfkd133.png)

太窄了，该神经元永远不会输出明确的0或1。

![](/images/tfkd134.png)

这还不错，神经元在这个小批量中会有一系列的行为。

为了解决这个问题，批量归一将每个批量训练数据中的神经元输出归一化，即减去平均值并除以标准差。但是，这样做太残酷了，如果每个神经元都有完美居中和分布广泛的区域，所有的神经元都会具有相同的行为。诀窍是为每个神经元引入两个额外的可学习参数$\alpha$和$\beta$，并进行计算：

$\alpha$ . normalized_out + $\beta$

然后应用激活函数（sigmoid，relu等），这样网络通过机器学习来决定在每个神经元上应有多少居中和重新缩放。如果这样做是正确的，那么读者可以验证是否存在$\alpha$和$\beta$的值可以完全消除归一化。

实际上，$\alpha$和$\beta$并非总是都需要，在Keras中，它们分别称为“比例”和“中心”，你可以有选择地使用其中之一，例如：

```python
tf.keras.layers.BatchNormalization(scale=False, center=True)
```

批量归一的问题在于，在预测时，你没有可用于计算神经元输出统计信息的训练批量数据，但是你仍然需要这些值。在训练过程中，一个“典型”的神经元输出平均值和标准差是通过“足够”数量的批量数据计算出来的，实际上是使用指数平均值，然后在预测时使用这些统计数据。

好消息是，在Keras中，可以使用`tf.keras.layers.BatchNormalization`层，所有这些计算将自动进行。

这是理论，在实践中，只需记住一些规则：

批量归一化“规则”：

- 批量归一化介于层的输出与其激活函数之间。
- 如果你在批量归一化中使用center=True，则不需要在层中使用偏差，批量归一化的偏移量起着偏差的作用。
- 如果你使用的激活函数是比例不变的（即缩放后不会改变形态），则可以设置scale=False，Relu是比例不变的，sigmoid不是。

然而，这是一个理论，找出违反这些规则是否有害的唯一方法是尝试。

现在让我们按书来做，在每个神经网络层（除最后一个）上添加一个批量归一化层，不要将其添加到最后一个“softmax”层，那里没有用。

```python
# Modify each layer: remove the activation from the layer itself.
# Set use_bias=False since batch norm will play the role of biases.
tf.keras.layers.Conv2D(..., use_bias=False),
# Batch norm goes between the layer and its activation.
# The scale factor can be turned off for Relu activation.
tf.keras.layers.BatchNormalization(scale=False, center=True),
# Finish with the activation.
tf.keras.layers.Activation('relu'),
```

现在的准确率如何？

![](/images/tfkd135.png)

稍微调整一下（BATCH_SIZE=64，学习率衰减为0.666，密集层的dropout率为0.3），再加上一点运气，就可以达到99.5%，学习率和dropout调整是按照使用批量归一化的“最佳实践”进行的：

- 批量归一化有助于神经网络收敛，通常可以让你训练得更快。
- 批量归一化是一个正则项，你通常可以减少使用dropout的量，甚至根本不使用dropout。

该解决方案有99.5%的准确率：

[keras_05_mnist_batch_norm.ipynb](https://colab.research.google.com/github/GoogleCloudPlatform/tensorflow-without-a-phd/blob/master/tensorflow-mnist-tutorial/keras_05_mnist_batch_norm.ipynb)

## 14. 在强大的硬件上进行云训练：AI平台

![](/images/tfkd141.png)

你将在GitHub上的[mlengine](https://github.com/GoogleCloudPlatform/tensorflow-without-a-phd/tree/master/tensorflow-mnist-tutorial/mlengine)文件夹中找到该代码的云版本，以及在[Google Cloud AI Platform](https://cloud.google.com/ai-platform/)上运行该代码的说明。在运行此部分之前，你必须创建一个Google Cloud帐户并启用账单，完成实验所需的资源花费应该不到几美元（假设在一个GPU上训练1小时）。准备你的帐户：

1. 创建一个[Google Cloud Platform](http://cloud.google.com/console)项目。
2. 启用账单。
3. 安装[GCP](https://cloud.google.com/sdk/downloads#interactive)命令行工具。
4. 创建一个Google Cloud Storage存储器（放在us-central1区域）。它将用于准备训练代码并存储你的训练模型。
5. 启用必要的API并请求必要的配置（运行一次training命令，应该会收到错误消息，告诉你要启用什么）。

## 15. 恭喜！

你已经建立了第一个神经网络，并对其进行了全面的训练，达到了99%的准确率。这一过程中学习到的技术并非特定于MNIST数据集，实际上，它们在处理神经网络时被广泛使用。作为临别礼物，这是卡通版的“纲要”卡片，你可以用它来记住所学到的知识：

![](/images/tfkd151.png)

**下一步**

- 在完全连接和卷积网络之后，你应该看看[递归神经网络](https://youtu.be/fTUwdXUFfI8)。
- 为了在分布式基础架构上的云中进行训练或推理，Google Cloud提供了[AI平台](https://cloud.google.com/ai-platform/)。
- 最后，我们喜欢反馈，如果你发现本实验室中有什么问题，或者你认为应该加以改进，请告诉我们，我们通过GitHub[问题反馈链接](https://github.com/googlecodelabs/feedback/issues/new?title=[cloud-tensorflow-mnist]:&labels[]=content-platform&labels[]=cloud) 处理反馈。

