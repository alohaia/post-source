---
title: Vulkan
comments: true
mathjax: false
date: 2021-07-24 18:46:41
tags:
    - Computer Graphics
    - Vulkan
    - Index
categories:
    - [Learning, Computer, Computer Graphics]
---

[Vulkan Tutorial](https://vulkan-tutorial.com/) 的学习记录。

<!-- more -->

推荐资源：
- [Physically Based Rendering: From Theory To Implementation](https://www.pbr-book.org/)
- [Learning Modern 3D Graphics Programming](https://paroj.github.io/gltut/)
- [Ray Tracing in One Weekend Book Series](https://github.com/RayTracing/raytracing.github.io)

# Overview

要绘制一个三角形，需要以下步骤：

1. **Instance and physical device selection**：Vulkan 首先需要通过创建 `VkInstance` 来准备（set up）Vulkan API。创建好实例（instance）后，就可以查询 Vulkan 支持的设备并选择一个或多个 `VkPhysicalDevice` 用于后续操作。
2. **Logical device and queue families**：选择合适的硬件设备后，需要创建一个逻辑设备 `VkDevice`。你可以使用它来通过 `VkPhysicalDeviceFeatures` 进行更详细的设置。Vulkan 执行的大多数操作，都是通过提交到一个 `VkQueue` 中异步执行的。队列（queue）是从队列族（queue family）中分配的，每个队列族在其队列中支持一组特定的操作。
3. **Window surface and swap chain**：要将渲染的图像显示到一个窗口，还需要一个窗口表面（window surface）`VkSurfaceKHR` 和一个交换链（swap chain）`VkSwapChainKHR`。后缀 `KHR` 意思是说这两个是 Vulkan 扩展的一部分。`VkSurfaceKHR` 是一个跨平台的窗口抽象，通常通过提供一个原生窗口句柄初始化。

    交换链是一系列最终会展示给用户的图像的集合，并负责管理图像交替用于渲染和显示。

    一些平台允许你通过 `VK_KHR_display_swapchain` 和 `VK_KHR_display` 扩展直接将渲染到显示器上而不与窗口管理器进行交互。你可以用这一特性编写自己的窗口管理器。
4. **Image views and framebuffers**：要绘制从交换链获取的图像，我们需要将图像包装进 `VkImageView` 和 `VkFramebuffer`。Image view 引用要使用的图像的特定部分，framebuffer 则引用用于颜色、深度和模版目标（stencil target）的 image views。由于交换链中由许多不同的图像，所以我们会抢先为每个图像创建一个 image view 和 framebuffer。
5. **Render passes**：Render pass 描述了渲染操作中所使用的图像的类型（如用于透明度、法线的图像）、图像将被如何使用以及图像的内容将被如何处理。渲染通道只描述了图像的类型，而实际由 `VkFramebuffer` 将指定图像“插入对应的卡槽中”。
6. **Graphics pipeline**：Vulkan 的 graphics pipeline 的创建是通过创建 `VkPipeline` 来完成的。它描述了显卡的可配置属性（configurable state），如视口大小、深度缓冲操作（depth buffer operation）以及使用 `VkShaderModule` 对象（object）的可编程属性（programmable state）。`VkShaderModule` 从着色器（shader）字节码（byte code）中创建。我们可以通过引用 render pass 来告知驱动程序哪个渲染对象将在管线中使用。

    Vulkan 的与现存图形 APIs 不同的一大特点就是 graphics pipeline 的几乎所有配置（视口大小、clear color 等除外）都需要预先完成，如果你想对管线做任何修改，都必须重新创建整个管线。这意味着你需要预先创建许多 `VkPipeline` 对象。
7. **Command pools and command buffers**：在前面提到的将 Vulkan 操作提交到队列中前，需要先将操作记录到一个 `VkCommandBuffer` 对象中。Command buffer 从一个 `VkCommandPool` 中分配，`VkCommandPool` 与一个特定的队列族相关。

    要绘制一个简单的三角形，我们需要将以下操作记录到一个 command buffer 中：
    1. 开始 render pass
    2. 绑定 graphics pipeline
    3. 绘制三个顶点（vertice）
    4. 结束 render pass

    由于 framebuffer 中的图像取决于交换链给的是哪张图像，我们需要为每个可能用到的图像记录一个 command buffer，并在绘制时选择正确的 command buffer。另一种低效的替代方法是在每一帧再次记录 command buffer。
8. **Main loop**：现在绘制命令都被包装到一个 command buffer 中，主循环就非常简单了。我们先用 `vkAcquireNextImageKHR` 从交换链中获取一张图像。然后我们可以为图像选择适当的 command buffer，并用 `vKQueueSubmit` 执行。最后，我们将图像返回到交换链并用 `vkQueuePresentKHR` 显示到屏幕上。

简而言之，我们需要以下步骤来绘制一个三角形：
1. 创建一个 `VkInstance` 实例
2. 选择一个受支持的显卡（`VkPhysicalDevice`）
3. 创建 `VkDevice` 和 `VkQueue` 用来绘制和显示
4. 创建一个窗口、窗口表面（window surface）和交换链
5. 将交换链包装进 `VkImageView`
6. 创建 render pass 来指定渲染对象及其用途
7. 为 render pass 创建 framebuffer
8. 设置（set up）graphics pipeline
9. 为交换链中每个可能用到的图像分配一个 command buffer 并用绘制命令将其填充
10. 绘制帧：从交换链中获取图像，提交正确的 command buffer，然后将图像返回到交换链中

# Development environment

按照 [Vulkan Tutorial 的说明](https://vulkan-tutorial.com/Development_environment#page_Linux) 安装即可。

## Guidance for ArchLinux users

- Vulkan：`sudo pacman -S vulkan-icd-loader vulkan-intel nvidia-utils vulkan-devel`，详见 [Vulkan - ArchWiki](https://wiki.archlinux.org/title/Vulkan)
- GLFW：`sudo pacman -S glfw-x11` 或者 `sudo pacman -S glfw-wayland`
- GLM: `sudo pacman -S glm`

## Testing project

```cpp main.cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include <glm/vec4.hpp>
#include <glm/mat4x4.hpp>

#include <iostream>

int main() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    GLFWwindow* window = glfwCreateWindow(800, 600, "Vulkan window", nullptr, nullptr);

    uint32_t extensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

    std::cout << extensionCount << " extensions supported\n";

    glm::mat4 matrix;
    glm::vec4 vec;
    auto test = matrix * vec;

    while(!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
```

```makefile Makefile
CFLAGS = -std=c++17 -O2
LDFLAGS = -lglfw -lvulkan -ldl -lpthread -lX11 -lXxf86vm -lXrandr -lXi

VulkanTest: main.cpp
	g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)

.PHONY: test clean

test: VulkanTest
	./VulkanTest

clean:
	rm -f VulkanTest
```

也可以使用 CMake 而不是直接使用 Make，CMakeLists.txt 如下：

```cmake CMakeLists.txt
cmake_minimum_required(VERSION 3.21)

# Compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O2")

# Project
project(VulkanTest)

# Executables
add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME}
    PRIVATE -lglfw -lvulkan -ldl -lpthread -lX11 -lXxf86vm -lXrandr -lXi
)

# Include directories
target_include_directories(${PROJECT_NAME}
    PRIVATE
        ${PROJECT_SOURCE_DIR}/include
)
```

# Drawing a triangle

## Base code

### General structure

```cpp main.cpp
#include <vulkan/vulkan.h>

#include <iostream>
#include <stdexcept>
#include <cstdlib>

class HelloTriangleApplication {
public:
    void run() {
        initVulkan();
        mainLoop();
        cleanup();
    }

private:
    void initVulkan() {

    }

    void mainLoop() {

    }

    void cleanup() {

    }
};

int main() {
    HelloTriangleApplication app;

    try {
        app.run();
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;
        return EXIT_FAILURE;
    } catch (...) {
        std::cout << "Undefined exception.\n";
    }

    return EXIT_SUCCESS;
}
```

### Resource management

在 C++ 中，有两种方式可以进行自动资源管理：[RAII](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization) 或 `<memory>` 头文件中的智能指针。

Vulkan 对象要么是通过形如 `VkCreateXXX` 的函数创建的，要么是通过形如 `VkAllocateXXX` 的函数由另一个对象分配的。当确定一个对象不再使用后，你需要分别使用对应的 `VkDestroyXXX` 或 `VkFreeXXX` 将其销毁。对不同对象，这些函数的参数有所不同，但是有一个参数是它们共有的：`pAllocator`。这个参数是一个可选参数，你可以用它来指定所创建对象的自定义内存配置器。在这个教程中，我们总是忽视这个参数，将 `nullptr` 传递给该参数。

### Integration GLFW

Vulkan 只做离屏渲染也没问题，但是实际显示一些东西会更加令人兴奋。首先要将 `#include <vulkan/vulkan.h>` 替换为：

{% include_code "00_base_code.cpp:1-2" lang:cpp from:1 to:2 Vulkan/00_base_code.cpp %}

这样就可以包含 GLFW 的自己的定义（definitions）并自动加载 Vulkan 的头文件。接下来在 `run` 函数中的其他函数调用前添加 `initWindow` 函数，我们将要使用这个函数来初始化 GLFW 并创建一个窗口。

{% include_code "00_base_code.cpp:13-18" lang:cpp from:13 to:18 Vulkan/00_base_code.cpp %}

在 `initWindow` 中最先应该调用 `glfwInit()` 函数，以初始化 GLFW 库。因为 GLFW 本来是被设计为创建一个 OpenGL 上下文（context）的，所以紧跟着要使用一条函数调用来告诉它不要创建 OpenGL 上下文。

{% include_code "00_base_code.cpp:26" lang:cpp from:26 to:26 Vulkan/00_base_code.cpp %}

处理缩放窗口需要额外的注意，因此现在我们用另一条函数调用禁用窗口缩放。

{% include_code "00_base_code.cpp:27" lang:cpp from:27 to:27 Vulkan/00_base_code.cpp %}

添加私有成员 `GLFWwindow* window;`，并在 `initWindow` 中完成其初始化。

{% include_code "00_base_code.cpp:8-9" lang:cpp from:8 to:9 Vulkan/00_base_code.cpp %}

{% include_code "00_base_code.cpp:29-29" lang:cpp from:29 to:29 Vulkan/00_base_code.cpp %}

然后在添加 `mainloop` 函数的定义，使程序一直运行直到出现错误或窗口关闭。

{% include_code "00_base_code.cpp:36-40" lang:cpp from:36 to:40 Vulkan/00_base_code.cpp %}

最后，定义 `cleanup` 用来销毁窗口和结束 GLFW。

{% include_code "00_base_code.cpp:42-46" lang:cpp from:42 to:46 Vulkan/00_base_code.cpp %}

编译并运行程序，出现一个标题为“Vulkan”窗口（如下图），就表明成功了，恭喜！

{% asset_img "result_base-code.png" %}

## Instance

### Creating an instance

最先要做的事是通过创建一个实例来初始化 Vulkan 库。这个实例是连接你的应用和 Vulkan 库的纽带，创建它需要向驱动程序指明你的应用的一些细节。

首先添加 `initVulkan` 函数的定义并在其中添加 `createInstance` 函数的调用。

{% include_code "01_instance_creation.cpp:34-36" lang:cpp from:34 to:36 Vulkan/01_instance_creation.cpp %}

另外，添加一个私有成员来保存实例。

{% include_code "01_instance_creation.cpp:23" lang:cpp from:23 to:23 Vulkan/01_instance_creation.cpp %}

现在要创建一个实例，首先需要填充 `VkApplicationInfo` 结构体。这在技术上是可选的，但是可以为驱动提供一些信息以优化我们的程序。添加私有成员函数 `createInstance` 的定义并在函数体中添加如下内容：

{% include_code "01_instance_creation.cpp:53-59" lang:cpp from:53 to:59 Vulkan/01_instance_creation.cpp %}

Vulkan 中许多结构体需要在 `sType` 成员中指定类型。另外，`VkApplicationInfo` 也是许多具有 `pNext` 成员的结构之一，可以指向未来的扩展信息，不过这里我们将其留空为 `nullptr`。

Vulkan 中的许多信息都是通过结构体而不是函数参数传递的。我们还需要（在 `createInstance` 函数体中）填充一个非可选的 `VkInstanceCreateInfo` 结构体。这个结构体告知 Vulkan 驱动我们要用哪些全局扩展（global extensions）和验证层（validation layers）。这里的“全局”的意思是扩展汇兑整个程序生效而不是某个特定驱动。

{% include_code "01_instance_creation.cpp:61-70" lang:cpp from:61 to:70 Vulkan/01_instance_creation.cpp %}

前两个成员无需过多解释；后两个参数指定了所需的全局扩展。Vulkan 是一个平台不可知的 API（platform agnostic API），需要一个扩展来与窗口系统结合。GLFW 内置了一个方便的函数 `glfwGetRequiredInstanceExtensions`，可以返回所需的扩展。

另外还有两个成员指定了要启用的验证层，不过这里先将其留空：

{% include_code "01_instance_creation.cpp:72" lang:cpp from:72 to:72 Vulkan/01_instance_creation.cpp %}

现在我们已经指定了 Vulkan 创建一个实例所需的一切，终于可以发出 `vkCreateInstance` 函数调用了：

{% include_code "01_instance_creation.cpp:74-76" lang:cpp from:74 to:76 Vulkan/01_instance_creation.cpp %}

如你所见，Vulkan 函数参数的一般模式为：
1. 指向包含创建信息的结构体的指针
2. 指向自定义资源配置器回调的指针
3. 指向用来储存新对象句柄的变量的指针

一切顺利的话，`instance` 中将会储存有新对象的句柄，函数将会返回一个 `VkResult` 值 `VK_SUCCESS`。

### Checking for extension support

如果你去查看 [`vkCreateInstance`](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/vkCreateInstance.html) 的文档，你会发现有一个 `VK_ERROR_EXTENSION_NOT_PRESENT` 错误码（扩展不存在）。在前面的代码中，当收到错误码时，我们简单地抛出异常并终止程序。这对某些关键扩展，如窗口系统接口来说没有问题，但是要是我们要检查可选扩展呢？

{% note warning %}
在编写代码时，建议每添加一个扩展就检查一次是否出现 `VK_ERROR_EXTENSION_NOT_PRESENT` 错误码。 
{% endnote %}

要在创建实例前获取支持的扩展的列表，可以使用 `vkEnumerateInstanceExtensionProperties` 函数。该函数需要一个指向用来储存扩展数量的变量的指针，一个用来储存每个扩展的细节的 `VkExtensionProperties` 数组以及一个（第一个）提供用来过滤扩展的验证层的可选参数参数。

要分配 `VkExtensionProperties` 数组，我们先需要知道扩展的数量：

```cpp
uint32_t extensionCount = 0;
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);
```

然后就可以分配数组并获取扩展信息：

```cpp
std::vector<VkExtensionProperties> extensions(extensionCount);
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, extensions.data());
```

可以用以下代码查看扩展的信息：

```cpp
std::cout << "available extensions:\n";

for (const auto& extension : extensions) {
    std::cout << '\t' << extension.extensionName << '\n';
}
```

另外，还可以用 `glfwGetRequiredInstanceExtensions` 函数获取所需扩展的列表。下面是在 `createInstance` 函数中额外添加的代码（可选）：

```cpp
// {{{1 Checking for extension support
// Supported extensions
uint32_t extensionCount = 0;
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);
std::vector<VkExtensionProperties> extensions(extensionCount);
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, extensions.data());

std::cout << "available extensions:\n";

for (const auto& extension : extensions) {
        std::cout << '\t' << extension.extensionName << '\n';
}

// Required extensions
uint32_t requiredExtensionCount = 0;
const char ** requiredExtensions = glfwGetRequiredInstanceExtensions(&requiredExtensionCount);

std::cout << "required extensions:\n";

for(int i = 0; i < requiredExtensionCount; ++i) {
        std::cout << '\t' << requiredExtensions[i] << '\n';
}
// }}}1
```

总的输出如下：

```xxx 输出
available extensions:
	VK_KHR_device_group_creation
	VK_KHR_display
	VK_KHR_external_fence_capabilities
	VK_KHR_external_memory_capabilities
	VK_KHR_external_semaphore_capabilities
	VK_KHR_get_display_properties2
	VK_KHR_get_physical_device_properties2
	VK_KHR_get_surface_capabilities2
	VK_KHR_surface
	VK_KHR_surface_protected_capabilities
	VK_KHR_xcb_surface
	VK_KHR_xlib_surface
	VK_EXT_acquire_xlib_display
	VK_EXT_debug_report
	VK_EXT_debug_utils
	VK_EXT_direct_mode_display
	VK_EXT_display_surface_counter
	VK_KHR_wayland_surface
required extensions:
	VK_KHR_surface
	VK_KHR_xcb_surface
```

### Cleaning up

`VkInstance` 应该在程序退出前销毁。在 `cleanup` 函数中调用 `vkDestroyInstance` 函数：

{% include_code "01_instance_creation.cpp:44-50" lang:cpp from:44 to:50 Vulkan/01_instance_creation.cpp %}

## Validation layers

### What are validation layers?

Vulkan 引入了**验证层**（validation layer）来进行错误检查。一个验证层的函数看起来像这样（注意函数返回值）：

```cpp
VkResult vkCreateInstance(
    const VkInstanceCreateInfo* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkInstance* instance) {

    if (pCreateInfo == nullptr || instance == nullptr) {
        log("Null pointer passed to required parameter!");
        return VK_ERROR_INITIALIZATION_FAILED;
    }

    return real_vkCreateInstance(pCreateInfo, pAllocator, instance);
}
```
验证层可以自由堆叠以获得你感兴趣的功能。你可以简单地在 debug 构建（build）中启用验证层，在 release 构建中禁用验证层。

Vulkan 不提供任何内置的验证层，但是 LunarG Vulkan SDK 提供了一组用以检查常规错误的验证层（[开源地址](https://github.com/KhronosGroup/Vulkan-ValidationLayers)）。只有在安装了验证层的环境中才可以使用验证层，对 ArchLinux 用户，请确认你安装了 vulkan-devel 软件包或单独安装了 vulkan-validation-layers 软件包。

在 Vulkan 中原先有两种不同类型的验证层：实例（instance）特定和设备（device）特定。原先的想法是实例验证层只检查 Vulkan 实例这样的全局对象，而设备验证层只检查与特定设备相关的函数调用。现在设备验证层已经被弃用，所有的函数调用都使用实例验证层。但规范文件仍建议为了兼容性在设备水平（level）也启用验证层。

### Using validation layers

与扩展相同，验证层也需要通过指定其名称来启用。LunarG Vulkan SDK 的所用 validations 都都被绑成一捆放进了名为 `VK_LAYER_KHRONOS_validation` 的 layer 中。

首先添加两个配置变量来分别指定要启用的验证层和是否启用验证层，后者（`enableValidationLayers`）的值由是否在 debug 模式变异来决定。`NDEBUG` 是 C++ 标准的一部分，意思是“not debug”，可用来决定是否启用 [`assert`](https://zh.cppreference.com/w/cpp/error/assert) 宏。

{% include_code "02_validation_layers.cpp:13-21" lang:cpp from:13 to:21 Vulkan/02_validation_layers.cpp %}

我们添加了一个新函数 `checkValidationLayerSupport` 来检查是否所有所需的验证层都是可用的。首先使用 `vkEnumerateInstanceLayerProperties` 函数来获取所有可用的验证层，其用法与 [前面](#checking-for-extension-support) 的 `vkEnumerateInstanceExtensionProperties` 相同。另外，还需要包含 `<cstring>` 库以使用 `strcmp` 函数检查是否所有的 `validationLayers` 中的层都存在于 `availableLayers`。

{% include_code "02_validation_layers.cpp:158-181" lang:cpp from:158 to:181 Vulkan/02_validation_layers.cpp %}

现在我们可以在 `createInstance` 函数中使用 `checkValidationLayerSupport` 函数了：

{% include_code "02_validation_layers.cpp:87-89" lang:cpp from:87 to:89 Vulkan/02_validation_layers.cpp %}

最后，修改 `VkInstanceCreateInfo` 的实例化以包含验证层名称：

```cpp 02_validation_layers.cpp
if (enableValidationLayers) {
    createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
    createInfo.ppEnabledLayerNames = validationLayers.data();
} else {
    createInfo.enabledLayerCount = 0;
}
```

### Message callback

验证层默认会输出调试信息到标准输出，但是我们可以通过提供一个回调函数来自己处理调试信息。这是可选的，你也可以选择 [跳过这小一节](#debugging-instance-creation-and-destruction)。

要设置回调函数来处理调试信息相关细节，我们先要使用 `VK_EXT_debug_utils` 扩展来设立一个*调试信使*（debug messenger）。首先创建 `getRequiredExtensions` 函数，该函数根据验证层是否启用来返回所需扩展列表：

{% include_code "02_validation_layers.cpp:144-156" lang:cpp from:144 to:156 Vulkan/02_validation_layers.cpp %}

GLFW 指定的扩展总是必须的，调试信使的扩展则根据情况添加。注意，这里用了 `VK_EXT_DEBUG_UTILS_EXTENSION_NAME` 宏，等价于字符串 `"VK_EXT_debug_utils"`，但可以利用 LSP 补全功能避免打字错误。

现在可以在 `createInstance` 函数中使用该函数：

{% include_code "02_validation_layers.cpp:103-105" lang:cpp from:103 to:105 Vulkan/02_validation_layers.cpp %}

现在来添加调试回调函数，用 [`PFN_vkDebugUtilsMessengerCallbackEXT`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/PFN_vkDebugUtilsMessengerCallbackEXT.html) 原型创建一个名为 `debugCallback` 的函数。这里额外添加了 [`VKAPI_ATTR`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VKAPI_ATTR.html) 和 [`VKAPI_CALL`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VKAPI_CALL.html) 以确保该函数有 Vulkan 调用所需的正确签名。

{% include_code "02_validation_layers.cpp:183-191" lang:cpp from:183 to:191 Vulkan/02_validation_layers.cpp %}

第一个参数指定消息的严重性，可以是以下的宏之一：
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT`：诊断信息
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT`：提示信息，如资源的创建
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT`：警告信息，算不上错误但很可能是一个 bug
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT`：错误信息，非法并可能导致崩溃

由上到下，随着严重性的增加，宏的值也逐渐增大，你可以写这样的代码：

```cpp
if (messageSeverity >= VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT) {
    // Message is important enough to show
}
```

第二个参数 `messageType` 可以是以下值：
- `VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT`：发生了与规范（specification）和性能（performance）无关的事件
- `VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT`：发生了违反规范或暗示可能发生错误的事件
- `VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT`：潜在的 Vulkan 的非最佳用法

`pCallbackData` 参数指向一个包含信息本身的 `VkDebugUtilsMessengerCallbackDataEXT` 结构体，其中包含一些重要成员：
- `pMessage`：调试信息，是一个 null-terminated 字符串
- `pObjects`：与该信息相关的 Vulkan 对象组成的的数组
- `objectCount`：数组中对象的数量

最后，`pUserData` 包含一个在回调设立期间指定的指针，你可以向其传递你自己的数据。

剩下的就是告诉 Vulkan 这个回调函数了。可能会令人有些意外，Vulkan  中就连调试回调都是由一个句柄管理并且需要显式创建和销毁。这样一个回调是*调试信使*的一部分，你可以有任意多个。为句柄在 `instance` 下面添加一个成员：

{% include_code "02_validation_layers.cpp:52" lang:cpp from:52 to:52 Vulkan/02_validation_layers.cpp %}

在 `initVulkan` 函数中添加 `setupDebugMessenger` 函数的调用。

{% include_code "02_validation_layers.cpp:63-66" lang:cpp from:63 to:66 Vulkan/02_validation_layers.cpp %}

先这样定义 `setupDebugMessenger` 函数：

```cpp
void setupDebugMessenger() {
    if (!enableValidationLayers) return;

}
```

我们需要用关于*信使*和其回调的细节填充一个结构体，创建 `populateDebugMessengerCreateInfo` 函数并在 `setupDebugMessenger` 函数中添加初始化和调用：

{% include_code "02_validation_layers.cpp:125-131" lang:cpp from:125 to:131 Vulkan/02_validation_layers.cpp %}

{% include_code "02_validation_layers.cpp:136-137" lang:cpp from:136 to:137 Vulkan/02_validation_layers.cpp %}

需要填充的 `VkDebugUtilsMessengerCreateInfoEXT` 对象 `createInfo` 的成员：
- `sType`：结构体类型
- `messageSeverity`：指定会触发回调函数的信息的严重性
- `messageType`：指定会触发回调函数的信息的类型
- `pfnUserCallback`：回调函数
- `pUserData`：用户数据，可选，会被传递给前面的 `debugCallback` 函数的同名参数

{% note info %}
还有其他许多方式可以用来配置验证层的消息和调试回调，详见 [extension specification](https://www.khronos.org/registry/vulkan/specs/1.1-extensions/html/vkspec.html#VK_EXT_debug_utils)。
{% endnote %}

填充完成后就是将结构体传递给 `vkCreateDebugUtilsMessengerEXT` 函数以创建 `VkDebugUtilsMessengerEXT` 对象。不幸的是，该函数是一个扩展函数，不会自动加载，我们需要使用 `vkGetInstanceProcAddr` 函数来手动查找其地址。这里我们添加了一个 `CreateDebugUtilsMessengerEXT` 代理函数来在幕后进行处理：

{% include_code "02_validation_layers.cpp:23-30" lang:cpp from:23 to:30 Vulkan/02_validation_layers.cpp %}

> 如果 `vkCreateDebugUtilsMessengerEXT` 加载失败，`vkGetInstanceProcAddr` 会返回 `nullptr`。

现在可以在 `setupDebugMessenger` 函数中调用 `CreateDebugUtilsMessengerEXT` 函数了：

{% include_code "02_validation_layers.cpp:139-141" lang:cpp from:139 to:141 Vulkan/02_validation_layers.cpp %}

倒数第二个参数又是是配置器回调，我们这次也将其设置为 `nullptr`。由于*调试信使*特定于 Vulkan 实例，所以需要通过第一个参数显式指定。你会在后面的其他*子对象*（child objects）中也看到这种模式。其他参数相当直白，无需多余解释。

> 这是 `setupDebugMessenger` 函数最终的样子：
> 
> {% include_code "02_validation_layers.cpp:133-142" lang:cpp from:133 to:142 Vulkan/02_validation_layers.cpp %}
> 
> `VkDebugUtilsMessengerEXT` 对象需要通过调用 `vkDestroyDebugUtilsMessengerEXT` 来清除。类似于 `vkCreateDebugUtilsMessengerEXT`，该函数也需要手动加载。我们在 `CreateDebugUtilsMessengerEXT` 后面又创建了一个函数来进行处理：

{% include_code "02_validation_layers.cpp:32-37" lang:cpp from:32 to:37 Vulkan/02_validation_layers.cpp %}

确保该函数是一个静态成员函数或是一个在类外的函数，以便在 `cleanup` 函数中调用：

{% include_code "02_validation_layers.cpp:74-84" lang:cpp from:74 to:84 Vulkan/02_validation_layers.cpp %}

### Debugging instance creation and destruction

现在我们用验证层为程序添加了调试功能，但是并没有覆盖整个程序。`vkCreateDebugUtilsMessengerEXT` 的调用需要一个已创建的有效实例，`vkDestroyDebugUtilsMessengerEXT` 则必须在实例销毁前被调用。这使我们无法处理 `vkCreateInstance` 和 `vkDestroyInstance` 中出现的问题。

然而如果你详细阅读了 [extension specification](https://www.khronos.org/registry/vulkan/specs/1.1-extensions/html/vkspec.html#VK_EXT_debug_utils)，你会发现一个专门为这两个函数创建独立调试信使的方法。你只需要简单地传递一个指向一个 `VkDebugUtilsMessengerCreateInfoEXT` 结构体的指针给 `VkInstanceCreateInfo` 的 `pNext` 成员。我们已经将填充信使创建信息的代码抽出到了一个单独的函数 `populateDebugMessengerCreateInfo` 中，可以在 `createInstance` 函数中使用该函数：

{% include_code "02_validation_layers.cpp:107-118" lang:cpp from:107 to:118 Vulkan/02_validation_layers.cpp %}

注意 `debugCreateInfo` 被放在了 if-else 结构外以确保其不会在调用 `vkCreateInstance` 前被销毁。这样创建的的信使会在自动在 `vkCreateInstance` 和 `vkDestroyInstance` 被使用并在之后被自动销毁。

### Testing

现在我们来故意引入一些错误：暂时移除 `cleanup` 中的 `DestroyDebugUtilsMessengerEXT` 调用并编译运行你的程序。程序退出时你应该会看到类似这样的信息：

{% asset_img "Validation_layer_error_testing.png" %}

{% note warning %}
再次提醒：对 ArchLinux 用户，请确认你安装了 vulkan-devel 软件包或单独安装了 vulkan-validation-layers 软件包；其他发行版的用户也需确保 Validation Layers 已安装。参考 [Verify the Installation](https://vulkan.lunarg.com/doc/view/1.2.131.1/windows/getting_started.html#user-content-verify-the-installation)。
{% endnote %}

### Configuration

除了在 `VkDebugUtilsMessengerCreateInfoEXT` 中设置标志外，还可以对验证层的行为进行许多其他设置。查看 Vulkan SDK 中的 `Config` 目录，其中的 `vk_layer_settings.txt` 说明了如何进行设置。要为单独的程序进行设置，可将文件复制到项目的 `Debug` 和 `Release` 目录。

## Physical devices and queue families

### Selecting a physical device

通过 `VkInstance` 

### 
