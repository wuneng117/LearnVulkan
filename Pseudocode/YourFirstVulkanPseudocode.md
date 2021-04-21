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

```
/*** 6. 创建交换链 ***/

// 开始将指令记录到指令缓存当中
vkBeginCommandBuffer(cmd, &cmdBufInfo); //ZT_TAG 这2个哪里来的，怎么越写越懒啊

// 获取表面的性能参数
vkGetPhysicalDeviceSurfaceCapabilitiesKHR(gpu, surface, &surfCapabilities);

// 获取表面的展示模式
vkGetPhysicalDeviceSurfacePresentModesKHR(gpu, surface, &presentModeCount, NULL);
VkPresentModeKHR presentModes[presentModeCount];
vkGetPhysicalDeviceSurfacePresentModesKHR(gpu, surface, &presentModeCount, presentModes);

// 创建交换链
VkSwapchainCreateInfoKHR swapChainInfo = {};
fpCreateSwapchainKHR(device, &swapChainInfo, NULL, &swapChain);
// 创建所需交换链图像对应的图像视图
vkGetSwapChainImagesKHR(device, swapChain, &swapchainImageCount, NULL);
VkImage swapchainImages[swapchainImageCount];
vkGetSwapChainImagesKHR(device, swapChain, &swapchainImageCount, swapchainImages);

// 获取交换链中的图像
foreach swapchainImages {
    // 设置布局方式，与具体驱动的实现兼容
    SetImageLayout(...)

    // 插入流水线屏障
    VkImageMemoryBarrier imagerMemoryBarrier = {...};
    vkCmdPipelineBarrier(cmd, srtStages, destStages, 0, 0, NULL, 0, NULL, 1, &imageMemoryBarrier);
    
    // 创建图像视图
    vkCreateImageView(device, &colorImageView, NULL, &scBuffer.view);

    // 保存图像视图， 为应用程序所用
    buffers.push_back(scBuffer);
}
```

```
/*** 7. 创建深度图像 ***/
// 查询当前物理设备所支持的格式
vkGetPhysicalDeviceFormatsProperties(gpus, depthFormat, &properties);

// 创建一个图像对象
vkCreateImage(device, &imageInfo, NULL, &imageObject);

// 获取图像资源所需的内存空间
vkGetImageMemoryRequirements(device, image, &memoryRequirements);

// 分配内存
vkAllocateMemory(device, &memAlloc, NULL, &memorys);

// 绑定内存
vkBindImageMemory(device, imageObject, mem, 0);
// 设置图像布局，可用于当前设备
SetImageLayout(...)

//插入新的流水线屏障，确保刚才设置的图像布局在图像被真正使用之前就已经创建了
vkCmdPipelineBarrier(cmd, srtStages, destStages, 0, 0, NULL, 0, NULL, 1, &imagePipelineBarrier);

// 创建图像视图
vkCreateImageView(device, &imgViewInfo, NULL, &view);
```

# 2.2.3 着色器的支持

```
/*** 8. 构建着色器模块 ***/
VkPipelineShaderStageCreateInfo vtxShdrStage = {...};
VkShaderModuleCreateInfo moduleCreateInfo = {...};

// spvVertexShaderData中保存了二进制格式的顶点着色器代码
moduleCreateInfo.pCode = spvVertexShaderData;

// 在设备端创建着色器模块
vkCreateShaderModule(device, &moduleCreateInfo, NULL, &vtxShdrStage.module);
```

# 2.2.4 构建布局————描述符与流水线布局
描述符负责将资源与着色器通过布局绑定的方式关联起来。我们通用用它来绑定一致和采样器变量到着色器中。
```
/*** 9. 创建描述符以及流水线布局 ***/

// 描述符的定义类型与着色器中是对应的
VkDescriptorSetLayoutBinding layoutBind[2];

layoutBind[0].descriptorType    = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
layoutBind[0].binding           = 0;
layoutBind[0].stageFlags        = VK_SHADER_STAGE_VERTEX_BIT;

layoutBind[1].descriptorType    = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SIMPLER;
layoutBind[1].binding           = 0;
layoutBind[1].stageFlags        = VK_SHADER_STAGE_FRAGMENT_BIT;

// 设置布局绑定，创建描述符集
VKDescriptorSetLayoutCreateInfo descriptorLayout = {};
descriptorLayout.pBindings = layoutBind;

VkDescriptorSetLayout descLayout[2];
vkCreateDescriptorSetLayout(device, &descriptorLayout, NULL, descLayout.data());

// 现在我们可以使用描述符来创建一个流水线布局了
VkPipelineLayoutCreateInfo pipelineLayoutCI = {...};
pipelineLayoutCI.pSetLayouts = descLayout.data();
vkCreatePipelineLayout(device, &pipelineLayoutCI, NULL, &pipelineLayout);
```

