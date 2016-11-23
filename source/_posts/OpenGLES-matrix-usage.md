---
title: 矩阵的应用
date: 2016-11-21 23:22:31
tags: OpenGL ES
categories: OpenGL ES
---
# 单位矩阵
与任何向量相乘都得到原来的向量。
$$
 \left[
 \begin{matrix}
   1 & 0 & 0 & 0 \\\\
   0 & 1 & 0 & 0 \\\\
   0 & 0 & 1 & 0 \\\\
   0 & 0 & 0 & 1
  \end{matrix}
  \right]
$$

***
# 模型矩阵

## 平移矩阵
$$
 \left[
 \begin{matrix}
   1 & 0 & 0 & X_t \\\\
   0 & 1 & 0 & Y_t \\\\
   0 & 0 & 1 & Z_t \\\\
   0 & 0 & 0 & 1
  \end{matrix}
  \right]
$$
例如：Vec(2,2,0) 沿着X轴平移3，沿着Y轴平移3，则：
$$
\left[
    \begin{matrix}
        1 & 0 & 0 & 3 \\\\
        0 & 1 & 0 & 3 \\\\
        0 & 0 & 1 & 0 \\\\
        0 & 0 & 0 & 1
    \end{matrix}
\right]
\left[
    \begin{matrix}
        2 \\\\
        2 \\\\
        0 \\\\
        1
    \end{matrix}
\right]
=
\left[
    \begin{matrix}
        5 \\\\
        5 \\\\
        0 \\\\
        1
    \end{matrix}
\right]
$$

## 缩放矩阵
$$
\begin{bmatrix}x'\\\\ y'\\\\ z'\end{bmatrix}
=
\begin{bmatrix}s_{x} & 0 & 0\\\\ 0 & s_{y} & 0\\\\ 0 & 0 & s_{z} \end{bmatrix}
\begin{bmatrix}x\\\\ y\\\\ z\end{bmatrix}
$$

## 旋转矩阵
旋转变换有一些复杂，先看在二维平面上的旋转变换：
很容易得到：

$$\begin{matrix}x'=x\cos\theta-y\sin\theta\\y'=x\sin\theta+y\sin\theta\end{matrix}$$

矩阵形式的表达更加简洁，后面大多使用这种形式：

$$\begin{bmatrix}x'\\ y'\end{bmatrix}=\begin{bmatrix}\cos\theta & -\sin\theta\\ \sin\theta & \cos\theta\end{bmatrix}\begin{bmatrix}x\\ y\end{bmatrix}$$
点绕z轴旋转：

$$\begin{bmatrix}x'\\ y'\\ z'\end{bmatrix}=\begin{bmatrix}\cos\theta & -\sin\theta & 0\\ \sin\theta & \cos\theta & 0\\ 0 & 0 & 1\end{bmatrix}\begin{bmatrix}x\\ y\\ z\end{bmatrix}$$

点绕x轴旋转：

$$\begin{bmatrix}x'\\ y'\\ z'\end{bmatrix}=\begin{bmatrix}1 & 0 & 0\\ 0 & \cos\theta & -\sin\theta\\ 0 & \sin\theta & \cos\theta\end{bmatrix}\begin{bmatrix}x\\ y\\ z\end{bmatrix}$$

点绕y轴旋转：

$$\begin{bmatrix}x'\\ y'\\ z'\end{bmatrix}=\begin{bmatrix}\cos\theta & 0 & -\sin\theta\\ 0 & 1 & 0\\ \sin\theta & 0 & \cos\theta\end{bmatrix}\begin{bmatrix}x\\ y\\ z\end{bmatrix}$$

***

# 正交投影矩阵
- 作用：正交投影矩阵可以把虚拟坐标转换回归一化设备坐标（正交投影矩阵乘以虚拟坐标）。
- 归一化设备坐标：在OpenGL里，一切物体都要映射到X、Y轴和Z轴的[-1,1]范围内，这个范围内的坐标被称为归一化设备坐标，其独立于屏幕实际的尺寸。归一化设备坐标假定坐标空间是个正方形。
- 虚拟坐标空间：为了让屏幕形状考虑进来，把宽和高中较小的一个的范围定在[-1,1]内，另外一个根据屏幕尺寸比例调整为较大的范围。

正交投影矩阵如下：
$$
 \left[
 \begin{matrix}
   \frac{2}{right-left} & 0 & 0 & -\frac{right+left}{right-left} \\\\
   0 & \frac{2}{top-bottom} & 0 & -\frac{top+bottom}{top-bottom} \\\\
   0 & 0 &  -\frac{2}{far-near} & -\frac{far+near}{far-near} \\\\
   0 & 0 & 0 & 1
  \end{matrix}
  \right]
$$
注意：使用的是左手坐标系还是右手坐标系，这两者的Z轴是相反的。
其实在一个顶点着色器的顶点位置(gl_Position)变为归一化设备坐标前，还会做透视除法（xyz都除以w）。

***
# 透视投影矩阵
透视投影矩阵最大的作用是产生正确的w值。w值可以理解为距离，w值越大，离中心点越近。

透视投影矩阵
$$
 \left[
 \begin{matrix}
   \frac{α}{width/height} & 0 & 0 & 0 \\\\
   0 & α & 0 & 0 \\\\
   0 & 0 & -\frac{far+near}{far-near} & -\frac{2\*far\*near}{far-near} \\\\
   0 & 0 & -1 & 0
  \end{matrix}
  \right]
$$
其中α是焦距:
$$α = \frac {1} {tan(FOV/2)}$$
FOV是相机的垂直视角，而不是水平视角
width:屏幕宽度
height:屏幕高度
far: 到远处平面的距离（>0 && > near）
near: 到近处平面的距离（>0）
