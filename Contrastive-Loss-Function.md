# Dimensionality Reduction by Learning an Invariant Mapping

Raia Hadsell, Sumit Chopra, Yann LeCun The Courant Institute of Mathematical Sciences

## 0. Abstract

Dimensionality reduction involves mapping a set of high dimensional input points onto a low dimensional manifold so that “similar” points in input space are mapped to nearby points on the manifold. Most existing techniques for solving the problem suffer from two drawbacks. First, most of them depend on a meaningful and computable distance metric in input space. Second, they do not compute a “function” that can accurately map new input samples whose relationship to the training data is unknown. We present a method - called Dimensionality Reduction by Learning an Invariant Mapping (DrLIM) - for learning a globally coherent non-linear function that maps the data evenly to the output manifold. The learning relies solely on neighborhood relationships and does not require any distance measure in the input space. The method can learn mappings that are invariant to certain transformations of the inputs, as is demonstrated with a number of experiments. Comparisons are made to other techniques, in particular LLE.

降维是将高维度的输入点集映射到一个低维度的流形，这样输入空间中类似的点，映射到了流形中的相近的点。解决这个问题的多数已有技术，都有两个缺点。第一，多数依靠在输入空间中有意义的可计算的距离度量。第二，它们都没有计算一个得到一个函数，可以准确的将新的输入样本进行映射，新样本与训练数据之间的关系是未知的。我们提出一种方法，称为通过学习不变映射的降维(DrLIM)，以学习一个全局连续的非线性函数，将数据均匀的映射到输出流形中。这个学习只依靠邻域的关系，不需要任何输入空间中的距离度量。这个方法可以学习到对输入的特定变换不变的映射，这在几个试验中都进行了证实。和其他技术进行了对比，特别是LLE。

## 1. Introduction

Modern applications have steadily expanded their use of complex, high dimensional data. The massive, high dimensional image datasets generated by biology, earth science, astronomy, robotics, modern manufacturing, and other domains of science and industry demand new techniques for analysis, feature extraction, dimensionality reduction, and visualization.

现代应用非常多的应用复杂的、高维度的数据。大量高维度的图像数据集，在生物学、地球科学、天文学、机器人、现代制造和其他科学领域和工业应用中产生出来，需要新的分析、特征提取、降维和可视化的技术。

Dimensionality reduction aims to translate high dimensional data to a low dimensional representation such that similar input objects are mapped to nearby points on a manifold. Most existing dimensionality reduction techniques have two shortcomings. First, they do not produce a function (or a mapping) from input to manifold that can be applied to new points whose relationship to the training points is unknown. Second, many methods presuppose the existence of a meaningful (and computable) distance metric in the input space.

降维的目标是将高维度数据转变到一个低维度的表示，这样类似的输入目标可以映射到流形中的近邻点上。多数已有的降维技术有两种缺陷。第一，它们没有得到一个从输入到流形的函数（或映射），以应用到新的点上，这里的数据点与训练数据的关系是未知的。第二，很多方法假设有在输入空间中有一个有意义的距离度量。

For example, Locally Linear Embedding (LLE) [15] linearly combines input vectors that are identified as neighbors. The applicability of LLE and similar methods to image data is limited because linearly combining images only makes sense for images that are perfectly registered and very similar. Laplacian Eigenmap [2] and Hessian LLE [8] do not require a meaningful metric in input space (they merely require a list of neighbors for every sample), but as with LLE, new points whose relationships with training samples are unknown cannot be processed. Out-of-sample extensions to several dimensionality reduction techniques have been proposed that allow for consistent embedding of new data samples without recomputation of all samples [3]. These extensions, however, assume the existence of a computable kernel function that is used to generate the neighborhood matrix. This dependence is reducible to the dependence on a computable distance metric in input space.

比如，局部线性嵌入(LLE)将识别为相邻的输入向量线性结合到一起。LLE和类似方法在图像数据中的应用是有限的，因为将图像线性结合到一起，只对完全配准的并且非常类似的图像有意义。Laplacian Eigenmap [2]和Hessian LLE [8]在输入空间中不需要有意义的度量（它们只对每个样本需要一个相邻列表），但对于LLE，与训练样本关系不明的新的点，不能被处理。对几个降维技术的样本不足情况的拓展提出了几种技术，使得在不需要重新计算所有样本的时候可以对新的数据样本进行一致嵌入[3]。但是，这些拓展假设存在一个可以计算的核函数，可用于生成邻域矩阵。这种依赖性可以简化成在输入空间中对一个可计算的距离度量的依赖。

Another limitation of current methods is that they tend to cluster points in output space, sometimes densely enough to be considered degenerate solutions. Rather, it is sometimes desirable to find manifolds that are uniformly covered by samples. 目前方法的一个限制是，它们倾向于在输出空间中对点进行聚类，有时密集到足以认为是降质解。有时候，发现流形中均匀分布着样本，是很理想的情况。

The method proposed in the present paper, called Dimensionality Reduction by Learning an Invariant Mapping (DrLIM), provides a solution to the above problems. DrLIM is a method for learning a globally coherent non-linear function that maps the data to a low dimensional manifold. The method presents four essential characteristics:

