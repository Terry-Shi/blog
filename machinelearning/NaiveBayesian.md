## 贝叶斯原理的解释和应用
### 贝叶斯公式
贝叶斯公式定义及其推导过程：
http://baike.baidu.com/view/541856.htm

贝叶斯定理用来描述两个条件概率之间的关系，比如 P(A|B) 和 P(B|A)。按照乘法法则：P(A∩B) = P(A)*P(B|A)=P(B)*P(A|B)，可以立刻导出。如上公式也可变形为：P(B|A) = P(A|B)*P(B) / P(A)。

![公式图](http://g.hiphotos.baidu.com/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=c05cea45d209b3deffb2ec3aadd607e4/dbb44aed2e738bd4250eff1ca28b87d6277ff9f4.jpg)

贝叶斯分类算法：
http://baike.baidu.com/view/2579342.htm
设每个数据样本用一个n维特征向量来描述n个属性的值，即：X={x1，x2，…，xn}，假定有m个类，分别用C1, C2,…，Cm表示。给定一个未知的数据样本X（即没有类标号），若朴素贝叶斯分类法将未知的样本X分配给类Ci，则一定是 P(Ci|X)>P(Cj|X) 1≤j≤m，j≠i

根据贝叶斯定理 P(Ci|X) = P(X|Ci)P(Ci) / P(X)
由于计算所有类的分母都是P(X)，最大化后验概率P(Ci|X)可转化为最大化先验概率P(X|Ci)P(Ci)。如果训练数据集有许多属性和元组，计算P(X|Ci)的开销可能非常大，为此，通常假设各属性的取值互相独立，这样
先验概率P(x1|Ci)，P(x2|Ci)，…，P(xn|Ci)可以从训练数据集求得。
根据此方法，对一个未知类别的样本X，可以先分别计算出X属于每一个类别Ci的概率P(X|Ci)P(Ci)，然后选择其中概率最大的类别作为其类别。
朴素贝叶斯算法成立的前提是各属性之间互相独立。当数据集满足这种独立性假设时,分类的准确度较高，否则可能较低。另外，该算法没有分类规则输出。

### 贝叶斯推断及其应用过滤垃圾邮件
Ref: (硕士学位论文 －基于贝叶斯的中文垃圾邮件过滤系统的设计与实现)
http://www.doc88.com/p-481336694094.html

*:贝叶斯网络
A Bayesian Approach to Filtering Junk E-Mail ftp://ftp.research.microsoft.com/pub/ejh/junkfilter.pdf

*:马尔可夫模型


