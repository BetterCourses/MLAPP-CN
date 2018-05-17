# MLAPP 读书笔记 - 04 高斯模型(Gaussian models)

> A Chinese Notes of MLAPP，MLAPP 中文笔记项目 
https://zhuanlan.zhihu.com/python-kivy

记笔记的人：[cycleuser](https://www.zhihu.com/people/cycleuser/activities)

2018年05月16日10:49:49


## 4.1 简介

本章要讲的是多元高斯分布(multivariate Gaussian),或者多元正态分布(multivariate normal ,缩写为MVN)模型,这个分布是对于连续变量的联合概率密度函数建模来说最广泛的模型了.未来要学习的其他很多模型也都是以此为基础的.

然而很不幸的是,本章所要求的数学水平也是比很多其他章节都要高的.具体来说是严重依赖线性代数和矩阵积分.要应对高维数据,这是必须付出的代价.初学者可以跳过标记了星号的章.另外本章有很多等式,其中特别重要的用方框框了起来.

### 4.1.1 记号

这里先说几句关于记号的问题.向量用小写字母粗体表示,比如**x**.矩阵用大写字母粗体表示,比如**X**.大写字母加下标表示矩阵中的项,比如$X_{ij}$.
所有向量都假设为列向量(column vector),除非特别说明是行向量.通过堆叠(stack)D个标量(scalar)得到的类向量记作$[x_1,...,x_D]$.与之类似,如果写**x=**$[x_1,...,x_D]$,那么等号左侧就是一个高列向量(tall column vector),意思就是沿行堆叠$x_i$,一般写作**x=**$(x_1^T,...,x_D^T)^T$,不过这样很丑哈.如果写**X=**$[x_1,...,x_D]$,等好做吧的就是矩阵,意思就是沿列堆叠$x_i$,建立一个矩阵.

### 4.1.2 基础知识

回想一下本书2.5.2中关于D维度下的多元正态分布(MVN)d概率密度函数(pdf)的定义,如下所示:
$N(x|\mu,\Sigma)*= \frac{1}{(2\pi)^{D/2}|\Sigma |^{1/2}}\exp[ -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)]$(4.1 重要公式)


此处参考原书图4.1

指数函数内部的是一个数据向量**x**和均值向量**$\mu$**之间的马氏距离(马哈拉诺比斯距离,Mahalanobis distance).对$\Sigma$进行特征分解(eigendecomposition)有助于更好理解这个量.$\Sigma = U\wedge U ^T$,其中的U是标准正交矩阵(orthonormal matrix),满足$U^T U = I$,而$\wedge $是特征值组成的对角矩阵.经过特征分解就得到了:

$\Sigma^{-1}=U^{-T}\wedge^{-1}U^{-1}=U\wedge ^{-1}U^T=\sum^D_{i=1}\frac{1}{\lambda_i}u_iu_i^T$(4.2)

上式中的$u_i$是U的第i列,包含了第i个特征向量(eigenvector).因此就可以把马氏距离写作:


$$\begin{aligned}
(x-\mu)^T\Sigma^{-1}(x-\mu)&=(x-\mu)^T(\sum^D_{i=1}\frac{1}{\lambda_i}u_iu_i^T)(x-\mu)&\text{(4.3)}\\
&= \sum^D_{i=1}\frac{1}{\lambda_i}(x-\mu)^Tu_iu_i^T(x-\mu)=\sum^D_{i=1}\frac{y_i^2}{\lambda_i}&\text{(4.4)}\\
\end{aligned}$$

上式中的$y_i*= u_i^T(x-\mu)$.二维椭圆方程为:
$\frac{y_1^2}{\lambda_1}+\frac{y_2^2}{\lambda_2}=1$(4.5)

因此可以发现高斯分布的概率密度的等值线沿着椭圆形,如图4.1所示.特征向量决定了椭圆的方向,特征值决定了椭圆的形态即宽窄比.

一般来说我们将马氏距离(Mahalanobis distance)看作是对应着变换后坐标系中的欧氏距离(Euclidean distance),平移$\mu$,旋转U.

### 4.1.3 多元正态分布(MVN)的最大似然估计(MLE)