本文提出的方法，称为DrLIM，对上述问题提出了一种解决方案。DrLIM是学习一种全局连续的非线性函数的方法，将数据映射到一个低维流形中。这种方法有四种重要的性质：

- It only needs neighborhood relationships between training samples. These relationships could come from prior knowledge, or manual labeling, and be independent of any distance metric. 它只需要训练样本之间的相邻关系。这些关系可以是从先验知识中来，或手动标注，与任何距离度量都没有关系。

- It may learn functions that are invariant to complicated non-linear trnasformations of the inputs such as lighting changes and geometric distortions. 可以学习到对复杂的非线性变换不变的函数，如光照变化和几何形变。

- The learned function can be used to map new samples not seen during training, with no prior knowledge. 学习到的函数可以用于映射新的样本，这些样本在训练时未曾见过。

- The mapping generated by the function is in some sense “smooth” and coherent in the output space. 这个函数生成的映射，在某种意义上，在输出空间中是平滑和一致的。

A contrastive loss function is employed to learn the parameters W of a parameterized function $G_W$, in such a way that neighbors are pulled together and non-neighbors are pushed apart. Prior knowledge can be used to identify the neighbors for each training data point.

采用了一个对比损失函数，来学习参数化函数$G_W$的参数W，这个函数可以使相邻的样本拉到一起，不是相邻的样本分开。先验知识可以用于判定，对每个训练数据点是否是相邻的。

The method uses an energy based model that uses the given neighborhood relationships to learn the mapping function. For a family of functions G, parameterized by W, the objective is to find a value of W that maps a set of high dimensional inputs to the manifold such that the euclidean distance between points on the manifold, $D_W (\vec X_1, \vec X2) = ||G_W (\vec X_1) − G_W (\vec X_2)||_2$ approximates the “semantic similarity”of the inputs in input space, as provided by a set of neighborhood relationships. No assumption is made about the function $G_W$ except that it is differentiable with respect to W.

本方法使用一种基于能量的模型，使用给定的相邻关系来学习这个映射函数。对于一族函数G，其参数为W，目标是找到W的值，将高维输入映射到流形上，以使在各点在流形上的欧式距离，$D_W (\vec X_1, \vec X2) = ||G_W (\vec X_1) − G_W (\vec X_2)||_2$，近似输入空间中输入的语义相似性，这是由相邻关系的集合给定的。关于函数$G_W$没有什么假设，只有对W是可微的假设。

### 1.1. Previous Work

The problem of mapping a set of high dimensional points onto a low dimensional manifold has a long history. The two classical methods for the problem are Principal Component Analysis (PCA) [7] and Multi-Dimensional Scaling (MDS) [6]. PCA involves the projection of inputs to a low dimensional subspace that maximizes the variance. In MDS, one computes the projection that best preserves the pairwise distances between input points. However both the methods - PCA in general and MDS in the classical scaling case (when the distances are euclidean distances) - generate a linear embedding.

将高维点集映射到低维流形的问题，有一个很长的历史。对这个问题有两个经典的解决方法，分别是主成分分析(PCA)和多维缩放(MDS)。PCA是将输入投影到一个低维的子空间，对方差进行最大化。在MDS中，计算的投影，是能最好的保持输入点之间的成对距离的。但是，这两种方法产生的都是线性嵌入。

In recent years there has been a lot of activity in designing non-linear spectral methods for the problem. These methods involve solving the eigenvalue problem for a particular matrix. Recently proposed algorithms include ISOMAP (2000) by Tenenbaum et al. [1], Local Linear Embedding - LLE (2000) by Roweis and Saul [15], Laplacian Eigenmaps (2003) due to Belkin and Niyogi [2] and Hessian LLE (2003) by Donoho and Grimes [8]. All the above methods have three main steps. The first is to identify a list of neighbors of each point. Second, a gram matrix is computed using this information. Third, the eigenvalue problem is solved for this matrix. The methods differ in how the gram matrix is computed. None of these methods attempt to compute a function that could map a new, unknown data point without recomputing the entire embedding and without knowing its relationships to the training points. Out-of-sample extensions to the above methods have been proposed by Bengio et al. in [3], but they too rely on a predetermined computable distance metric.

最近几年，对这个问题设计非线性的谱方法，有很多活动。这些方法一般是对特定的矩阵求解特征值问题。最近提出的算法包括ISOMAP，LLE，Laplacian Eigenmaps和Hessian LLE。所有上述方法都有三个主要步骤，第一是识别出每个点的邻域列表。第二，使用这些信息计算一个gram矩阵。第三，对这个矩阵求解特征值问题。这些方法不同的地方在于，gram矩阵是如何计算得到的。这些方法都没有计算得到一个函数，能将一个新的，未曾见过的数据点，在不计算整个嵌入和不知道其与训练点之间关系时进行映射。Bengio等[3]提出了这些方法在样本外的拓展，但他们同样依赖于一个预确定好的可计算的距离度量。

