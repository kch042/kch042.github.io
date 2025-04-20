---
title: Nvidia CUDA Compatibility
cover: /img/work.png
mathjax: true
categories: CUDA
tags: 
    - CUDA
    - Nvidia
abbrlink: 29427138
date: 2023-06-28 00:00:00
updated: 2023-06-28 00:00:00
---

When training deep learning model, we often use Nvidia GPU on the server to accelerate the training, and use **docker container** to avoid messing up the server. 

However, there are so many problems we may come up when building the environment. One of them is **cuda version compatibility**.

## Hierarchy

There are 3 layers of abstraction when using Nvidia GPU (cuda) to train the model.

- nvidia runtime (`libcudart.so`, `cuda-toolkit`, `nvcc`)
- nvidia driver
    - nvidia user-mode driver (`libcuda.so`)
    - nvidia kernel-mode driver (`nvidia.ko`)

![](/img/cuda/cuda-hierarchy.png)

We must make sure these 3 layers have versions compatible with each other to run the program!!

## Check versions

### Runtime
We have two methods to check runtime cuda versions.

1. `nvcc -V`

`nvcc` is a compiler to compile the deep learning source code. It will use `cuda-toolkit` to convert the source code into cuda language.
![](/img/cuda/nvcc-version.png)

As we can see the runtime version is `10.2.89`.

2. find the version of `libcudart.so`

We are using `libcudart.so` for runtime linking.
![](/img/cuda/libcudart.png)

Note that `libcudart.so` is a **soft link**, and refer to a specific version.
![](/img/cuda/libcudart-version.png)

As we can see, it is refering to the runtime cuda version `10.2.89`.

### Driver
Similar to checking runtime version, we have two ways to check driver versions.

But for driver we have 2 versions to consider, user and kernel. **For now, just consider them to be the same.**

1. find the `libcuda.so`

![](/img/cuda/libcuda.png)

We are interested in `libcuda.so` which is also a soft link.

![](/img/cuda/libcuda-version.png)

We can see our driver version is `470.182.03`

2. `nvidia-smi`

nvidia-smi is an application installed when installing the nvidia driver. 

![](/img/cuda/nvidia-smi.png)

We can see top row, we have `Driver Version: 470.182.03`

### The simplest version compatibility
Also, there's an `CUDA version: 11.4` on the right, it tells us that the current driver version is supposed to match the runtime cuda version `11.4` to work.

The above version compatibility sounds critical and not so flexible, right? Fortunately, Nvidia introduced some mechanisms to relax the version compatibility.

- Backward Compatibility
- Minor Version Compatibility
- Forward Compatibility

## Backward Compatibility
Backward compatibility allows us, given current driver version, run **older cuda runtime version**.

In the above examples, we can only match driver version
`470.182.03` with runtime version `11.4`. With backward compatibity, driver version `470.182.03` is now compatible with runtime version `<=11.4` !!

## Minor Version Compatibility
By backward compatibility, we know that to use a certain cuda runtime version, we must have a driver whose version is **large** enough.

For runtime cuda version smaller than `11`, we have a **different** driver version threshold for each **minor** version.
![](/img/cuda/cuda-min10.png)


From cuda `11` onwards, we have the **same** driver version threshold for each minor version of the major version.
![](/img/cuda/cuda-min11.png)

## Forward Compatibility

**This mechanism is mostly for GPUs of data center level. (e.g. Tesla brand). So for normal users, it is rarely used.**

However, we might encounter error like `Error 804: forward compatibility was attempted on non supported HW` when building the environment. So let's take a look on the mechanism and see why this might happen.

For data centers, it is troublesome to upgrade driver versions (due to the need of rigorous testing), but they also need a new enough driver to use newer runtime toolkit. 
![](/img/cuda/libcuda-user-kernel-same-version.png)


So Nvidia provides a more convenient solution: upgrading only **user-mode driver** allows you to use newer runtime toolkit !!

In the previous chapters, we consider kernel-mode and user-mode driver to be of the same version. However, from now on, **with forward compatibility, two driver versions can differ.**

![](/img/cuda/libcuda-user-kernel-diff-version.png)

### Error 804: forward compatibility was attempted on non supported HW

This happens if the system recognizes our driver versions are different.

In the following example, we have kernel-mode driver version `470.182.03`.
![](/img/cuda/libcuda-kernel-version.png)

And the user-mode driver version is `510.108.03`.
![](/img/cuda/libcuda-user-version.png)

#### Why will this happen ?
The container image has its own `libcuda.so`, and the host machine has its own `libcuda.so`. 

When starting a container, nvidia docker will select the best `libcuda.so` to allow us using the latest cuda runtime toolkit. However, this will requires forward compatibilitym, which is not supported on our GPU, so the error occurs !!

#### How to solve it ?
In the previous example, we have kernel-mode driver version `470.182.03` and user-mode driver version `510.108.03`.

We only need to change the user-mode driver version to the same version as the kernel-mode driver. (i.e. both `470.182.03`)
![](/img/cuda/error804-sol-step1.png)

Change directory into where `libcuda.so` is stored.
![](/img/cuda/error804-sol-step2.png)

Rebuild the softlink
![](/img/cuda/error804-sol-step3.png)


And voila
![](/img/cuda/error804-sol-step4.png)

Check `nvidia-smi`
![](/img/cuda/error804-sol-step5.png)

Notice that `CUDA Version` is changed from `11.6` to `11.4`. But we are still able to run program compiled with cuda runtime `11.6`. (thanks to the minor version compatibility)
![](/img/cuda/error804-sol-step6.png)


## Conclusion
In this article, we introduce the hierarchy of cuda layers, their compatibility requirements, and three mechanisms for compatibility.


## References
1. [CUDA Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#minor-version-compatibility)
2. [PyTorch的CUDA错误：Error 804: forward compatibility was attempted on non supported HW](https://zhuanlan.zhihu.com/p/361545761)