接下来说的是使用最大似然估计(MLE)来估计多元正态分布(MVN)的参数.在后面的章节里面还会说道用贝叶斯推断来估计参数,能够减轻过拟合,并且能对估计值的置信度提供度量.

#### 定理4.1.1(MVN的MLE)
如果有N个独立同分布样本符合正态分布,即$x_i ∼ N(\mu,\Sigma)$,则对参数的最大似然估计为:
$\hat\mu_{mle}=\frac{1}{N}\sum^N_{i=1}x_i *= \bar x$(4.6)
$\hat\Sigma_{mle}=\frac{1}{N}\sum^N_{i=1}(x_i-\bar x)(x_i-\bar x)^T=\frac{1}{N}(\sum^N_{i=1}x_ix_i^T)-\bar x\bar x^T$(4.7)

也就是MLE就是经验均值(empirical mean)和经验协方差(empirical covariance).在单变量情况下结果就很熟悉了:
$\hat\mu =\frac{1}{N}\sum_ix_i=\bar x $(4.8)
$\hat\sigma^2 =\frac{1}{N}\sum_i(x_i-x)^2=(\frac{1}{N}\sum_ix_i^2)-\bar x^2$(4.9)

#### 4.1.3.1 证明

要证明上面的结果,需要一些矩阵代数的计算,这里总结一下.等式里面的**a**和**b**都是向量,**A**和**B**都是矩阵.记号$tr(A)$表示的是矩阵的迹(trace),是其对角项求和,即$tr(A)=\sum_i A_{ii}$.

$$
\begin{aligned}
\frac{\partial(b^Ta)}{\partial a}&=b\\
\frac{\partial(a^TAa)}{\partial a}&=(A+A^T)a\\
\frac{\partial}{\partial A} tr(BA)&=B^T\\
\frac{\partial}{\partial A} \log|A|&=A^{-T}*= (A^{-1})^T\\
tr(ABC)=tr(CAB)&=tr(BCA)
\end{aligned}
$$(4.10 重要公式)


上式中最后一个等式也叫做迹运算的循环置换属性(cyclic permutation property).利用这个性质可以推广出很多广泛应用的求迹运算技巧,对标量内积$x^TAx$就可以按照如下方式重新排序:
$x^TAx=tr(x^TAx)=tr(xx^TA)=tr(Axx^T)$(4.11)


##### 证明过程
接下来要开始证明了,对数似然函数为:

$l(\mu,\Sigma)=\log p(D|\mu,\Sigma)=\frac{N}{2}\log|\wedge| -\frac{1}{2}\sum^N_{i=1}(x_i-\mu)^T\wedge (x_i-\mu) $(4.12)

上式中$\wedge=\Sigma^{-1}$,是精度矩阵(precision matrix)

然后进行一个替换(substitution)$y_i=x_i-\mu$,再利用微积分的链式法则:

$$
\begin{aligned}
\frac{\partial}{\partial\mu} (x_i-\mu)^T\Sigma^{-1}(x_i-\mu) &=  \frac{\partial}{\partial y_i}y_i^T\Sigma^{-1}y_i\frac{\partial y_i}{\partial\mu}   &\text{(4.13)}\\
&=-1(\Sigma_{-1}+\Sigma^{-T})y_i &\text{(4.14)}\\
\end{aligned}
$$

因此:

$$
\begin{aligned}
\frac{\partial}{\partial\mu}l(\mu.\Sigma) &= -\frac{1}{2} \sum^N_{i=1}-2\Sigma^{-1}(x_i-\mu)=\Sigma^{-1}\sum^N_{i=1}(x_i-\mu)=0 &\text{(4.15)}\\
&=-1(\Sigma_{-1}+\Sigma^{-T})y_i &\text{(4.16)}\\
\end{aligned}
$$
所以$\mu$的最大似然估计(MLE)就是经验均值(empirical mean).

然后利用求迹运算技巧(trace-trick)来重写对$\wedge$的对数似然函数:
$$
\begin{aligned}
l(\wedge)&=  \frac{N}{2}\log|\wedge|-\frac{1}{2}\sum_i tr[(x_i-\mu)(x_i-\mu)^T\wedge] &\text{(4.17)}\\
&= \frac{N}{2}\log|\wedge| -\frac{1}{2}tr[S_{\mu}\wedge]&\text{(4.18)}\\
& &\text{(4.19)}\\
\end{aligned}
$$