Along a somewhat different line Schoelkopf et al. in 1998 [13] proposed a non-linear extension of PCA, called Kernel PCA. The idea is to non-linearly map the inputs to a high dimensional feature space and then extract the principal components. The algorithm first expresses the PCA computation solely in terms of dot products and then exploits the kernel trick to implicitly compute the high dimensional mapping. The choice of kernels is crucial: different kernels yield dramatically different embeddings. In recent work, Weinberger et al. in [11, 12] attempt to learn the kernel matrix when the high dimensional input lies on a low dimensional manifold by formulating the problem as a semidefinite program. There are also related algorithms for clustering due to Shi and Malik [14] and Ng et al. [17].

在一条不同的线上，Schoelkopf[13]等提出了一个PCA的非线性拓展，称为核PCA。其思想是，将输入非线性的映射到一个高维特征空间，然后提出主要分量。算法首先只用点积来表达PCA的计算，然后使用核的技巧来隐式的计算高维映射。核的选择是很关键的：不同的核会生成非常不同的嵌入。在最近的工作中，Weinberger等[11,12]试图在高维输入在低维流形上时，通过将问题表述为一个半定问题，学习核矩阵。也有相关的算法进行聚类。

The proposed approach is different from these methods; it learns a function that is capable of consistently mapping new points unseen during training. In addition, this function is not constrained by simple distance measures in the input space. The learning architecture is somewhat similar to the one discussed in [4, 5].

我们提出的方法与这些方法不同；它学习到的是一个函数，可以一直将新的在训练时未曾见到的点进行映射。另外，这个函数不受到输入空间中简单的距离度量的约束。学习框架与[4,5]中讨论的有某些类似。

Section 2 describes the general framework, the loss function, and draws an analogy with a mechanical spring system. The ideas in this section are made concrete in section 3. Here various experimental results are given and comparisons to LLE are made. 第2部分描述了通用的框架，损失函数，并与机械上的弹簧系统进行了类比。本节的思想在第3部分中进行了详述。进行了各种试验，并给出了结果，与LLE进行了对比。

## 2. Learning the Low Dimensional Mapping

The problem is to find a function that maps high dimensional input patterns to lower dimensional outputs, given neighborhood relationships between samples in input space. The graph of neighborhood relationships may come from information source that may not be available for test points, such as prior knowledge, manual labeling, etc. More precisely, given a set of input vectors $I = \{ \vec X_1,..., \vec X_P \}$, where $\vec X_i ∈ R^D$,∀i = 1,...,n, find a parametric function $G_W: R^D → R^d$ with d≪D,such that it has the following properties:

问题是要找到一个函数，在给定输入空间中样本间的相邻关系后，将高维输入模式映射到低维输出。相邻关系的图来自的信息源，对于测试点是并不可用的，比如先验知识，手动标注，等。更精确的，给定一个输入向量集$I = \{ \vec X_1,..., \vec X_P \}$，其中$\vec X_i ∈ R^D$,∀i = 1,...,n，找到一个参数化的函数$G_W: R^D → R^d$，其中d≪D，映射具有以下性质：

- Simple distance measures in the output space (such as euclidean distance) should approximate the neighborhood relationships in the input space. 输出空间中的简单距离度量（如欧几里得距离），应当近似输入空间的相邻关系。

- The mapping should not be constrained to implementing simple distance measures in the input space and should be able to learn invariances to complex transformations. 这个映射不应当局限在，实现输入空间中的简单距离度量，应当能够学习对复杂变换的不变性。

- It should be faithful even for samples whose neighborhood relationships are unknown. 即使对于相邻关系不明的样本，也应当忠实的反映。

### 2.1. The Contrastive Loss Function

Consider the set I of high dimensional training vectors $\vec X_i$. Assume that for each $\vec X_i ∈ I$ there is a set $S_{\vec X_i}$ of training vectors that are deemed similar to $X_i$. This set can be computed by some prior knowledge - invariance to distortions or temporal proximity, for instance - which does not depend on a simple distance. A meaningful mapping from high to low dimensional space maps similar input vectors to nearby points on the output manifold and dissimilar vectors to distant points. A new loss function whose minimization can produce such a function is now introduced. Unlike conventional learning systems where the loss function is a sum over samples, the loss function here runs over pairs of samples. Let $\vec X_1, \vec X_2 ∈ I$ be a pair of input vectors shown to the system. Let Y be a binary label assigned to this pair. Y = 0 if $\vec X_1$ and $\vec X_2$ are deemd similar, and Y = 1 if they are deemed dissimilar. Define the parameterized distance function to be learned $D_W$ between $\vec X_1, \vec X_2$ as the euclidean distance between the outputs of $G_W$. That is,

考虑高维训练向量$\vec X_i$的集合I。假设对每个$\vec X_i ∈ I$，都有一个训练向量的集合$S_{\vec X_i}$，这个集合里的元素是类似于$X_i$的。这个集合可以通过一些先验知识计算得到的，不依赖于一个简单的距离，比如，可以对变形或时间上的接近是不变的。从高到低维空间的有意义的映射，将类似的输入向量映射到在输出流形上相近的点，不相似的向量映射到较远的点。现在提出了一种新的损失函数，其值的最小化，可以生成这样一个函数。传统的学习系统其损失函数是在很多样本上进行求和，与之不同的是，这里的损失函数是对成对的样本进行计算。如果$\vec X_1$和$\vec X_2$是类似的，那么Y=0；如果是不相似的，则Y=1。我们将要学习的$\vec X_1, \vec X_2$之间的距离$D_W$，即参数化的距离函数，为$G_W$的输出之间的欧式距离。即：