# 2.2.5 创建渲染通道————定义通道属性

```
/*** 10. 渲染通道 ***/

// 定义两个附件， 分别对应了颜色和深度缓存
VkAttachmentDescription attachments[2];
attachments[0].format = colorImageformat;
attachments[0].loadOp = clear ? VK_ATTACHMENT_LOAD_OP_CLEAR : VK_ATTACHMENT_LOAD_OP_DONT_CARE;
attachments[1].format = depthImageformat;
attachments[1].loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;

VkAttachmentReference colorReference, depthReference = {...};

// 使用颜色图像和深度图像来构建子通道
VKSubpassDescription subpass = {};
subpass.pcolorAttachments = &colorReference;
subpass.pdepthAttachments = &depthReference;

// 定义渲染通道控制的结构体
VkRednerPassCreateInfo rpInfo = { &attachments, &subpass ...};
vkCreateRenderPass(device, &rpInfo, NULL, &renderPass);

```

# 2.2.6 帧缓存————将绘制推向关联到渲染通道

```
/*** 11. 创建帧缓存 ***/

VKImageView attachments[2]; // 0表示颜色， 1表示深度
attachments[1] = Depth.view;

VkFramebufferCreateInfo fbInfo = {};
fbInfo.renderPass = renderPass;     // 渲染缓存对象
fbInfo.pAttachments = attachments;  // 图像视图附件
fbInfo.width = width;   // 帧缓存宽度
fbInfo.height = height; // 帧缓存高度

// 为交换链中的每幅图像的帧缓存对象分配内存，每幅图像只有一个帧缓存
VkFramebuffer framebuffers[交换链中的绘制图像数量];

foreach (drawing buffer in swapchain) {
    attachments[0] = currentSwapChainDrawImage.view;
    vkCreateFramebuffer(device, &fbInfo, NULL, &framebuffers[i]);
}
```

# 2.2.7 产生几何体————在GPU内存中保存一个顶点

```
/*** 12. 产生几何体， 在GPU内存中保存顶点 ***/

static const VertexWithColor triangleData[] = {
/*  {  x,      y,    z,    w,    r,    g,    b,    a  },*/
    { 0.0f,   1.0f, 0.0f, 1.0f, 1.0f, 0.0f, 0.0f, 1.0 },
    { -1.0f, -1.0f, 0.0f, 1.0f, 0.0f, 0.0f, 1.0f, 1.0 },
    { 1.0f,  -1.0f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f, 1.0 },
};

VkBuffer buffer;
VkMemoryRequirements mem_requirement;
VkDeviceMemory deviceMemory;

// 创建缓存对象，查询所需的内存空间，分配内存
VkBufferCreateInfo buffer_info = {...};
vkCreateBuffer(device, &buffer_info, NULL, &buffer);

vkGetbufferMemoryRequirements(device, buffer, &mem_requirement);

VkMemoryAllocateInfo alloc_info = {...};
vkAllocateMemory(device, &alloc_info, NULL, &(deviceMemory));

// 通过映射的方式，将三角形几何数据复制到GPU端
uint8_t *pData;
vkMapMemory(device, deviceMemory, 0, mem_requirement.size, 0, pData);
memcpy(pData, triangleData, dataSize); // 复制数据
vkUnmapMemory(device, deviceMemory);

// 绑定分配后的内存
vkBindBufferMemory(device, buffer, deviceMemory, 0);

/*** 13. 顶点绑定 ***/
VkVertexInputBindingDescription viBinding;
viBinding.binding = 0;
viBinding.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;
viBinding.stride = sizeof(triangleData);    // 数据间隔

VkVertexInputAttributeDescription viAttribs[2];
viAttribs[0].binding = 0;
viAttribs[0].location = 0;
viAttribs[0].format = VK_FORMAT_R32G32B32A32_SFLOAT;
viAttribs[0].offset = 0;
viAttribs[1].binding = 0;
viAttribs[1].location = 1;
viAttribs[1].format = VK_FORMAT_R32G32B32A32_SFLOAT;
viAttribs[1].offset = 16;
```


