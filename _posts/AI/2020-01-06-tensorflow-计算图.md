---
type: article
title:  "[tensorflow]计算图概念，来自公众号"
categories: AI
tags: AI tensorflow
author: DengYuting
---

## 三种张量类型

-  tf.Variable
> 可以包含神经网络的权重，它们会在训练期间改变，以便为特定问题找到最佳值
-  tf.constant
> 整体运行过程不会改变
-  tf.placeholder
> 占位符，可以包含要用于训练神经网络的数据集，一旦赋值，它就不会在计算阶段发生变化。

## Tensorflow计算步骤

必须始终遵循三个步骤来计算图  

1. 创建计算图
2. 然后创建会话
3. 最后运行图

## 三种张量的计算图例子

- tf.constant

```python
x1 = tf.constant(1) 
x2 = tf.constant(2) 
z = tf.add(x1, x2)
sess = tf.Session() 
print(sess.run(z))
```

- tf.placeholder

```python
x1 = tf.placeholder(tf.float32, 1) 
x2 = tf.placeholder(tf.float32, 1)
z = tf.add(x1,x2)
sess = tf.Session()
feed_dict={ x1: [1], x2: [2]}
print(sess.run(z, feed_dict))

# 多维情况的例子
x1 = tf.placeholder(tf.float32, [2]) 
x2 = tf.placeholder(tf.float32, [2])
z = tf.add(x1,x2) 
feed_dict={ x1: [1,5], x2: [1,1]}
sess = tf.Session() 
sess.run(z, feed_dict)
```

- tf.variable

```python
x1 = tf.Variable(1) 
x2 = tf.Variable(2) 
z = tf.add(x1,x2)
sess = tf.Session()
print(sess.run(z))

# 上面直接计算会报错，因为tf.variable需要初始化
sess = tf.Session() 
sess.run(x1.initializer) 
sess.run(x2.initializer) 
print(sess.run(z))

# 简单一点
init = tf.global_variables_initializer()
sess = tf.Session() 
sess.run(init) 
print(sess.run(z)) 
sess.close()
```

## 参考

<a href="https://mp.weixin.qq.com/s/vnozl5gEvIlGeC1Flt5pTQ"> TensorFlow是什么？怎么用？终于有人讲明白了[Python数据之道 - 公众号] </a>