$$D_W (\vec X_1, \vec X_2) = ||G_W (\vec X_1) − G_W (\vec X_2)||_2$$(1)

To shorten notation, $D_W (\vec X_1, \vec X_2)$ is written $D_W$. Then the loss function in its most general form is 为简化表示，$D_W (\vec X_1, \vec X_2)$写为$D_W$，那么损失函数的最一般形式为

$$L(w) = \sum_{i=1}^P L(W,(Y, \vec X_1, \vec X_2)^i)$$(2)

$$L(W, (Y, \vec X_1, \vec X_2)^i) = (1-Y)L_S(D_W^i)+YL_D(D_W^i)$$(3)

where $(Y, \vec X_1, \vec X_2)^i$ is the i-th labeled sample pair, $L_S$ is the partial loss function for a pair of similar points, $L_D$ the partial loss function for a pair of dissimilar points, and P the number of training pairs (which may be as large as the square of the number of samples).

其中$(Y, \vec X_1, \vec X_2)^i$是第i个标注的样本对，$L_S$是一对相似点的部分损失函数，$L_D$是一对不相似点的部分损失函数，P是训练对的数量（可能与样本数量的平方一样大）。

$L_S$ and $L_D$ must be designed such that minimizing L with respect to W would result in low values of $D_W$ for similar pairs and high values of $D_W$ for dissimilar pairs. $L_S$和$L_D$的设计，需要满足，变化W使L最小化，会使得对于相似的点对，$D_W$较低，对于不相似的点对，$D_W$较高。

The exact loss function is 准确的损失函数是：

$$L(W, Y, \vec X_1, \vec X_2) = (1-Y)\frac{1}{2}(D_W)^2 + Y\frac{1}{2}\{max(0,m-D_W)\}^2$$(4)

where m > 0 is a margin. The margin defines a radius around $G_W (\vec X)$. Dissimilar pairs contribute to the loss function only if their distance is within this radius (See figure 1). The contrastive term involving dissimilar pairs, $L_D$, is crucial. Simply minimizing $D_W (\vec X_1, \vec X_2)$ over the set of all similar pairs will usually lead to a collapsed solution, since $D_W$ and the loss L could then be made zero by setting $G_W$ to a constant. Most energy-based models require the use of an explicit contrastive term in the loss function.

其中m>0是预留的余地。这个边缘定义了$G_W (\vec X)$周围的一个半径。不相似的对，只有在其距离小于半径时，才会对损失函数有贡献。对比项涉及不相似的对，$L_D$，是非常关键的。只在所有相似对的集合上最小化$D_W (\vec X_1, \vec X_2)$，通常都会得到一个坍缩解，因为$D_W$和损失L会通过设$G_W$为一个常数，会置为0。多数基于能量的模型需要在损失函数中使用一个显式的对比项。

### 2.2. Spring Model Analogy 弹簧模型的类比

An analogy to a particular mechanical spring system is given to provide an intuition of what is happening when the loss function is minimized. The outputs of $G_W$ can be thought of as masses attracting and repelling each other with springs. Consider the equation of a spring

我们给出一个特定的机械弹簧系统的类比，以给出在损失函数最小化时发生什么的直觉认识。$G_W$的输出可以认为是，物质在通过弹簧吸引和排斥彼此。考虑一个弹簧等式

$$F = −KX$$(5)

where F is the force, K is the spring constant and X is the displacement of the spring from its rest length. A spring is *attract-only* if its rest length is equal to zero. Thus any positive displacement X will result in an attractive force between its ends. A spring is said to be m-repulse-only if its rest length is equal to m. Thus two points that are connected with a m-repulse-only spring will be pushed apart if X is less than m. However this spring has a special property that if the spring is stretched by a length X > m, then no attractive force brings it back to rest length. Each point is connected to other points using these two kinds of springs. Seen in the light of the loss function, each point is connected by attract-only springs to similar points, and is connected by m-repulse-only spring to dissimilar points. See figure 2.

其中F是力，K是弹簧常数，X是弹簧从其静态长度的偏移。如果其静态长度为0，则成弹簧是*attract-only*的。因此，任何正的偏移X，都是使得两个端点是一种吸引的力。如果弹簧的静态长度为m，则称弹簧为m-repulse-only。因此两个点用m-repulse-only弹簧连接，如果X值小于m，则会被推开。但是这种弹簧有一种特殊的性质，即如果弹簧的伸展长度X>m，则吸引力不会将其带回到静态长度了。每个点都用这两种弹簧与其他点相连。根据损失函数，每个点与相似点之间的连接，是用attract-only弹簧的，与不相似的点，则是通过m-repulse-only弹簧连接的。见图2。

