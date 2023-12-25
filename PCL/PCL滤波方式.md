# PCL滤波方式

## 直通滤波器(PassThrough)：用于阈值滤除

直通滤波器(PassThrough)，顾名思义就是使用某个阈值直接过滤掉不符合阈值的滤波器。  

例如有值范围在1到100的十万个数字，直通滤波器设置阈值为<50，则经过直通滤波器过滤后，这十万个数字里所有>=50的数字将被遗弃掉，仅保留满足阈值的所有数字。

## 体素滤波器(VoxelGrid filter)：用于下采样

## 统计离群滤波器(StatisticalOutlierRemoval filter)：用于离群点滤除

## 条件和半径滤波器(Conditional or RadiusOutlier )：用于离群点滤除
