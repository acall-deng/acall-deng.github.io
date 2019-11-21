---
layout: post
title:  "线性代数的本质-小结"
categories: 数学基础
tags: 数学基础 线性代数
author: DengYuting
---
* content
{:toc}

# 简介  
线性代数本质理解为坐标的变换。





# 所有章节   

### 1. 向量究竟是什么?   
   向量是空间中的一个有方向的线段，始终从原点出发。

### 2. 线性组合、张成的空间与基  
   可以认为向量都是在空间中由基底向量线性组合而成的一个有方向的线段  
   张成空间为：通过基底的线性组合所扩展出的整个空间，例如在二维场景中，不在同一直线上的两个向量的张成空间是整个二维平面。

### 3. 矩阵与线性变换  
   线性变换能保证空间中的『所有平行的轴(网格线)之间的间距保持一致，所有点也是均匀分布的』，且『原点位置保持不变』否则为非线性变换。  

### 4. 矩阵乘法与线性变换复合  
   本质为线性变换，变换基向量的位置，每一列代表新的基向量。  

### 5. 三维空间中的线性变换  
   类似于二维情况下的变换，三维空间中的变换也可以理解为对基向量做的改变。  
   
### 6. 行列式  
   行列式的值为这几个向量所『包含的面积/体积』，高维空间依次类推。如果发生轴的翻转，则行列式的值为负数。  
   > 注意：m*m大小矩阵如果行列式的值为0，意味着其中至少有两个向量是线性相关的，其被压缩到了一个更低的维度上。例如二维空间被压缩到了一维上，则其面积肯定是0。  

### 7. 逆矩阵、列空间与零空间  
   这一节中提出了矩阵的秩的概念，矩阵的秩代表的是这个多维矩阵中的向量的张成空间维度是多少。
   > 这里综合一下前面的内容，在求解线性方程组的时候，可以视为对[x y z]这个向量施加了一个A的变换，最后得到v向量，此时，如果A的张成空间为二维，则当且仅当v向量在这个二维平面的时候有解。

   ![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/1.png)

### 8. 非方阵  

   对于m*n的非方阵，可视为n个向量。如果m < n，这意味着m维空间里只有n个向量，则这个变换是降维的。反之，如果 m > n，则是把二维的向量映射到三维空间上，但其张成空间仍然是个二维平面。  

   ![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/2.png)

### 9. 点积与对偶性  

点积的几何意义是投影。因为把这个向量投影到一维数轴上，然后进行变换。    

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/3.png)    

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/4.png)    

至于说的对偶性，应该指的是这个：任何一个二维到一维的线性变换，它都与某个向量相关。这个角度上，应用变换和做点积是一样的。     

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/5.png)    

### 10. 叉积的标准介绍    

叉积（向量积）的标准几何意义是垂直于两个向量组成空间的一个向量，这个新向量的长度为这两个原始向量围成的面积。公式很难记，在下一节里面说明了怎么理解。   

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/6.png)   

### 11. 以线性代数的眼光看叉积  

叉积的计算如下：    

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/7.png)  

此时定义一个函数，将v和w为定值，[x y z]为变量，由于叉积的几何意义是投影，可以见得下面这个式子成立的标准是p向量是v和w组成的平面的垂直方向且长度为v和w的面积，如下面的第二张图，这样[x y z]在上面的投影就是垂直于v和w组成的平面，其乘积为[x y z]和v、w共同组成的三维立体的体积。  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/8.png)  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/9.png)  

把[x y z]换成[i j k]就是最后的坐标了，从而，叉积可以理解为两个向量垂直方向，长度等于行列式的一个向量，正负号取决于右手定则。

### 12. 基变换  

最关键的一个过程如下图所示。假设从A基向量转换为B基向量，则基变换等同于，把在A基向量下的一个向量先转为B基向量表示下的坐标，然后应用变换，再转换回A基向量表示下的坐标。  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/10.png)  

### 13. 特征向量与特征值  

特征向量的本质在进行变换的时候仍然保持原方向，只在其张成空间内移动，没有发生偏移的一些向量。举个例子说明意义：对于一个三维物体的旋转来说，其特征向量就是其转轴！  

所以，对于A的特征向量v施加A变换时，其转换后的结果相当于对v向量进行了一定倍数的拉伸，而这个倍数就是特征值，如下图。  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/11.png)  

求解特征向量和特征值的作用在于，如果一个向量以变换矩阵的特征向量作为基底，则在这个基底之下的变换一定是个对角矩阵，因为基底只发生线性的拉长或者缩短而没有位置的偏移。具体的转换方式如下图所示。  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/12.png)  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/13.png)  


### 14. 抽象向量空间  

把求导看成是一次矩阵的变换，此时的坐标取坐标为下图中的抽象的坐标系，则求导就转换为了下面第二张图中的矩阵计算。很神奇  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/14.png)  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/15.png)  

### 15. 克莱姆法则  
    
用于求解原始的坐标。以二维场景下的为例，此时的做法是，把变换的其中一个向量作为固定值，另外一个向量当成是未知的量，则这个未知量的位置会影响到这个二维图形的面积。  
      
基于这个原则，将原始的横坐标（原始矩阵A第一列）当固定向量，结果向量为第二列的矩阵的行列式值（变换后的面积）除以原始矩阵delt(A)的值即为y，就能得到纵坐标（原始矩阵A第二列）缩放的倍数，因为横坐标没有变，那么纵坐标的变化倍数就等于面积的变化倍数。这个也很神奇的说。  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/16.png)  

![](https://raw.githubusercontent.com/acall-deng/acall-deng.github.io/master/_posts/img/2019-11-16-linear-algebra/17.png)  