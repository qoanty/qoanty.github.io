---
title: "使用TensorFlow Hub进行文本分类：电影评论"
date: 2019-10-09T14:01:25+08:00
tags: ["tensorflow", "keras", "deep learning"]
categories: ["Learn", "TensorFlow"]
draft: false
---

[本笔记](https://www.tensorflow.org/tutorials/keras/text_classification_with_hub)使用评论内容将电影评论分为正面的或负面的。这是二进制（或两类）分类的一个示例，它是一种重要且广泛适用的机器学习问题。

本教程演示了使用TensorFlow Hub和Keras进行迁移学习的基本应用。

我们将使用[IMDB](https://www.tensorflow.org/api_docs/python/tf/keras/datasets/imdb)数据集，其中包含来自网络电影数据库的50,000个电影评论的文本。这些内容分为25,000条用于训练的评论和25,000条用于测试的评论。训练集和测试集是平衡的，这意味着它们包含相同数量的正面和负面评论。

本笔记本使用[tf.keras](https://www.tensorflow.org/guide/keras)，一个高级API在TensorFlow以及[TensorFlow Hub](https://www.tensorflow.org/hub)（用于转移学习的库和平台）中构建和训练模型，。有关使用[tf.keras](https://www.tensorflow.org/api_docs/python/tf/keras)进行更高级的文本分类教程，请参阅[《MLCC文本分类指南》](https://developers.google.com/machine-learning/guides/text-classification/)。

```python
from __future__ import absolute_import, division, print_function, unicode_literals

import numpy as np
import tensorflow as tf

!pip install -q tensorflow-hub
!pip install -q tensorflow-datasets
import tensorflow_hub as hub
import tensorflow_datasets as tfds

print("Version: ", tf.__version__)
print("Eager mode: ", tf.executing_eagerly())
print("Hub version: ", hub.__version__)
print("GPU is", "available" if tf.config.experimental.list_physical_devices("GPU") else "NOT AVAILABLE")

Version:  2.0.0
Eager mode:  True
Hub version:  0.6.0
GPU is available
```

## 下载IMDB数据集

IMDB数据集可以[TensorFlow datasets](https://github.com/tensorflow/datasets)处获取。以下代码将IMDB数据集下载到你的计算机（或colab处）：

```python
# Split the training set into 60% and 40%, so we'll end up with 15,000 examples
# for training, 10,000 examples for validation and 25,000 examples for testing.
train_validation_split = tfds.Split.TRAIN.subsplit([6, 4])

(train_data, validation_data), test_data = tfds.load(
    name="imdb_reviews",
    split=(train_validation_split, tfds.Split.TEST),
    as_supervised=True)

Dataset imdb_reviews downloaded and prepared to /root/tensorflow_datasets/imdb_reviews/plain_text/0.1.0. Subsequent calls will reuse this data.
```

## 研究数据

让我们花一点时间了解下数据格式。每个示例都是代表电影评论的句子和相应的标签。句子不经过任何预处理。标签是0或1的整数值，其中0是负面评论，而1是正面评论。

让我们看看前10个示例。

```python
train_examples_batch, train_labels_batch = next(iter(train_data.batch(10)))
train_examples_batch

<tf.Tensor: id=219, shape=(10,), dtype=string, numpy=
array([b"As a lifelong fan of Dickens, I have invariably been disappointed by adaptations of his novels.<br /><br />Although his works presented an extremely accurate re-telling of human life at every level in Victorian Britain, throughout them all was a pervasive thread of humour that could be both playful or sarcastic as the narrative dictated. In a way, he was a literary caricaturist and cartoonist. He could be serious and hilarious in the same sentence. He pricked pride, lampooned arrogance, celebrated modesty, and empathised with loneliness and poverty. It may be a clich\xc3\xa9, but he was a people's writer.<br /><br />And it is the comedy that is so often missing from his interpretations. At the time of writing, Oliver Twist is being dramatised in serial form on BBC television. All of the misery and cruelty is their, but non of the humour, irony, and savage lampoonery. The result is just a dark, dismal experience: the story penned by a journalist rather than a novelist. It's not really Dickens at all.<br /><br />'Oliver!', on the other hand, is much closer to the mark. The mockery of officialdom is perfectly interpreted, from the blustering beadle to the drunken magistrate. The classic stand-off between the beadle and Mr Brownlow, in which the law is described as 'a ass, a idiot' couldn't have been better done. Harry Secombe is an ideal choice.<br /><br />But the blinding cruelty is also there, the callous indifference of the state, the cold, hunger, poverty and loneliness are all presented just as surely as The Master would have wished.<br /><br />And then there is crime. Ron Moody is a treasure as the sleazy Jewish fence, whilst Oliver Reid has Bill Sykes to perfection.<br /><br />Perhaps not surprisingly, Lionel Bart - himself a Jew from London's east-end - takes a liberty with Fagin by re-interpreting him as a much more benign fellow than was Dicken's original. In the novel, he was utterly ruthless, sending some of his own boys to the gallows in order to protect himself (though he was also caught and hanged). Whereas in the movie, he is presented as something of a wayward father-figure, a sort of charitable thief rather than a corrupter of children, the latter being a long-standing anti-semitic sentiment. Otherwise, very few liberties are taken with Dickens's original. All of the most memorable elements are included. Just enough menace and violence is retained to ensure narrative fidelity whilst at the same time allowing for children' sensibilities. Nancy is still beaten to death, Bullseye narrowly escapes drowning, and Bill Sykes gets a faithfully graphic come-uppance.<br /><br />Every song is excellent, though they do incline towards schmaltz. Mark Lester mimes his wonderfully. Both his and my favourite scene is the one in which the world comes alive to 'who will buy'. It's schmaltzy, but it's Dickens through and through.<br /><br />I could go on. I could commend the wonderful set-pieces, the contrast of the rich and poor. There is top-quality acting from more British regulars than you could shake a stick at.<br /><br />I ought to give it 10 points, but I'm feeling more like Scrooge today. Soak it up with your Christmas dinner. No original has been better realised.",
       b"Oh yeah! Jenna Jameson did it again! Yeah Baby! This movie rocks. It was one of the 1st movies i saw of her. And i have to say i feel in love with her, she was great in this move.<br /><br />Her performance was outstanding and what i liked the most was the scenery and the wardrobe it was amazing you can tell that they put a lot into the movie the girls cloth were amazing.<br /><br />I hope this comment helps and u can buy the movie, the storyline is awesome is very unique and i'm sure u are going to like it. Jenna amazed us once more and no wonder the movie won so many awards. Her make-up and wardrobe is very very sexy and the girls on girls scene is amazing. specially the one where she looks like an angel. It's a must see and i hope u share my interests",
       b"I saw this film on True Movies (which automatically made me sceptical) but actually - it was good. Why? Not because of the amazing plot twists or breathtaking dialogue (of which there is little) but because actually, despite what people say I thought the film was accurate in it's depiction of teenagers dealing with pregnancy.<br /><br />It's NOT Dawson's Creek, they're not graceful, cool witty characters who breeze through sexuality with effortless knowledge. They're kids and they act like kids would. <br /><br />They're blunt, awkward and annoyingly confused about everything. Yes, this could be by accident and they could just be bad actors but I don't think so. Dermot Mulroney gives (when not trying to be cool) a very believable performance and I loved him for it. Patricia Arquette IS whiny and annoying, but she was pregnant and a teenagers? The combination of the two isn't exactly lavender on your pillow. The plot was VERY predictable and but so what? I believed them, his stress and inability to cope - her brave, yet slightly misguided attempts to bring them closer together. I think the characters, acted by anyone else, WOULD indeed have been annoying and unbelievable but they weren't. It reflects the surreality of the situation they're in, that he's sitting in class and she walks on campus with the baby. I felt angry at her for that, I felt angry at him for being such a child and for blaming her. I felt it all.<br /><br />In the end, I loved it and would recommend it.<br /><br />Watch out for the scene where Dermot Mulroney runs from the disastrous counselling session - career performance.",
       b'This was a wonderfully clever and entertaining movie that I shall never tire of watching many, many times. The casting was magnificent in matching up the young with the older characters. There are those of us out here who really do appreciate good actors and an intelligent story format. As for Judi Dench, she is beautiful and a gift to any kind of production in which she stars. I always make a point to see Judi Dench in all her performances. She is a superb actress and a pleasure to watch as each transformation of her character comes to life. I can only be grateful when I see such an outstanding picture for most of the motion pictures made more recently lack good characters, good scripts and good acting. The movie public needs heroes, not deviant manikins, who lack ingenuity and talent. How wonderful to see old favorites like Leslie Caron, Olympia Dukakis and Cleo Laine. I would like to see this movie win the awards it deserves. Thank you again for a tremendous night of entertainment. I congratulate the writer, director, producer, and all those who did such a fine job.',
       b'I have no idea what the other reviewer is talking about- this was a wonderful movie, and created a sense of the era that feels like time travel. The characters are truly young, Mary is a strong match for Byron, Claire is juvenile and a tad annoying, Polidori is a convincing beaten-down sycophant... all are beautiful, curious, and decadent... not the frightening wrecks they are in Gothic.<br /><br />Gothic works as an independent piece of shock film, and I loved it for different reasons, but this works like a Merchant and Ivory film, and was from my readings the best capture of what the summer must have felt like. Romantic, yes, but completely rekindles my interest in the lives of Shelley and Byron every time I think about the film. One of my all-time favorites.',
       b"This was soul-provoking! I am an Iranian, and living in th 21st century, I didn't know that such big tribes have been living in such conditions at the time of my grandfather!<br /><br />You see that today, or even in 1925, on one side of the world a lady or a baby could have everything served for him or her clean and on-demand, but here 80 years ago, people ventured their life to go to somewhere with more grass. It's really interesting that these Persians bear those difficulties to find pasture for their sheep, but they lose many the sheep on their way.<br /><br />I praise the Americans who accompanied this tribe, they were as tough as Bakhtiari people.",
       b'Just because someone is under the age of 10 does not mean they are stupid. If your child likes this film you\'d better have him/her tested. I am continually amazed at how so many people can be involved in something that turns out so bad. This "film" is a showcase for digital wizardry AND NOTHING ELSE. The writing is horrid. I can\'t remember when I\'ve heard such bad dialogue. The songs are beyond wretched. The acting is sub-par but then the actors were not given much. Who decided to employ Joey Fatone? He cannot sing and he is ugly as sin.<br /><br />The worst thing is the obviousness of it all. It is as if the writers went out of their way to make it all as stupid as possible. Great children\'s movies are wicked, smart and full of wit - films like Shrek and Toy Story in recent years, Willie Wonka and The Witches to mention two of the past. But in the continual dumbing-down of American more are flocking to dreck like Finding Nemo (yes, that\'s right), the recent Charlie & The Chocolate Factory and eye-crossing trash like Red Riding Hood.',
       b"I absolutely LOVED this movie when I was a kid. I cried every time I watched it. It wasn't weird to me. I totally identified with the characters. I would love to see it again (and hope I wont be disappointed!). Pufnstuf rocks!!!! I was really drawn in to the fantasy world. And to me the movie was loooong. I wonder if I ever saw the series and have confused them? The acting I thought was strong. I loved Jack Wilde. He was so dreamy to an 10 year old (when I first saw the movie, not in 1970. I can still remember the characters vividly. The flute was totally believable and I can still 'feel' the evil woods. Witchy poo was scary - I wouldn't want to cross her path.",
       b'A very close and sharp discription of the bubbling and dynamic emotional world of specialy one 18year old guy, that makes his first experiences in his gay love to an other boy, during an vacation with a part of his family.<br /><br />I liked this film because of his extremly clear and surrogated storytelling , with all this "Sound-close-ups" and quiet moments wich had been full of intensive moods.<br /><br />',
       b"This is the most depressing film I have ever seen. I first saw it as a child and even thinking about it now really upsets me. I know it was set in a time when life was hard and I know these people were poor and the crops were vital. Yes, I get all that. What I find hard to take is I can't remember one single light moment in the entire film. Maybe it was true to life, I don't know. I'm quite sure the acting was top notch and the direction and quality of filming etc etc was wonderful and I know that every film can't have a happy ending but as a family film it is dire in my opinion.<br /><br />I wouldn't recommend it to anyone who wants to be entertained by a film. I can't stress enough how this film affected me as a child. I was talking about it recently and all the sad memories came flooding back. I think it would have all but the heartless reaching for the Prozac."],
      dtype=object)>
```

让我们也看看前10个标签。

```python
train_labels_batch

<tf.Tensor: id=220, shape=(10,), dtype=int64, numpy=array([1, 1, 1, 1, 1, 1, 0, 1, 1, 0])>
```

## 建立模型

神经网络是通过叠加层来创建的，这需要三个主要的体系结构决策：

- 如何表示文本？
- 在模型中使用多少层？
- 每层使用多少个隐藏单元？

在本例中，输入的数据由句子组成，要预测的标签为0或1。

表示文本的一种方法是将句子转换为嵌入向量。我们可以使用预先训练好的文本嵌入作为第一层，这将具有三个优点：

- 我们不必担心文本预处理，
- 我们可以从迁移学习中受益，
- 嵌入的大小是固定的，因此更容易处理。

在本例中，我们将使用来自[TensorFlow Hub](https://www.tensorflow.org/hub)的一个“预先训练好的文本嵌入模型”[google/tf2-preview/gnews-switve-20dim/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1)。

为了学习本教程，还需要测试其他三个预先训练的模型：

- [google/tf2-preview/gnews-swivel-20dim-with-oov/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim-with-oov/1)：与[google/tf2-preview/gnews-swivel-20dim/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1)相同，但有2.5%的词汇转换为OOV存储。如果任务的词汇表和模型的词汇表没有完全重叠，则可以提供帮助。
- [google/tf2-preview/nnlm-en-dim50/1](https://tfhub.dev/google/tf2-preview/nnlm-en-dim50/1)：一个更大的模型，大约有100万个词汇表和50个维度。
- [google/tf2-preview/nnlm-en-dim128/1](https://tfhub.dev/google/tf2-preview/nnlm-en-dim128/1)：一个巨大的模型，大约有100万个词汇表和128个维度。

首先让我们使用TensorFlow Hub模型创建一个嵌入句子的Keras层，并在几个输入示例中进行尝试。请注意，无论输入文本的长度如何，嵌入的输出形状均为：`(num_examples, embedding_dimension)`。

```python
embedding = "https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1"
hub_layer = hub.KerasLayer(embedding, input_shape=[],
                           dtype=tf.string, trainable=True)
hub_layer(train_examples_batch[:3])

<tf.Tensor: id=402, shape=(3, 20), dtype=float32, numpy=
array([[ 3.9819887 , -4.4838037 ,  5.177359  , -2.3643482 , -3.2938678 ,
        -3.5364532 , -2.4786978 ,  2.5525482 ,  6.688532  , -2.3076782 ,
        -1.9807833 ,  1.1315885 , -3.0339816 , -0.7604128 , -5.743445  ,
         3.4242578 ,  4.790099  , -4.03061   , -5.992149  , -1.7297493 ],
       [ 3.4232912 , -4.230874  ,  4.1488533 , -0.29553518, -6.802391  ,
        -2.5163853 , -4.4002395 ,  1.905792  ,  4.7512794 , -0.40538004,
        -4.3401685 ,  1.0361497 ,  0.9744097 ,  0.71507156, -6.2657013 ,
         0.16533905,  4.560262  , -1.3106939 , -3.1121316 , -2.1338716 ],
       [ 3.8508697 , -5.003031  ,  4.8700504 , -0.04324996, -5.893603  ,
        -5.2983093 , -4.004676  ,  4.1236343 ,  6.267754  ,  0.11632943,
        -3.5934832 ,  0.8023905 ,  0.56146765,  0.9192484 , -7.3066816 ,
         2.8202746 ,  6.2000837 , -3.5709393 , -4.564525  , -2.305622  ]],
      dtype=float32)>
```

现在让我们构建完整的模型：

```python
model = tf.keras.Sequential()
model.add(hub_layer)
model.add(tf.keras.layers.Dense(16, activation='relu'))
model.add(tf.keras.layers.Dense(1, activation='sigmoid'))
model.summary()

Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
keras_layer (KerasLayer)     (None, 20)                400020
_________________________________________________________________
dense (Dense)                (None, 16)                336
_________________________________________________________________
dense_1 (Dense)              (None, 1)                 17
=================================================================
Total params: 400,373
Trainable params: 400,373
Non-trainable params: 0
_________________________________________________________________
```

依次叠加各层以构建分类器：

1. 第一层是TensorFlow Hub层。该层使用预先训练的“保存模型”将句子映射到其嵌入向量中。我们正在使用的预先训练的文本嵌入模型（[google/tf2-preview/gnews-swivel-20dim/1](https://tfhub.dev/google/tf2-preview/gnews-swivel-20dim/1)）将句子拆分为标记，嵌入每个标记，然后组合嵌入。生成的维度是：`(num_examples, embedding_dimension)`。
2. 这个固定长度的输出向量通过一个具有16个隐藏单元的完全连接（密集）层进行传输。
3. 最后一层与单个输出节点紧密连接。使用sigmoid激活函数，该值是0到1之间的浮点数，表示概率或可信度。

让我们编译模型。

**损失函数和优化器**

模型需要损失函数和用于训练的优化器。由于这是一个二元分类问题，并且该模型输出一个概率（具有sigmoid激活的单个单元层），因此我们将使用`binary_crossentropy`损失函数。

这不是损失函数的唯一选择，例如，你可以选择`mean_squared_error`。但是，总的来说，`binary_crossentropy`更适合于处理概率，它可以测量概率分布之间的距离，或者在我们的例子中，测量实地分布与预测之间的距离。

稍后，当我们探讨回归问题（例如，预测房屋价格）时，我们将看到如何使用另一个称为均方差的损失函数。

现在，配置模型以使用优化器和损失函数：

```python
model.compile(optimizer='adam',
              loss='binary_crossentropy',
              metrics=['accuracy'])
```

## 训练模型

在512个样本的迷你批量中训练20个时代(epoch)的模型。这是`x_train`和`y_train`张量中所有样本的20次迭代。训练时，监控来自验证集的10,000个样本的模型损失和准确性：

```python
history = model.fit(train_data.shuffle(10000).batch(512),
                    epochs=20,
                    validation_data=validation_data.batch(512),
                    verbose=1)

Epoch 1/20
30/30 [==============================] - 5s 163ms/step - loss: 0.8001 - accuracy: 0.5715 - val_loss: 0.0000e+00 - val_accuracy: 0.0000e+00
Epoch 2/20
30/30 [==============================] - 4s 121ms/step - loss: 0.6338 - accuracy: 0.6469 - val_loss: 0.6148 - val_accuracy: 0.6665
Epoch 3/20
30/30 [==============================] - 4s 121ms/step - loss: 0.5958 - accuracy: 0.6895 - val_loss: 0.5837 - val_accuracy: 0.6967
Epoch 4/20
30/30 [==============================] - 4s 119ms/step - loss: 0.5609 - accuracy: 0.7275 - val_loss: 0.5494 - val_accuracy: 0.7254
Epoch 5/20
30/30 [==============================] - 4s 118ms/step - loss: 0.5165 - accuracy: 0.7551 - val_loss: 0.5102 - val_accuracy: 0.7507
Epoch 6/20
30/30 [==============================] - 4s 118ms/step - loss: 0.4757 - accuracy: 0.7897 - val_loss: 0.4759 - val_accuracy: 0.7756
Epoch 7/20
30/30 [==============================] - 4s 119ms/step - loss: 0.4391 - accuracy: 0.8167 - val_loss: 0.4465 - val_accuracy: 0.7948
Epoch 8/20
30/30 [==============================] - 4s 117ms/step - loss: 0.4070 - accuracy: 0.8343 - val_loss: 0.4196 - val_accuracy: 0.8135
Epoch 9/20
30/30 [==============================] - 4s 119ms/step - loss: 0.3717 - accuracy: 0.8519 - val_loss: 0.3959 - val_accuracy: 0.8262
Epoch 10/20
30/30 [==============================] - 4s 118ms/step - loss: 0.3451 - accuracy: 0.8658 - val_loss: 0.3758 - val_accuracy: 0.8354
Epoch 11/20
30/30 [==============================] - 4s 119ms/step - loss: 0.3171 - accuracy: 0.8804 - val_loss: 0.3580 - val_accuracy: 0.8449
Epoch 12/20
30/30 [==============================] - 4s 117ms/step - loss: 0.2906 - accuracy: 0.8918 - val_loss: 0.3430 - val_accuracy: 0.8514
Epoch 13/20
30/30 [==============================] - 4s 118ms/step - loss: 0.2742 - accuracy: 0.9005 - val_loss: 0.3311 - val_accuracy: 0.8575
Epoch 14/20
30/30 [==============================] - 4s 118ms/step - loss: 0.2530 - accuracy: 0.9091 - val_loss: 0.3214 - val_accuracy: 0.8618
Epoch 15/20
30/30 [==============================] - 4s 118ms/step - loss: 0.2370 - accuracy: 0.9169 - val_loss: 0.3129 - val_accuracy: 0.8658
Epoch 16/20
30/30 [==============================] - 4s 118ms/step - loss: 0.2196 - accuracy: 0.9228 - val_loss: 0.3068 - val_accuracy: 0.8681
Epoch 17/20
30/30 [==============================] - 4s 118ms/step - loss: 0.2112 - accuracy: 0.9291 - val_loss: 0.3008 - val_accuracy: 0.8726
Epoch 18/20
30/30 [==============================] - 4s 118ms/step - loss: 0.1940 - accuracy: 0.9339 - val_loss: 0.2970 - val_accuracy: 0.8741
Epoch 19/20
30/30 [==============================] - 4s 119ms/step - loss: 0.1827 - accuracy: 0.9388 - val_loss: 0.2945 - val_accuracy: 0.8765
Epoch 20/20
30/30 [==============================] - 4s 117ms/step - loss: 0.1728 - accuracy: 0.9426 - val_loss: 0.2926 - val_accuracy: 0.8771
```

## 评估模型

让我们看看模型的表现。将返回两个值，损失（代表误差的数字，数值越低越好）和准确性。

```python
results = model.evaluate(test_data.batch(512), verbose=2)

for name, value in zip(model.metrics_names, results):
  print("%s: %.3f" % (name, value))

49/49 - 3s - loss: 0.3148 - accuracy: 0.8668
loss: 0.315
accuracy: 0.867
```

这种相当简单的方法可以达到约87%的准确率。如果采用更高级的方法，模型应接近95%。

## 进一步阅读

有关使用字符串输入的更通用方法以及对训练期间准确性和损失进度的更详细的分析，请看[这里](https://www.tensorflow.org/tutorials/keras/basic_text_classification)。
