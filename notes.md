## There are three things that happen with vulkan:

1. Instance and physical device selection.
This chooses what GPU you're using, creates a vkInstance which exposes what API's you're going to call, you can query for your PhysicalDevice's, or properties like VRAM size, or your device capabilities.
This process you could also decide if you want to use an integrated or dedicated gpu, for example.
2. Logical device and queue families. 
When I choose what hardware I want to use, then I need to create a VkDevice. This describes which PhysicalDeviceFeatures i want to be using such as multi view port rendering and 64-bit floats.
Then i need to specify which queue families I would like to use.

most operations in vulkan are performed by submitting them into a VkQueue. Queues are just allocated from queue families, where each family supports a specific set of operations within that said queue.

A good example of this would be where there are separate queue families for the following: 
Graphics, 
Compute, and 
Memory Transfer Operations.

Generally, all graphics cards that support vulkan will also support all queue operations that we will use in this tutorial.

3. Window Surface and swapchain
we need a window surface, VkSurfaceKHR, and a swapchain ( VkSwapChainKHR). the KHR means that these objects are part of a Vulkan Extension. Vulkan's API is platform agnostic so we need to use a standardized WSI extension to interact with the window manager for each platform. This surface is a cross-platform abstraction over windows to render to and is generally instantiated by providing a reference to the native window handle, like HWND on windows. But, vulkanalia has optional integration with the winit crate for rust, which we will be using.

What is the swapchain? It is a collection of render targets. its purpose is to make sure that the image that we are currently rendering to is different from the one that is currently on the screen. This is to ensure that only complete images are shown. Whenever we want to draw a frame, we need to get the swapchain to provide us with an image to render to. When we finish drawing the frame, the image is then returned to the swapchain for it to be presented on the screen at some point. We can have varying amounts of render targets and conditions for presenting finished images to the screen. The number depends on the present mode like Vsync, and triple buffering.

4. Image Views and Framebuffers.
To draw to an image acquired from the swapchain, we have to wrap it into a VkImageView and a VkFrameBuffer. 
The image view references a specific part of an image to be used, and then the frame buffer references image views that are to be used for things such as color, depth and stencil targets.

5. Render passes
These describe the type of images that are used during rendering operations, how they will be used and how their contents should be treated.
We're gonna tell vulkan that we will use a single image as a color target, and that we want it be cleared to a solid color right before the drawing operation. Whereas the render pass only describes the types of images, a VkFrameBuffer actually binds specific images to these slots.

6. Graphics Pipeline
this is setup with the VkPipeline object. It will describe the configurable state of the graphics card, such as view port size, depth buffer operations, and then the programmable state using VkShaderModule objects. These objects are created from shader byte code. The driver will also need to know which render targets will be used in the pipeline. which we specify by referencing the Render pass.

the thing with Vulkan is that almost all configuration to the pipeline needs to be set in advance. If i want to switch to a different shader or change my vertex layout, i need to create several VkPipeline objects in advance for the diff combinations I need for my rendering operations. Some basic configs like viewport or clear color can be changed dynamically. But, this allows for more predictable behavior and better optimization because I need to be more explicit in my writing.

7. Command pools and command buffers.
Many operations in Vulkan that we want to execute, such as drawing operations, have to be submitted into a queue. These operations need to be recorded with a VkCommandBuffer before they can be submitted. These command buffers are allocated from a CommandPool that is associated with the specific queue family. to draw a triangle, we need to record a command buffer with these following operations:
	1. Begin the render pass
	2. Bind the graphics pipeline
	3. draw 3 vertices
	4. end the render pass

Because the image in the framebuffer depends on which specific image the swap chain will give us, we need to record a command buffer for each possible image and select the right one at draw time. The alternative could be to record the command buffer again every frame, which is a lot more taxing

8. Main loop
now that the drawing commands have been wrapped into a command buffer, the main loop is quite straightforward. 
	1. We first acquire an image from the swapchain with vkAcquireNextImageKHR. 
	2. Then we can select the appropriate command buffer for that image and execute it with vkQueueSubmit. 
	3. Finally, we return the image to the swapchain for presentation to the screen with vkQueuePresentKHR.