上式中
$S_{\mu}*= \sum^N_{i=1}(x_i-\mu)(x_i-\mu)^T$(4.20)

是以$\mu$为中心的一个散布矩阵(scatter matrix).对上面的表达式关于$\wedge$进行求导就得到了:
$$
\begin{aligned}
\frac{\partial l(\wedge)}{\partial\wedge} & = \frac{N}{2}\wedge^{-T} -\frac{1}{2}S_{\mu}^T=0 &\text{(4.21)}\\
\wedge^{-T} & = \wedge^{-1}=\Sigma=\frac{1}{N}S_{\mu} &\text{(4.22)}\\
\end{aligned}
$$

因此有:
$\hat\Sigma=\frac{1}{N}\sum^N_{i=1}(x_i-\mu)(x_i-\mu)^T$(4.23)

正好也就是以$\mu$为中心的经验协方差矩阵(empirical covariance matrix).如果插入最大似然估计$\mu=\bar x$(因为所有参数都同时进行优化),就得到了协方差矩阵的最大似然估计的标准方程.


### 4.1.4 高斯分布最大熵推导(Maximum entropy derivation of the Gaussian)*

在本节,要证明的是多元高斯分布(multivariate Gaussian)是适合于有特定均值和协方差的具有最大熵的分布(参考本书9.2.6).这也是高斯分布广泛应用到一个原因,均值和协方差这两个矩(moments)一般我们都能通过数据来进行估计得到(注:一阶矩（期望）归零，二阶矩（方差）),所以我们就可以使用能捕获这这些特征的分布来建模,另外还要尽可能少做附加假设.

为了简单起见,假设均值为0.那么概率密度函数(pdf)就是:
$p(x)=\frac{1}{Z}\exp (-\frac{1}{2}x^T\Sigma^{-1}x)$(4.24)

如果定义$f_{ij} (x) = x_i x_j , \lambda_{ij} = \frac{1}{2} (\Sigma^{−1})_{ij}\\i, j \in \{1, ... , D\}$,就会发现这个和等式9.74形式完全一样.这个分布（使用自然底数求对数）的（微分）熵为:
$h(N(\mu,\Sigma))  =\frac{1}{2}\ln[(2\pi e)^D|\Sigma|]$(4.25)

接下来要证明有确定的协方差$\Sigma$的情况下多元正态分布(MVN)在所有分布中有最大熵.

#### 定理 4.1.2

设$q(x)$是任意的一个密度函数,满足$\int q(x)x_ix_j=\Sigma_{ij}$.设$p=N(0,\Sigma)$.那么$h(q)\le h(p)$.

证明.(参考(Cover and Thomas 1991, p234)).
(注:KL是KL 散度(Kullback-Leibler divergence),也称相对熵(relative entropy),可以用来衡量p和q两个概率分布的差异性(dissimilarity).更多细节参考2.8.2.)

$$
\begin{aligned}
0 &\le KL(q||p) =\int q(x)\log \frac{q(x)}{p(x)}dx&\text{(4.26)}\\
& = -h(q) -\int q(x)\log p(x)dx &\text{(4.27)}\\
& =* -h(q) -\int ps(x)\log p(x)dx &\text{(4.28)}\\
& = -h(q)+h(p) &\text{(4.29)}\\
\end{aligned}
$$


等式4.28那里的星号表示这一步是关键的,因为q和p对于由$\log p(x)$编码的二次形式产生相同的矩(moments).

## 4.2 高斯判别分析(Gaussian discriminant analysis)

多元正态分布的一个重要用途就是在生成分类器中定义类条件密度,也就是:
$p(x|y=c,\theta)=N(x|\mu_c,\Sigma_c)$(4.30)

