# 2.2.1初始化————与设备握手的过程
## 1)枚举实例层属性

```
/*** 1.枚举势力的层属性 ***/
// 获取实例层的数量
uint32_t instanceLayerCount;

// 设置第二个参数为NULL即可返回层的数量`
vkEnumerateInstanceLayerProperties(&instanceLayerCount, NULL);

vkLayerProperties *layerProperty = NULL;
vkEnumerateInstanceLayerProperties(*instanceLayerCount, layerProperty);

// 遍历每一个可用的实例层，获取功能扩展信息
foreach layerPropery {
    VKExtensionProperties *instanceExtensions;
    res = vkEnumerateInstanceExtensionProperties(layer_name, &instanceLayerCount, instanceExtensions);
}
```

## 2)创建实例

```
/*** 2. 创建实例 ***/
// vulkan实例对象
VKInstance instance;
VKInstanceCreateInfo instanceInfo = {};

// 设置实例中需要开启的层的名称
instanceInfo.ppEnabledLayerNames  = {
    "VK_LAYER_LUNARG_standard_validation",
    "VK_LAYER_LUNARG_object_tracker"};

// 设置实例中需要开启的功能扩展
instanceInfo.ppEnabledExtensionNames = {
    VK_KHR_SURFACE_EXTENSION_NAME,
    VK_KHR_WIN32_SURFACE_EXTENSION_NAME};

vkCreateInstance(&instanceInfo, NULL, &instance);
```

## 3)创建设备
枚举当前系统中所有的物理设备或者GPU的数量，并且调用vkEnumeratePhysicalDevicesa()API函数。

```
/*** 3.  枚举物理设备 ***/
VKPhysicalDevice gpu;    // gpu
uint32_t gpuCount;    // gpu的数量
vector<VKPhysicalDevice>gpuList;    // gpu列表

// 获取gpu的数量
vkEnumeratePhysicalDevices(instance, &gpuCount, NULL);

// 获取gpu的信息
vkEnumeratePhysicalDevices(instance, &gpuCount, gpuList);
```






