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
枚举当前系统中所有的物理设备或者GPU的数量，并且调用vkEnumeratePhysicalDevices()API函数。

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

注：上下文之间少了枚举设备层扩展和启动设备层扩展

```
/*** 4. 创建设备 ***/
// 这里省略掉了queueCount, queueProperties, memoryPropperties, gpuProps, device的声明

// 获取队列和队列类型
vkGetPhysicalDeviceQueueFamilyProperties(gpu, &ququeCount, queueProperties);

// 获取物理设备或者Gpu中的内存属性
vkGetPhysicalDeviceMemoryProperties(gpu, &memoryProperties);

// 获取物理设备或者gpu的属性
vkGetPhysicalDeviceProperties(gpu, &gpuProps);

// 从物理设备创建逻辑设备对象
vkDeviceCreateInfo deviceInfo = {};
vkCreateDevice(gpuList[0], &deviceInfo, NULL, &device);

```

# 2.2.2 交换链初始化————查询WSI扩展

```
/*** 5. 展示层的初始化 ***/

// 创建空窗口
CreateWindowEx(...);     // windows
xcb_create_window(...);  // linux

// 查询SWI扩展函数并保存为函数指针的形式
// 例如: vcCreateSwapchainKHR, vkCreateSwapchainKHR......
// 创建一个抽象表面对象
vkWin32SurfaceCreateInfoKHR createInfo = {};
vkCreateWin32SurfaceKHR(instance, &createInfo, NULL, &surface); // ZT_TAG：surface哪里来的？

// 从所有的队列中，选择一个支持当前展示层的队列
foreach Queue in All Queues {
    vkGetPhysicalDeviceSurfaceSupportKHR(gpu, queueIndex, surface, &isPresentationSupported);   // ZT_TAG queueIndex哪里来的
    if (isPresentationSupported) {
        graphicsQueueFamilyIndex = Queue.index;
        break;
    }
}

// 获取兼容层的队列，它同时也是一个图形队列
vkGetDeviceQueue(device, graphicsQueueFamilyIndex, 0, &queue);  // ZT_TAG queue哪里来的

// 分配内存空间来记录绘制表面的格式的总数
uint32_t formatCount;
vkGetPhysicalDeviceSurfaceFormatsKHR(gpu, surface, &formatCount, NULL);
vkSurfaceFormatKHR *surfaceFormats = allocate memroy(formatCount * VKSurfaceFormatKHR); // ZT_TAG 申请堆内存

// 将表面格式保存到VKSurfaceFormatKHR对象中
vkGetPhysicalDeviceSurfaceFormatsKHR(gpu, surface, &formatCount, surfaceFormats);
```







