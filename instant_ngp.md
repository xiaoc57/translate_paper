# Instant Neural Graphics Primitives with a Multiresolution Hash Encoding


## E.2 占用网格（Occupancy Grids）
为了在空白空间中跳过光线步进，我们维护一个K个级联的多尺度占用网格，在合成NeRF场景中K=1（单网格），在大型真实世界场景中K属于1到5（最多5个网格，取决于场景的大小）。每一个网格的分辨率都是128的三次方。跨越以(0.5, 0.5, 0.5)为中心的几何增长域[](这里还有个值)。  
每一个网格单元存储了占用以一个单独的位。这个单元被以莫顿编码（z曲线）的顺序放置为了方便数字微分分析程序（DDA）以内存一致性方式遍历。在光线步进期间，从前一个部分根据步长使当前采样点即将到达的时候，如果网格的位是低位，则这个采样点将被跳过。  
在K个网格中哪一个被索引决定于采样点的坐标x和步长delta t：在所有能够覆盖x的网格中，最应该被选择的是单元的变长大于delta t的那个。（这里翻译的好像不对，查询哪一个𝐾网格由样本位置x和步长Δ𝑡共同决定：在覆盖x的网格中，查询单元边长大于Δ𝑡的最细的网格。）  
*更新占用网格（Updating the occupancy grids）* 为了在训练中持续更新占用网格，我们维护了第二个系列的网格，它们具有相同的层级，不同的是它们存储了全精度浮点密度值而不是单个位。  
我们更新网格在每16次迭代后通过实行以下步骤：
(1) 我们衰减密度值以0.95的因子在每一个单元格中。
(2) 随机采样M个候选单元，将它们的值设为以下两个的最大值，它们的当前值和NeRF的密度组件计算出的单元中随机一点的密度值。
(3) 更新占用位通过每个单元格的密度与t = 0.01 * 1024 / 根号3 ，这对应于阈值化最小光线步进的不透明度。  
这种M个候选单元的采样策略依赖于训练过程而占用网格在迭代的早期没有存储相关的信息。在开始的256训练步长下，我们均匀不重复采样M=K✖️128的三次方个单元。对于其他的训练步来说，我们设置M=K乘128的三次方/2将划分分为两个集合。前M/2个单元在所有网格单元中均匀采样。其他的样本采用拒绝采样的方式以限制最近的占用（打球去了，翻译的不是人话）。