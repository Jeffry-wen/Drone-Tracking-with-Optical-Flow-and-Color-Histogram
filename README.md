# 🛸 Drone Tracking with Optical Flow and Color Histogram

本项目实现了一个基于 **光流（Optical Flow）** 和 **颜色直方图反向投影（Back Projection）** 的无人机目标跟踪系统，并结合多种优化方案以提升鲁棒性与实时性能。

---

## 📷 输入
- 视频文件 $X$，例如 `IMG_9045-4.MOV`
- 用户在首帧手动选取的 ROI 区域 $B_0 = (x_0, y_0, w_0, h_0)$

---

## 🔍 任务流程

给定输入图像序列  
$$
X = \{ I_0, I_1, \dots, I_T \},
$$  
系统首先在初始帧 $I_0$：

1. 转为灰度图 $I_0^{\text{gray}}$
2. 在 ROI 内提取角点特征 $P_0$
3. 在 ROI 内计算 HSV 颜色直方图 $H_{\text{ROI}}$

随后，对于每一帧 $I_t$ 执行以下步骤：

### 1️⃣ 光流追踪

基于稀疏光流（Lucas–Kanade 算法）计算特征点位置：
$$
P_t, \text{status}, \text{err} = \text{calcOpticalFlowPyrLK}\big(I_{t-1}^{\text{gray}}, I_t^{\text{gray}}, P_{t-1}\big).
$$

筛选有效点集合：
$$
P_t^{*} = \{\, p \in P_t \mid \text{status}(p) = 1 \,\}.
$$

计算有效特征点的中心：
$$
C_t = (c_x, c_y) = \frac{1}{|P_t^{*}|} \sum_{p \in P_t^{*}} p.
$$

并剔除距离中心过远的异常点。

---

### 2️⃣ 颜色直方图反向投影

对当前帧的 HSV 图像 $I_t^{\text{HSV}}$ 进行反向投影：
$$
BP_t = \text{calcBackProject}\big(I_t^{\text{HSV}}, H_{\text{ROI}}\big).
$$

根据 $BP_t$ 限制光流计算区域，并在特征点丢失时使用 MeanShift 进行 ROI 估计：
$$
(x_t, y_t, w_t, h_t) = \text{meanShift}(BP_t, \text{search window}).
$$

---

### 3️⃣ 丢失恢复与特征更新

- 若特征点数量过少或完全丢失：
  - 扩大搜索窗口（乘以系数 $2.0$）
  - 在新窗口内重新检测角点特征
- 前 $10$ 帧内，额外使用 MeanShift 颜色跟踪增强稳定性

---

## ⚙️ 主要优化方案

- ✅ **增加初始特征点数量：** `maxCorners = 300`  
- ✅ **光流结合反向投影：** 避免背景干扰  
- ✅ **剔除偏离中心的特征点：** 稳定 ROI 边界  
- ✅ **特征点丢失时自动扩大搜索范围恢复**  
- ✅ **前 10 帧双保险：光流 + MeanShift 同时跟踪**

---

## 📐 关键公式

- **光流计算：**
  $$
  P_t = \text{calcOpticalFlowPyrLK}\big(I_{t-1}^{\text{gray}}, I_t^{\text{gray}}, P_{t-1}\big)
  $$

- **颜色直方图：**
  $$
  H_{\text{ROI}} = \text{calcHist}\big(I_{\text{ROI}}^{\text{HSV}}\big)
  $$

- **反向投影：**
  $$
  BP_t = \text{calcBackProject}\big(I_t^{\text{HSV}}, H_{\text{ROI}}\big)
  $$

- **ROI 中心点：**
  $$
  C_t = \frac{1}{|P_t^{*}|} \sum_{p \in P_t^{*}} p
  $$

---

## 📦 依赖

- Python 3.x  
- OpenCV (`cv2`)  
- NumPy  

安装命令：
```bash
pip install opencv-python numpy
