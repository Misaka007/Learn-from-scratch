# PCL 新增自定义点云类型

## 一、需求

PCL库自身定义了很多点云类型，但是在使用过程中发现没有同时含有位置信息、RGB、点云强度和标签信息的类型，我们需要自定义点云类型。

## 二、方法

### 需要包含头文件

```cpp
#include <pcl/point_types.h>
```

### 在PCL头文件之前定义 PCL_NO_PRECOMPILE

```cpp
#define PCL_NO_PRECOMPILE
#include <pcl/point_types.h>
```

### 添加一个新的点云类型

```cpp
struct PointXYZIRGBL {
    PCL_ADD_POINT4D;
    PCL_ADD_RGB;
    PCL_ADD_INTENSITY;
 uint32_t label;
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW;
} EIGEN_ALIGN16;

```

### 在PCL中注册

```cpp
POINT_CLOUD_REGISTER_POINT_STRUCT(
PointXYZIRGBL,
(float, x, x)
(float, y, y)
(float, z, z)
(uint8_t, r, r)
(uint8_t, g, g)
(uint8_t, b, b)
(float, rgb, rgb)
(float, intensity, intensity)
(uint32_t, label, label)
)
```

至此，该新注册的点云类型即可正常使用。
