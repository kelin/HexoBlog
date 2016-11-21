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

# 平移矩阵
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

***

# 正交投影矩阵
作用：正交投影矩阵可以把虚拟坐标转换回归一化设备坐标（正交投影矩阵乘以虚拟坐标）。
归一化设备坐标：在OpenGL里，一切物体都要映射到X、Y轴和Z轴的[-1,1]范围内，这个范围内的坐标被称为归一化设备坐标，其独立于屏幕实际的尺寸。归一化设备坐标假定坐标空间是个正方形。
虚拟坐标空间：为了让屏幕形状考虑进来，把宽和高中较小的一个的范围定在[-1,1]内，另外一个根据屏幕尺寸比例调整为较大的范围。
$$
 \left[
 \begin{matrix}
   \frac{2}{right-left} & 0 & 0 & -\frac{right+left}{right-left} \\\\
   0 & \frac{2}{top-bottom} & 0 & -\frac{top+bottom}{top-bottom} \\\\
   0 & 0 &  \frac{2}{far-near} & -\frac{far+near}{far-near} \\\\
   0 & 0 & 0 & 1
  \end{matrix}
  \right]
$$
注意：使用的是左手坐标系还是右手坐标系，这两者的Z轴是相反的。