Consider the loss function $L_S(W,\vec X_1, \vec X_2)$ associated with similar pairs. 考虑类似的点对相关的损失函数。

$$L_S(W,\vec X_1, \vec X_2) = \frac{1}{2}(D_W)^2$$(6)

The loss function L is minimized using the stochastic gradient descent algorithm. The gradient of $L_S$ is 损失函数L使用随机梯度下降算法进行最小化。$L_S$的梯度是：

$$\frac {∂L_S}{∂W} = D_w \frac {D_w}{W}$$(7)

Comparing equations 5 and 7, it is clear that the gradient $\frac {∂L_S}{∂W}$ of $L_S$ gives the attractive force between the two points. $\frac {D_w}{W}$ defines the spring constant K of the spring and $D_W$, which is the distance between the two points, gives the perturbation X of the spring from its rest length. Clearly, even a small value of $D_W$ will generate a gradient (force) to decrease $D_W$. Thus the similar loss function corresponds to the attract-only spring (figure 2).

比较式5和7，很清楚$L_S$的梯度$\frac {∂L_S}{∂W}$给出了两个点之间的吸引力。$\frac {D_w}{W}$定义了弹簧常数K，$D_W$是两个点之间的距离，给出弹簧从其静态长度的扰动X。很清楚，即使$D_W$的一个很小的值，也会生成一个梯度（力）来降低$D_W$。因此，类似的损失函数对应着attract-only弹簧。

Now consider the partial loss function $L_D$. 现在考虑部分损失函数$L_D$。

$$L_D(W,\vec X_1, \vec X_2) = \frac{1}{2}\{max(0,m-D_W)\}^2$$(8)

When $D_W > m, \frac{∂L_D}{∂W} = 0$. Thus there is no gradient (force) on the two points that are dissimilar and are at a distance $D_W > m$. If $D_W < m$ then 当$D_W > m, \frac{∂LD}{∂W} = 0$。所以如果两个点之间不相似，并且有一定距离$D_W > m$，那么两个点之间就没有梯度（力）。如果$D_W < m$，那么

$$\frac{∂L_D}{∂W} = -m(m-D_W)\frac{∂D_W}{∂W}$$(9)

Again, comparing equations 5 and 9 it is clear that the dissimilar loss function $L_D$ corresponds to the m-repulse-only spring; its gradient gives the force of the spring, $\frac{∂D_W}{∂W}$ gives the spring constant K and ($m − D_W$) gives the perturbation X. The negative sign denotes the fact that the force is repulsive only. Clearly the force is maximum when $D_W$ = 0 and absent when $D_W$ = m. See figure 2.

再次比较式5和9，很清楚，不相似的损失函数$L_D$对应着m-repulse-only弹簧；其梯度给出了弹簧的力，$\frac{∂D_W}{∂W}$给出了弹簧常数K，($m − D_W$)给出了扰动X。负号表示，力是斥力。很明显，当$D_W$ = 0时，力是最大的，当$D_W$ = m时，力就不存在了。见图2。

Here, especially in the case of $L_S$, one might think that simply making $D_W$ = 0 for all attract-only springs would put the system in equilibrium. Consider, however, figure 2e. Suppose b1 is connected to b2 and b3 with attract-only springs. Then decreasing $D_W$ between b1 and b2 will increase $D_W$ between b1 and b3. Thus by minimizing the global loss function L over all springs, one would ultimately drive the system to its equilibrium state.

这里，尤其是在$L_S$的情况下，一个人可能认为，只要对所有attract-only的弹簧使$D_W$ = 0，就会使整个系统平衡下来。但考虑图2e的情况。假设b1与b2、b3的连接是attract-only弹簧。降低b1和b2之间的$D_W$，会增加b1和b3之间的$D_W$。因此，通过对在所有弹簧上的全局损失函数L进行最小化，就可以推动这个系统到其平衡状态。

### 2.3. The Algorithm

The algorithm first generates the training set, then trains the machine. 算法首先生成训练集，然后训练这个机器。

Step 1: For each input sample $\vec X_i$, do the following:

(a) Using prior knowledge find the set of samples $S_{\vec X_i} = \{ \vec X_j \}_{j=1}^p$, such that $\vec X_j$ is deemed similar to $\vec X_i$.

(b) Pair the sample $\vec X_i$ with all the other training samples and label the pairs so that:

$Y_{ij}=0$, if $\vec X_j ∈ S_{\vec X_i}$, and $Y_{ij}=1$ otherwise.

Combine all the pairs to form the labeled training set.

Step 2: Repeat until convergence:

(a) For each pair ($\vec X_i, \vec X_j$) in the training set, do

i. If $Y_{ij}=0$, then update W to decrease

$D_W = ||G_W(\vec X_i) - G_W(\vec X_j)||_2$

ii. If $Y_{ij}=1$, then update W to increase

$D_W = ||G_W(\vec X_i) - G_W(\vec X_j)||_2$

This increase and decrease of euclidean distances in the output space is done by minimizing the above loss function.

## 3. Experiments

