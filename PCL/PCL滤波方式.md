# PCL滤波方式

## 直通滤波器(PassThrough)：用于阈值滤除

直通滤波器(PassThrough)，顾名思义就是使用某个阈值直接过滤掉不符合阈值的滤波器。  
例如有值范围在1到100的十万个数字，直通滤波器设置阈值为<50，则经过直通滤波器过滤后，这十万个数字里所有>=50的数字将被遗弃掉，仅保留满足阈值的所有数字。

```cpp
#include <iostream>
#include <pcl/point_types.h>
#include <pcl/filters/passthrough.h>

int main () {
  // 原始点云指针
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud (new pcl::PointCloud<pcl::PointXYZ>);
  // 过滤后的点云指针
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud_filtered (new pcl::PointCloud<pcl::PointXYZ >);

  // 设置点云为无序点云，共5个点
  cloud->width = 5;
  cloud->height = 1;
  cloud->points.resize (cloud->width * cloud->height);

  for (auto& point: *cloud) //随机生成点坐标
  {
    point.x = 1024 * rand () / (RAND_MAX + 1.0f);
    point.y = 1024 * rand () / (RAND_MAX + 1.0f);
    point.z = 1024 * rand () / (RAND_MAX + 1.0f);
  }

  // 输出创建的点
  std::cerr << "Cloud before filtering: " << std::endl;
  for (const auto& point: *cloud)
    std::cerr << " " << point.x << " " << point.y << " " << point.z << std::endl;

  // 创建过滤器对象
  pcl::PassThrough<pcl::PointXYZ> pass;
  pass.setInputCloud (cloud); // 填入数据
  pass.setFilterFieldName ("z"); // 设置要过滤的维度名，这里为z向维度
  pass.setFilterLimits (580.0, 900); // 设置滤波阈值
  //pass.setFilterLimitsNegative (true);
  pass.filter (*cloud_filtered); // 开始滤波

  // 输出滤波后的点
  std::cerr << "Cloud after filtering: " << std::endl;
  for (const auto& point: *cloud_filtered)
    std::cerr << " " << point.x << " " << point.y << " " << point.z << std::endl;

  return (0);
}
```

## 体素滤波器(VoxelGrid filter)：用于下采样

将点云按照空间三维划分体素 (空间网格)，并使用数学方法将每个体素内的点云替换成一个点，从而减少点云数量。

```cpp
#include <iostream>
#include <pcl/io/pcd_io.h>
#include <pcl/point_types.h>
#include <pcl/filters/voxel_grid.h>

int main () {
  pcl::PCLPointCloud2::Ptr cloud (new pcl::PCLPointCloud2 ());
  pcl::PCLPointCloud2::Ptr cloud_filtered (new pcl::PCLPointCloud2 ());

  // 创建点云读取器对象
  pcl::PCDReader reader;
  // 将下载的pcd文件路径替代以下路径
  reader.read ("table_scene_lms400.pcd", *cloud);

  std::cerr << "PointCloud before filtering: " << cloud->width * cloud->height << " data points (" << pcl::getFieldsList (*cloud) << ").";

  // 创建滤波器对象
  pcl::VoxelGrid<pcl::PCLPointCloud2> sor;
  sor.setInputCloud (cloud);
  sor.setLeafSize (0.01f, 0.01f, 0.01f);
  sor.filter (*cloud_filtered);

  std::cerr << "PointCloud after filtering: " << cloud_filtered->width * cloud_filtered->height << " data points (" << pcl::getFieldsList (*cloud_filtered) << ").";

  pcl::PCDWriter writer;
  writer.write ("table_scene_lms400_downsampled.pcd", *cloud_filtered, Eigen::Vector4f::Zero (), Eigen::Quaternionf::Identity (), false);

  return (0);
}
```

## 统计离群滤波器(StatisticalOutlierRemoval filter)：用于离群点滤除

统计离群滤波器可以移除离群点，即那些在平均距离意义下偏离其邻居太远的点。
```cpp 
#include <iostream>
#include <pcl/io/pcd_io.h>
#include <pcl/point_types.h>
#include <pcl/filters/statistical_outlier_removal.h>

int main () {
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud (new pcl::PointCloud<pcl::PointXYZ>);
  pcl::PointCloud<pcl::PointXYZ>::Ptr cloud_filtered (new pcl::PointCloud<pcl::PointXYZ>);

  // 填入点云数据
  pcl::PCDReader reader;
  reader.read<pcl::PointXYZ> ("table_scene_lms400.pcd", *cloud);

  std::cerr << "Cloud before filtering: " << std::endl;
  std::cerr << *cloud << std::endl;

  // 创建滤波器对象
  pcl::StatisticalOutlierRemoval<pcl::PointXYZ> sor;
  sor.setInputCloud (cloud);
  sor.setMeanK (50);
  sor.setStddevMulThresh (1.0);
  sor.filter (*cloud_filtered);

  std::cerr << "Cloud after filtering: " << std::endl;
  std::cerr << *cloud_filtered << std::endl;

  pcl::PCDWriter writer;
  writer.write<pcl::PointXYZ> ("table_scene_lms400_inliers.pcd", *cloud_filtered, false);

  sor.setNegative (true);
  sor.filter (*cloud_filtered);
  writer.write<pcl::PointXYZ> ("table_scene_lms400_outliers.pcd", *cloud_filtered, false);

  return (0);
}

```

## 条件和半径滤波器(Conditional or RadiusOutlier )：用于离群点滤除

条件和半径滤波器可以根据条件和半径来滤除离群点。
