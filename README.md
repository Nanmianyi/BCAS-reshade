# BCAS-reshade
The most advanced bilateral filtering sharpening shader in Reshade, featuring multiple color adjustment capabilities with minimal performance overhead.
一种结合双边滤波分解、软限制抗过冲与 SCAA边缘解析的实时锐化变种。
目标：在增强细节的同时，最大限度抑制振铃/色度伪影/棋盘格，并提供电影级调色（可选）。


算法功能处理总览
数学与实现细节
YCoCg 颜色空间与亮度定义
MAD4 边缘强度与一致性估计
自适应双边基底与细节层
噪声门控与色度偏好保护
SCAA/EAA：方向性重构与覆盖率目标
AURA：统一软限制抗过冲与能量压缩
色度段抑伪影与方向矫正
暗部保护
电影级调色引擎（可选）
LUT 集成（可选）
调试模式

🚀优势：
本实现并非“在双边滤波上加锐化”而是把双边分解当作“亮度基底 + 细节层”，后几何一致性约束与软限制抗过冲，并对色度/噪声/暗部分通道治理，最终得到更稳健的锐化与更低伪影。

1. 与传统双边滤波的根本区别
维度	传统 BF / 双边锐化变体	本实现
处理目标	主要做边缘保留平滑；锐化多为“原图 − 平滑”再加回	先分解出 Ybase 与 detailY，在细节域内自适应增益 + 软限制，再做几何/色度治理
范围项 σ	固定或全局可调	σ_R 随局部梯度/MAD 自适应（RangeSigma × GradAdapt × MADref）
空间项	高斯/盒核	稳定常权近邻 + 可选 SpatialSigma 指数衰减，避免小核误估
中心权重	常与邻域同权	CenterWeight 明确提升中心保真，减少“被洗”的观感
锐化限幅	常见硬钳位（clip）或不加限幅	AURA 软限制（双极性 tanh）+ 可选能量压缩（幂平均），亮/暗可分（AR_L/D_Overshoot）
边缘几何	无几何对齐	SCAA/EAA：沿切线/法线虚拟重构、解析覆盖率目标，抑制锯齿与峰值拉伸
色度处理	通常与亮度同策略	色度分通道：方向一致性门控、棋盘格提示、蓝通道保护、色度抗过冲/降噪
噪声观测	少见显式门控	NoiseFloor/NoiseSuppress + 暗部保护（按局部对比自适应）
可观测性	调参黑箱	完备 Debug 模式（基底/细节/遮罩/尺度/方向等 17 项可视化）

