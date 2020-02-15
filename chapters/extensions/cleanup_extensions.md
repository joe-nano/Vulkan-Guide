# Cleanup Extensions

> These are extension which are unofficially called "cleanup extension". The Vulkan Guide defines them as cleanup extensions due to their nature of only adding a small bit of functionality, usually one that requires little to no hardware support and mainly to make the API easier to use.

# VK_KHR_driver_properties

The `VK_KHR_driver_properties` was added in Vulkan 1.2 core as it adds more information to query about each implementation. The [VkDriverId](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/html/vkspec.html#VkDriverId) will registered vendor's ID of the implementation. The [VkConformanceVersion](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/html/vkspec.html#VkConformanceVersion) displays which version of [the Vulkan Conformance Test Suite](../chapters/vulkan_cts.md) the implementation passed.

# VK_EXT_host_query_reset

This extension was promoted in Vulkan 1.2 and allows an application to call `vkResetQueryPool` from the host instead of needing to setup logic to submit `vkCmdResetQueryPool` since this is mainly just a quick write to memory for most implementations and little overhead to calling.

# VK_KHR_separate_depth_stencil_layouts

This extension was promoted in Vulkan 1.2 and allows an application when using a depth/stencil format to do an image translation on each the depth and stencil separately. Starting in Vulkan 1.2 this functionality is required for all implementations.

# pNext Expansion

There have been a few times where the Vulkan Working Group realized that some structs in the original 1.0 Vulkan spec were missing the ability to be extended properly due to missing `sType`/`pNext`.

Keeping backward compatibility between versions is very important, so the best solution was to create an extension to amend the mistake. These extensions mainly new structs, but also need to create new function entry points to make use of the new structs.

The current list of extensions that fit this category are:
- `VK_KHR_get_memory_requirements2`
    - Added to core in Vulkan 1.1
- `VK_KHR_get_physical_device_properties2`
    - Added to core in Vulkan 1.1
- `VK_KHR_bind_memory2`
    - Added to core in Vulkan 1.1
- `VK_KHR_create_renderpass2`
    - Added to core in Vulkan 1.2

All of these are very simple extensions and were promoted to core in their respective versions to make it easier to use without having to query for their support.

> `VK_KHR_get_physical_device_properties2` has additional functionality as it adds the ability to query feature support for extensions and newer Vulkan versions. It has become a requirement for most other Vulkan extensions because of this.

## It is fine to not use these

Unless you need to make use of one of the extensions that rely on the above extensions, it is normally ok to use the original function/structs still.

One possible way to handle this is as followed:

```
void HandleVkBindImageMemoryInfo(const VkBindImageMemoryInfo* info) {
    // ...
}

//
// Entry points into tool/implementation
//
void vkBindImageMemory(VkDevice device,
                       VkImage image,
                       VkDeviceMemory memory,
                       VkDeviceSize memoryOffset)
{
    VkBindImageMemoryInfo info;
    // original call doesn't have a pNext or sType
    info.sType = VK_STRUCTURE_TYPE_BIND_IMAGE_MEMORY_INFO;
    info.pNext = nullptr;

    // Match the rest of struct the same
    info.image = image;
    info.memory = memory;
    info.memoryOffset = memoryOffset;

    HandleVkBindImageMemoryInfo(&info);
}

void vkBindImageMemory2(VkDevice device,
                        uint32_t bindInfoCount, const
                        VkBindImageMemoryInfo* pBindInfos)
{
    for (uint32_t i = 0; i < bindInfoCount; i++) {
        HandleVkBindImageMemoryInfo(pBindInfos[i]);
    }
}
```