The experiments presented in this section demonstrate the invariances afforded by our approach and also clarify the limitations of techniques such as LLE. First we give details of the parameterized machine GW that learns the mapping function.

本节中的试验，证明了我们方法提供的不变性，也澄清了LLE等技术的局限。首先我们给出参数化机器GW的细节，GW学习了这个映射函数。

### 3.1. Training Architecture

The learning architecture is similar to the one used in [4] and [5]. Called a siamese architecture, it consists of two copies of the function GW which share the same set of parameters W, and a cost module. A loss module whose input is the output of this architecture is placed on top of it. The input to the entire system is a pair of images ($\vec X_1, \vec X_2$) and a label Y. The images are passed through the functions, yielding two outputs G($\vec X_1$) and G($\vec X_2$). The cost module then generates the distance $D_W (G_W (\vec X_1), G_W (\vec X_2))$. The loss function combines $D_W$ with label Y to produce the scalar loss $L_S$ or $L_D$, depending on the label Y. The parameter W is updated using stochastic gradient. The gradients can be computed by back-propagation through the loss, the cost, and the two instances of $G_W$. The total gradient is the sum of the contributions from the two instances.

学习框架与[4,5]中使用的类似。称为siamese架构，由函数GW的两个复制组成，共享相同的参数W集，和一个代价函数模块。损失函数的输入是这个架构的输出。整个系统的输入是一对图像($\vec X_1, \vec X_2$)，和一个标签Y。图像送入函数，得到输出G($\vec X_1$)和G($\vec X_2$)。代价模块然后生成距离$D_W (G_W (\vec X_1), G_W (\vec X_2))$。损失函数将$D_W$和标签Y结合到一起，以生成标量损失$L_S$或$L_D$，这要依赖于标签Y。参数W使用随机梯度下降进行更新。梯度可以用反向传播进行计算。总共的梯度是两个实例的共同贡献。

The experiments involving airplane images from the NORB dataset [10] use a 2-layer fully connected neural network as $G_W$. The number of hidden and output units used was 20 and 3 respectively. Experiments on the MNIST dataset[9] used a convolutional network as $G_W$ (figure3). Convolutional networks are trainable, non-linear learning machines that operate at pixel level and learn low-level features and high-level representations in an integrated manner. They are trained end-to-end to map images to outputs. Because of a structure of shared weights and multiple layers, they can learn optimal shift-invariant local feature detectors while maintaining invariance to geometric distortions of the input image.

试验采用NORB数据集的飞机图像，使用和$G_W$类似的2层全连接神经网络。隐藏单元和输出单元的数量，分别是20和3。在MNIST数据集上的试验，使用的神经网络与$G_W$类似，如图3。卷积神经网络是可以训练的、非线性学习机器，在像素层进行运算，可以以一种集成的方式学习低层的特征和高层的表示。他们进行端到端的训练，以将图像映射到输出。由于有共享的权重和多层的结构，它们可以学习到最优的偏移不变的局部特征检测器，同时保持对输入图像的几何形变的不变性。

The layers of the convolutional network comprise a convolutional layer C1 with 15 feature maps, a subsampling layer S2, a second convolutional layer C3 with 30 feature maps, and fully connected layer F3 with 2 units. The sizes of the kernels for the C1 and C3 were 6x6 and 9x9 respectively.

卷积网络的层包含，C1卷积层有15个特征图，下采样层S2，第二个卷积层C3有30个特征图，全连接层F3有2个单元。C1和C3的核的大小分别为6x6和9x9。

### 3.2. Learned Mapping of MNIST samples

The first experiment is designed to establish the basic functionality of the DrLIM approach. The neighborhood graph is generated with euclidean distances and no prior knowledge.

第一个试验的设计是为了确定DrLIM方法的基本功能。相邻图是使用欧几里得距离生成的，没有使用先验信息。

The training set is built from 3000 images of the handwritten digit 4 and 3000 images of the handwritten digit 9 chosen randomly from the MNIST dataset [9]. Approximately 1000 images of each digit comprised the test set. These images were shuffled, paired, and labeled according to a simple euclidean distance measure: each sample $\vec X_i$ was paired with its 5 nearest neighbors, producing the set $S_{\vec X_i}$. All other possible pairs were labeled dissimilar, producing 30,000 similar pairs and on the order of 18 million dissimilar pairs.

训练集是用3000个手写数字4和3000个手写数字9构成的，这些图像是随机从MNIST数据集中构成的。测试集由每个数字的大约1000个图像构成。这些图像根据简单的欧式距离度量，进行了混洗，配对和标注：每个样本$\vec X_i$与5个最近的近邻进行配对，生成集合$S_{\vec X_i}$。所有其他可能的对都标注为不相似，一共生成了30,000类似的对，和18 million不相似的对。

The mapping of the test set to a 2D manifold is shown in figure 4. The lighter-colored blue dots are 9’s and the darker-colored red dots are 4’s. Several input test samples are shown next to their manifold positions. The 4’s and 9’s are in two somewhat overlapping regions, with an overall organization that is primarily determined by the slant angle of the samples. The samples are spread rather uniformly in the populated region.