这样就得到了高斯判别分析,也缩写为GDA,不过这其实还是生成分类器(generative classifier）,而并不是辨别式分类器（discriminative classifier）,这两者的区别参考本书8.6.如果\Sigma_c$$是对角矩阵,那这就等价于朴素贝叶斯分类器了.


此处参考原书图4.2

从等式2.13可以推导出来下面的决策规则,对一个特征向量进行分类:
$\hat y(x)=\arg \max_c[\log  p(y=c|\pi)  +\log p(x|\theta_c)] $(4.31)

计算x 属于每一个类条件密度的概率的时候,测量的距离是x到每个类别中心的马氏距离(Mahalanobis distance).这也是一种最近邻质心分类器(nearest centroids classiﬁer).

例如图4.2展示的就是二维下的两个高斯类条件密度,横纵坐标分别是身高和体重,包含了男女两类人.很明显身高体重这两个特征有相关性,就如同人们所想的,个子高的人更可能重.每个分类的椭圆都包含了95%的概率质量.如果对两类有一个均匀分布的先验,就可以用如下方式来对新的测试向量进行分类:


$\hat y(x)=\arg \max_c(x-\mu_c)^T\Sigma_c^{-1}(x-\mu_c) $(4.32)


### 4.2.1 二次判别分析(Quadratic discriminant analysis,QDA)

对类标签的后验如等式2.13所示.加入高斯密度定义后,可以对这个模型获得更进一步的理解:
$p(y=c|x,\theta)  =\frac{ \pi_c|2\pi\Sigma_c|^{-1/2} \exp [-1/2(x-\mu_c)^T\Sigma_c^{-1}(x-\mu_c)]   }{   \Sigma_{c'}\pi_{c'}|2\pi\Sigma_{c'}|^{-1/2} \exp [-1/2(x-\mu_{c'})^T\Sigma_{c'}^{-1}(x-\mu_{c'})]    }$(4.33)

对此进行阈值处理(thresholding)就得到了一个x的二次函数(quadratic function).这个结果也叫做二次判别分析(quadratic discriminant analysis,缩写为QDA).图4.3所示的是二维平面中决策界线的范例.


此处参考原书图4.3

此处参考原书图4.4


### 4.2.2 线性判别分析(Linear discriminant analysis,LDA)

接下来考虑一种特殊情况,此事协方差矩阵为各类所共享(tied or shared),即$\Sigma_c=\Sigma$.这时候就可以把等式4.33简化成虾米这样:
$$
\begin{aligned}
p(y=c|x,\theta)&\propto \pi_c\exp [\mu_c^T\Sigma^{-1}x-\frac12 x^T\Sigma^{-1}x - \frac12\mu_c^T\Sigma^{-1}\mu_c]&\text{(4.34)}\\
& = \exp [\mu_c^T\Sigma^{-1}x-\frac12 \mu_c^T\Sigma^{-1}\mu_c+\log\pi_c]\exp [-\frac12 x^T\Sigma^{-1}x]&\text{(4.35)}\\
\end{aligned}
$$

由于二次项$x^T\Sigma^{-1}$独立于类别c,所以可以抵消掉分子分母.如果定义了:


$$
\begin{aligned}
\gamma_c &= -\frac12\mu-c^T\Sigma^{-1}\mu_c+\log\pi_c&\text{(4.36)}\\
&\text{(4.37)}\\
\beta_c &= \Sigma^{-1}\mu_c\end{aligned}
$$

则有:
$p(y=c|x,\theta)=\frac{e^{\beta^T_c+\gamma_c}}{\Sigma_{c'}e^{\beta^T_{c'}+\gamma_{c'}}}=S(\eta)_c$(4.38)

当$\eta =[\beta^T_1x+\gamma_1,...,\beta^T_Cx+\gamma_C]$的时候,$S$就是Softmax函数(softmax function,注:柔性最大函数,或称归一化指数函数),其定义如下:
$S(\eta/T)= \frac{e^{\eta_c}}{\sum^C_{c'=1}e^{\eta_{c'}}}$(4.39)

Softmax函数如同其名中的Max所示,有点像最大函数.把每个$\eta_c$除以一个常数T,这个常数T叫做温度(temperature).然后让T趋于零,即$T\rightarrow 0$,则有:

$$S(\eta/T)_c=\begin{cases} 1.0&\text{if } c = \arg\max_{c'}\eta_{c'}\\
0.0 &\text{otherwise}\end{cases} 
$$(4.40)

