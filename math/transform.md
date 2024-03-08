把一个向量从一个空间变换到另一个空间后，这个向量还是从前的那个向量吗？

“任何向量都是**基向量**的线性组合”，现在以基向量 $ij$ [张成的](https://charlesliuyx.github.io/2017/10/06/【直观详解】线性代数的本质/)二维空间举例，我们把向量 $v$ 定义为 $v_x i + v_y j$ ，在这种定义下我们通常所说的 $v = \begin{bmatrix} v_x \\ v_y\end{bmatrix}$ 处于以基向量 $i = \begin{bmatrix} 1 \\ 0\end{bmatrix}$，$j = \begin{bmatrix} 0 \\ 1\end{bmatrix}$ 张成的空间中，即 $v = v_x \begin{bmatrix} 1 \\ 0\end{bmatrix} + v_y \begin{bmatrix} 0 \\ 1\end{bmatrix} = \begin{bmatrix} v_x \\ v_y\end{bmatrix}$

## 向量与矩阵的乘法
推广到一般情况，向量 $v$ 在 $i = \begin{bmatrix} i_x \\ i_y\end{bmatrix}$，$j = \begin{bmatrix} j_x \\ j_y\end{bmatrix}$ 张成的空间下的映射 $v^{\prime}$ 为 

$v^{\prime} = v_x \begin{bmatrix} i_x \\ i_y\end{bmatrix} + v_y \begin{bmatrix} j_x \\ j_y\end{bmatrix} = \begin{bmatrix} v_x i_x + v_y j_x \\ v_x i_y + v_y j_y\end{bmatrix}$

*是不是可以认为 $v$ 与 $v^{\prime}$ 是同一个向量，只是基变了*

好，书归正传，我们换一种写法

$v^{\prime} = v \begin{bmatrix} i_x \\ i_y\end{bmatrix} \begin{bmatrix} j_x \\ j_y\end{bmatrix} = v \begin{bmatrix} i_x & j_x\\i_y & j_y\end{bmatrix} = \begin{bmatrix} v_x i_x + v_y j_x \\ v_x i_y + v_y j_y\end{bmatrix}$ 

如此我们就定义了**向量与矩阵的乘法运算**

## 矩阵与矩阵的乘法
重复一遍“任何向量都是**基向量**的线性组合”，基向量也是向量, 把基向量 $ij$ 转换到基向量 $mn$ 张成的空间下

$i' = i_x m + i_y n$

$j' = j_x m + j_y n$

写成矩阵形式

$\begin{bmatrix} i'  \quad  j' \end{bmatrix} = \begin{bmatrix} i_x m + i_y n  \quad  j_x m + j_y n \end{bmatrix} = \begin{bmatrix} i_x m_x + i_y n_x & j_x m_x+ j_y n_x \\ i_x m_y + i_y n_y & j_x m_y + j_y n_y\end{bmatrix} = \begin{bmatrix} i_x & j_x\\i_y & j_y\end{bmatrix} \begin{bmatrix} m_x & n_x\\m_y & n_y\end{bmatrix} $

如此，我们又定义了**矩阵乘法**

## 矩阵加法？
https://github.com/KhronosGroup/glTF-Tutorials/blob/master/gltfTutorial/gltfTutorial_020_Skins.md