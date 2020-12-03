+++
title = "Trying To Run DOOM With The Raspberry Pi 4 Vulkan Driver"
date = "2020-12-02"
author = "Luigi Cruz"
authorTwitter = "luigifcruz"
showFullContent = false
math = false
slug = "trying-to-run-doom-with-the-raspberry-pi4-vulkan-driver"
cover = "/2020-12-02/trying-to-run-doom-with-the-raspberry-pi4-vulkan-driver/images/v3dv-pi-boot.jpg"
tags = ["RaspberryPi", "Vulkan", "V3DV", "GZDOOM", "DOOM", "RBDOOM-3-BFG", "MESA"]
+++

Vulkan is the new kid in the block for GPU accelerated tasks for Computing and Graphics. This new standard was released by the Khronos Group in 2016. When compared to OpenGL, Vulkan gives the developer more control over the hardware. This approach is widely used by other standards like DirectX 12 and Metal.

The Raspberry Pi Foundation is working with the Open-Source consultancy company called Igalia to develop [Vulkan drivers](https://www.raspberrypi.org/blog/vulkan-raspberry-pi-first-triangle/) for the Raspberry Pi 4 Broadcom SoC. This driver called V3DV has recently [become conformant](https://www.raspberrypi.org/blog/vulkan-update-were-conformant/) with the Vulkan v1.0 specification.

Their roadmap for the future is to improve performance and upgrade the conformant version by supporting required features by the Vulkan Specification. That is interesting because, with a higher conformant version, you can run more Vulkan based software on your favorite Single Board Computer!

My intention in trying to run DOOM games with this driver is to contribute to the development by identifying and requesting useful features used by real applications. I'm not in the development team, but I'm going to try to merge these patches into upstream Mesa. You can follow updates regarding this [here]().

> **Stop!** If you are looking for an easy way to run DOOM in your Pi, this is not a post for you. Try [chocolate-doom](https://www.makeuseof.com/tag/run-doom-raspberry-pi/) instead. This is a technical overview of the current state of the V3DV drivers with DOOM engines.


### The Ultimate DOOM (GZDOOM)

{{< youtube NgtjoOMsMqU >}}

The video above shows a demonstration of [GZDOOM](https://zdoom.org/downloads) opening a `doom.wad` file from disk and correctly rendering the game using the Vulkan engine. The screen resolution was set to 1280x720. At this resolution, the stock Raspberry Pi 4 2GB model can run the game with a framerate between 10 and 15 frames-per-second. A higher framerate is possible with lower visual quality and resolution. But in this case, the game is crashing randomly with Vulkan synchronization errors. The stability and performance will likely improve as more work is committed to the V3DV driver every day.

My patch for this game added support for the `VK_FORMAT_A2R10G10B10_UNORM_PACK32` pixel format. That was easy to add since it's practically the same as the already supported `VK_FORMAT_A2B10G10R10_UNORM_PACK32` as an `RGB10_A2` image format. The game also requested `VkPhysicalDeviceFeatures::depthClamp` which is currently unsupported by the V3DV driver. Overwriting this requirement didn't result in noticeable artifacts.

### DOOM 3: BFG Edition (RBDOOM-3-BFG)

I also wrote another patch for the "DOOM 3: BFG Edition" using the [RBDOOM-3-BFG](https://github.com/RobertBeckebans/RBDOOM-3-BFG) port. This game is larger than the "Ultimate DOOM" and thus requires a big chunk of video memory. That's why I didn't have the chance to test it with my 2GB model to verify if it works with the patch.

{{< img src="images/v3dv-bc-compression.jpg" >}}

Here, the culprit was the `VkPhysicalDeviceFeatures::textureCompressionBC` missing support. This feature is useful to compress the textures on the GPU to reduce memory usage. Luckily, the Raspberry Pi GPU has hardware support for the BC1, BC2, and BC3 formats. The game stopped crashing after I added support for `VK_FORMAT_BC3_UNORM_BLOCK` and `VK_FORMAT_BC1_RGB_UNORM_BLOCK` to the driver. The image above shows the [instancing](https://raw.githubusercontent.com/SaschaWillems/vulkan_slim/master/instancing/instancing.cpp) example from an old version of Sascha Willems' Vulkan demos that requires texture compression working as expected. 

{{< banner >}}