测试集到一个2D流形的映射，如图4所示。浅蓝色的点是9，深红色的点是4。几个输入测试样本在其流形未知上进行了显示。4和9是两个有一些重叠的区域，整体的组织是由样本的倾斜角度来决定的。样本在其区域中，还是分布比较均匀的。

### 3.3. Learning a Shift-Invariant Mapping of MNIST samples

In this experiment, the DrLIM approach is evaluated using 2 categories of MNIST, distorted by adding samples that have been horizontally translated. The objective is to learn a 2D mapping that is invariant to horizontal translations.

在本试验中，使用MNIST中的两类对DrLIM方法进行评估，这些样本中加入了水平平移。目标是学习对水平平移不变的2D映射。

In the distorted set, 3000 images of 4’s and 3000 images of 9’s are horizontally translated by -6, -3, 3, and 6 pixels and combined with the originals, producing a total of 30,000 samples. The 2000 samples in the test set were distorted in the same way.

在变形的集合里，3000个4的图像和3000个9的图像进行了-6, -3, 3和6个像素的水平平移，与原始图像放到一起，生成了总计30000样本。测试集中的2000个样本也进行了同样的变形。

First the system was trained using pairs from a euclidean distance neighborhood graph (5 nearest neighbors per sample), as in experiment 1. The large distances between translated samples creates a disjoint neighborhood relationship graph and the resulting mapping is disjoint as well. The output points are clustered according to the translated position of the input sample (figure 5). Within each cluster, however, the samples are well organized and evenly distributed.

首先系统的训练使用的图像对，是欧式距离临近的相邻图（每个样本5个最近距离），如试验中一样。平移样本产生了很大的距离，这得到了一种不相交的相邻关系图，得到的映射也不是不相交的。输出的点根据输入样本的平移位置进行了聚类（图5）。但是，在每个聚类中，样本都是均匀分布的。

For comparison, the LLE algorithm was used to map the distorted MNIST using the same euclidean distance neighborhood graph. The result was a degenerate embedding in which differently registered samples were completely separated (figure 6). Although there is sporadic local organization, there is no global coherence in the embedding.

为进行对比，使用了LLE算法来映射形变的MNIST，使用的欧式距离相邻图是一样的。得到的结果是一个降质的嵌入，不同对齐的图像完全分开了（图6）。虽然有局部的组织，但是这个嵌入中没有全局的连贯性。

In order to make the mapping function invariant to translation, the euclidean nearest neighbors were supplemented with pairs created using prior knowledge. Each sample was paired with (a) its 5 nearest neighbors, (b) its 4 translations, and (c) the 4 translations of each of its 5 nearest neighbors. Additionally, each of the sample’s 4 translations was paired with (d) all the above nearest neighbors and translated samples. All other possible pairs are labeled as dissimilar.

为使这个映射函数对平移不变，欧式最近相邻还加入了一些使用先验知识创建的图像对。每个样本与如下图像进行配对：9a
其最近的5个近邻，(b)其4个平移，和(c)其5个最近邻每个图像的4个平移。另外，每个样本的4个平移与(d)所有上述近邻和平移的样本进行了配对。所有其他可能的对都被标注为不相似。

The mapping of the test set samples is shown in figure 7. The lighter-colored blue dots are 4’s and the darker-colored red dots are 9’s. As desired, there is no organization on the basis of translation; in fact, translated versions of a given character are all tighlty packed in small regions on the manifold.

测试集样本的映射如图7所示。浅蓝色的点是4，深红色的点是9。和预期一样，没有基于平移的组织；实际上，平移版的给定数字紧密排列在一起，构成了流形中很小的区域。

### 3.4. Mapping Learned with Temporal Neighborhoods and Lighting Invariance

The final experiment demonstrates dimensionality reduction on a set of images of a single object. The object is an airplane from the NORB [10] dataset with uniform backgrounds. There are a total of 972 images of the airplane under various poses around the viewing half-sphere, and under various illuminations. The views have 18 azimuths (every 20 degrees around the circle), 9 elevations (from 30 to 70 degrees every 5 degrees), and 6 lighting conditions (4 lights in various on-off combinations). The objective is to learn a globally coherent mapping to a 3D manifold that is invariant to lighting conditions. A pattern based on temporal continuity of the camera was used to construct a neighborhood graph; images are similar if they were taken from contiguous elevation or azimuth regardless of lighting. Images may be neighbors even if they are very distant in terms of Eucliden distance in pixel space, due to different lighting.

最后一个试验展示了在单目标图像集合上的降维效果。目标是NORB数据集上的飞机图像，有着单一的背景。总共有972幅飞机的图像，包含各种姿态，各种光照。视角包含18个方位角（圆周上每隔20度），9种高程（从30度到70度，每隔5度），6种光照条件（4个光源，有各种开关组合）。其目标是学习一个全局连贯的到3D流形中的映射，对各种光照条件都是不变的。使用了一种基于时域连续性的模式，来构建相邻图；如果图像是从连续的高程或方位角采的，不管是什么光照，那么都认为是相似的。由于不同的光照，以像素空间的欧式距离来说，可能很远，但这些图像也可能是相邻的。

