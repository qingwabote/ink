# 向量 $\mathbf{v}$ 绕轴 $\mathbf{u}$ 旋转弧度 $θ$
如果把 $\mathbf{v}$ 分解为平行和垂直于 $\mathbf{u}$ 的分向量 $\mathbf{v} = \mathbf{v_∥} + \mathbf{v_⊥}$ 那么

$\mathbf{v^′_⊥} = \cos(θ)\mathbf{v_⊥} + \sin(θ)(\mathbf{u} × \mathbf{v_⊥})$

标量乘法对叉乘的结合律:  
$\mathbf{v^′_⊥} = \cos(θ)\mathbf{v_⊥} + (\sin(θ)\mathbf{u}) × \mathbf{v_⊥}$

然后分配律？：  
$\mathbf{v^′_⊥} = (\cos(θ) + \sin(θ)\mathbf{u}) × \mathbf{v_⊥}$  
*[可以查到**标量乘法对向量加法的分配律**与**叉乘对向量加法的分配律**，但是混合在一起？](https://jiaxianhua.github.io/math/2015/06/09/vector-operation)*

显然，想要简化它，得换条路。

回到起点：  
$\mathbf{v^′_⊥} = \cos(θ)\mathbf{v_⊥} + \sin(θ)(\mathbf{u} × \mathbf{v_⊥})$

*[四元数与三维旋转](https://github.com/Krasjet/quaternion)*

# 万向节死锁（Gimbal lock）
```ts
fromXRotation(radian: number): Mat4 {
    const s = Math.sin(radian);
    const c = Math.cos(radian);

    return [
        1, 0, 0, 0,
        0, c, s, 0,
        0, -s, c, 0,
        0, 0, 0, 1
    ];
}

fromYRotation(radian: number): Mat4 {
    const s = Math.sin(radian);
    const c = Math.cos(radian);

    return [
        c, 0, -s, 0,
        0, 1, 0, 0,
        s, 0, c, 0,
        0, 0, 0, 1
    ];
}

fromZRotation(radian: number): Mat4 {
    const s = Math.sin(radian);
    const c = Math.cos(radian);

    return [
        c, s, 0, 0,
        -s, c, 0, 0,
        0, 0, 1, 0,
        0, 0, 0, 1
    ];
}
```
**我把坐标变换定义为基向量的变换，把基向量的值排列起来就是矩阵，而基向量也是向量，改变基向量的基向量就是矩阵乘法**。与缩放不同，缩放基向量 i 时，j 与 k 并不受影响，所以你可以分别给定三个基向量的缩放即可。围绕 i 旋转时，j 和 k 发生了变化，然后围绕 j 旋转时 k 的值就不能只由自身的变换给出。

[万向节锁，有时这是个大麻烦，但有时就是你想要的结果](https://enjoyphysics.cn/Article508)



# 四元数