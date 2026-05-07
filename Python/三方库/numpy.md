# NumPy 速查手册

## 1. 创建数组

```python
import numpy as np

a = np.array([1, 2, 3])                # 从列表创建
b = np.array([[1,2],[3,4]])             # 2D 数组

np.zeros((2, 3))                        # 全 0，shape=(2,3)
np.ones((2, 3))                         # 全 1
np.full((2, 3), 7)                      # 全填充 7
np.empty((2, 3))                        # 未初始化（快，值随机）
np.eye(3)                               # 3×3 单位矩阵

np.arange(0, 10, 2)                     # [0 2 4 6 8]  类似 range
np.linspace(0, 1, 5)                    # [0. 0.25 0.5 0.75 1.]  等间距

np.random.rand(2, 3)                    # 均匀分布 [0,1)
np.random.randn(2, 3)                   # 标准正态分布
np.random.randint(0, 10, (2, 3))        # 随机整数
```

---

## 2. 属性

```python
a.shape      # 形状，如 (2, 3)
a.ndim       # 维度数
a.size       # 元素总数
a.dtype      # 数据类型
a.itemsize   # 每个元素字节数
```

---

## 3. 索引与切片

```python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])

a[0, 1]          # 第0行第1列 → 2
a[0]             # 第0行 → [1,2,3]
a[:, 1]          # 第1列 → [2,5,8]
a[0:2, 1:]       # 前2行、第1列起 → [[2,3],[5,6]]

# 布尔索引
a[a > 5]         # → [6,7,8,9]

# 花式索引
a[[0,2], [1,2]]  # → [a[0,1], a[2,2]] = [2, 9]
```

---

## 4. 变形

```python
a.reshape(3, 2)        # 变形为 3×2（不改原数组，返回视图）
a.reshape(-1)          # 展平为 1D（-1 自动推导）
a.flatten()            # 展平（返回副本）
a.ravel()              # 展平（返回视图）
a.T                    # 转置
a.transpose(1, 0)      # 等价于 .T（可指定轴顺序）

np.expand_dims(a, 0)   # 增加维度 (3,) → (1,3)
np.squeeze(a)          # 去掉长度为1的维度

np.concatenate([a, b], axis=0)  # 沿 axis 拼接
np.stack([a, b], axis=0)       # 沿新轴堆叠
np.split(a, 3, axis=0)         # 沿 axis 切分为 3 份
```

---

## 5. 数学运算（逐元素）

```python
a + b    # np.add(a, b)
a - b    # np.subtract(a, b)
a * b    # np.multiply(a, b)      ← 逐元素乘
a / b    # np.divide(a, b)
a ** 2   # np.power(a, 2)
np.sqrt(a)
np.exp(a)
np.log(a)       # 自然对数
np.abs(a)
np.sin(a)       # cos, tan 同理
```

---

## 6. 矩阵运算

```python
a @ b                   # 矩阵乘法（推荐）
np.dot(a, b)            # 同上
np.matmul(a, b)         # 同上

np.linalg.inv(a)        # 逆矩阵
np.linalg.det(a)        # 行列式
np.linalg.eig(a)        # 特征值 & 特征向量
np.linalg.norm(a)       # 范数（默认 L2）
np.linalg.solve(A, b)   # 解 Ax = b
```

---

## 7. 统计

```python
a.sum()             # 所有元素求和
a.sum(axis=0)       # 按列求和
a.sum(axis=1)       # 按行求和

a.mean()            # 均值
a.std()             # 标准差
a.var()             # 方差
a.min() / a.max()
a.argmin() / a.argmax()   # 最值索引
np.median(a)              # 中位数
np.percentile(a, 75)      # 百分位数
np.cumsum(a)              # 累积和
np.cumprod(a)             # 累积积
```

---

## 8. 广播（Broadcasting）

形状不同时自动扩展对齐，规则：**从右往左**逐维比较，维度相等或其中一个为 1 即可广播。

```python
a = np.ones((3, 4))    # shape (3,4)
b = np.array([1,2,3,4])# shape (4,)  → 广播为 (3,4)
a + b                   # OK

c = np.array([[1],[2],[3]])  # shape (3,1) → 广播为 (3,4)
a + c                        # OK
```

---

## 9. 排序与搜索

```python
np.sort(a)                # 返回排序副本
a.sort()                  # 原地排序
np.argsort(a)             # 排序后的索引

np.where(a > 5)           # 满足条件的索引
np.where(a > 5, a, 0)     # 三元：满足填 a，否则填 0

np.unique(a)                        # 去重
np.unique(a, return_counts=True)    # 去重 + 计数
```

---

## 10. 复制

```python
b = a            # 引用，共享内存
b = a.view()     # 浅拷贝（共享数据，形状独立）
b = a.copy()     # 深拷贝（完全独立）
```

---

## 11. 类型转换

```python
a.astype(np.float32)     # 转换数据类型
a.astype(int)

# 常用 dtype
# np.float32 / np.float64
# np.int32   / np.int64
# np.bool_   / np.str_
```

---

## 12. 文件 I/O

```python
np.save('a.npy', a)            # 保存单个数组
a = np.load('a.npy')           # 加载

np.savez('ab.npz', a=a, b=b)  # 保存多个
data = np.load('ab.npz')      # data['a'], data['b']

np.savetxt('a.csv', a, delimiter=',')   # 存文本
a = np.loadtxt('a.csv', delimiter=',')  # 读文本
```

---

## 13. 实用技巧

```python
# 设置打印选项
np.set_printoptions(precision=3, suppress=True)

# 判断相等（浮点安全）
np.allclose(a, b, atol=1e-8)

# 向量化自定义函数
vfunc = np.vectorize(lambda x: x ** 2 + 1)
vfunc(a)

# 条件计数
np.count_nonzero(a > 5)
(a > 5).sum()

# clip 截断
np.clip(a, 0, 255)   # 限制在 [0, 255]
```