The dataset was split into 660 training images and a 312 test images. The result of training on all 10989 similar pairs and 206481 dissimilar pairs is a 3-dimensional manifold in the shape of a cylinder (see figure 8). The circumference of the cylinder corresponds to change in azimuth in input space, while the height of the cylinder corresponds to elevation in input space. The mapping is completely invariant to lighting. This outcome is quite remarkable. Using only local neighborhood relationships, the learned manifold corresponds globally to the positions of the camera as it produced the dataset.

数据集分成660个训练图像，和312个测试图像。训练集图像可以形成10989对类似的和206481对不类似的图像，将其训练映射到一个3维流形，形状是一个圆柱（见图8）。圆柱形的圆周对应着输入空间方位角的变化，而圆柱形的高度对应着输入空间的高程。这个映射对光照完全不变。其输出是非常令人印象深刻的。只使用局部邻域关系，学习到的流形对应着相机的位置，因为其产生了这个数据集。

Viewing the weights of the network helps explain how the mapping learned illumination invariance (see figure 9). The concentric rings match edges on the airplanes to a particular azimuth and elevation, and the rest of the weights are close to 0. The dark edges and shadow of the wings, for example, are relatively consistent regardless of lighting.

观察网络的权重，可以帮助解释这个映射怎样学习到光照不变性（见图9）。同心环将飞机的边缘与特定的方位角和高程进行了匹配，其余的权重接近是0。比如，暗的边缘和机翼的暗影，相对来说对于光照来说是一致变化较少的。

For comparison, the same neighborhood relationships defined by the prior knowledge in this experiment were used to create an embedding using LLE. Although arbitrary neighborhoods can be used in the LLE algorithm, the algorithm computes linear reconstruction weights to embed the samples, which severely limits the desired effect of using distant neighbors. The embedding produced by LLE is shown (see figure 10). Clearly, the 3D embedding is not invariant to lighting, and the organization of azimuth and elevation does not reflect the real topology neighborhood graph.

为进行比较，用本试验中先验知识定义的相同的相邻关系，也用于使用LLE生成一个嵌入。虽然任意相邻关系可以用于LLE算法，算法计算线性重建权重来嵌入样本，这严重限制了使用距离相邻的期望效果。LLE产生的嵌入如图10所示。很明显，这个3D嵌入对光照不是不变的，按照方位角和高程的组织也没有反映相邻图的真实拓扑。

## 4. Discussion and Future Work

The experiments presented here demonstrate that, unless prior knowledge is used to create invariance, variabilities such as lighting and registration can dominate and distort the outcome of dimensionality reduction. The proposed approach, DrLIM, offers a solution: it is able to learn an invariant mapping to a low dimensional manifold using prior knowledge. The complexity of the invariances that can be learned are only limited by the power of the parameterized function GW. The function maps inputs that evenly cover a manifold, as can be seen by the experimental results. It also faithfully maps new, unseen samples to meaningful locations on the manifold.

这里的试验证明了，除非使用先验知识来生成不变性，光照和对齐这样的变化可以对降维的输出造成很严重的影响。我们提出的方法DrLIM给出了一个解决方案：可以学习一个不变的映射，使用先验知识将输入映射到低维流形。可以学习到的不变性的复杂度，只受参数化函数GW的能力的限制。函数将输入映射到一个流形，并且均匀分布，这由试验结果可以看到。还可以将新的，未曾加过的样本，也忠诚的映射到流形上有意义的位置。

The strength of DrLIM lies in the contrastive loss function. By using a separate loss function for similar and dissimilar pairs, the system avoids collapse to a constant function and maintains an equilibrium in output space, much as a mechanical system of interconnected springs does.

DrLIM的力量来源于对比损失函数。通过对类似的对和不相似的对，使用不同的损失函数，系统不会坍缩到一个常数函数，并在输出空间中保持一个均衡，与相互连接的弹簧机械系统非常相似。

The experiments with LLE show that LLE is most useful where the input samples are locally very similar and well-registered. If this is not the case, then LLE may give degenerate results. Although it is possible to run LLE with arbitrary neighborhood relationships, the linear reconstruction of the samples negates the effect of very distant neighbors. Other dimensionality reduction methods have avoided this limitation, but none produces a function that can accept new samples without recomputation or prior knowledge.

LLE的试验表明，LLE在输入样本非常相似而且很好的对齐时很好用。如果不是这个情况的话，那么LLE就会给出降质的结果。虽然可以在任意相邻下运行LLE，样本的线性重建会否定非常远的相邻的效果。其他降维方法避免过这种限制，但都没有产生一个函数，可以接受新的样本，而不需要重新计算，或不需要先验知识。

Creating a dimensionality reduction mapping using prior knowledge has other uses. Given the success of the NORB experiment, in which the positions of the camera were learned from prior knowledge of the temporal connections between images, it may be feasible to learn a robot’s position and heading from image sequences.

使用先验知识创建一个降维映射有一些其他用处。在NORB试验的成功中，相机位置从图像间的时域关系的先验知识中可以学到，从图像序列中可能学习到一个机器人的位置和朝向。