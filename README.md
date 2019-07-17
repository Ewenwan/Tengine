# Tengine Overview

[![GitHub license](http://OAID.github.io/pics/apache_2.0.svg)](./LICENSE)

TEngine是由OPEN AI LAB开发的一款轻量级模块化的高性能神经网络推理引擎，专门针对Arm嵌入式设备优化，提供超过所有已知开源框架的无与伦比的性能，并且无需依赖第三方库，可跨平台使用支持Android，Liunx。

OPEN AI LAB是Arm发起的，旨在推动Arm嵌入式智能机器的产业发展，为智能机器应用场景构建嵌入式SoC基础计算框架，推动芯片、硬件、算法软件整个产业链的深度协作，实现有计算的地方就有人工智能。

# TEngine介绍

TEngine支持各类常见卷积神经网络，包括SqueezeNet，MobileNet，AlexNet，ResNet等，支持层融合、8位量化等优化策略。并且通过调用针对不同CPU微构架优化的HCL库，将Arm CPU的性能充分挖掘出来。例如在RK3399平台Cortex-A72单线程运行移动端常用的MobileNet一次只需要111ms，比NCNN快50%以上1。TEngine目前已经适配Cortex-A7/A17,Cortex-A53/A72/A73，后续还将适配Arm最新构架CPU Cortex-A55/A75，并且支持8位点积指令和半精度浮点指令，大幅度提升推理性能。

除了对单核性能的挖掘，TEngine在多线程加速上也针对Arm SoC系统构架做了专门的优化，并且通过优化调度，双线程加速比达到170%，四线程加速比达到270%。

为了满足嵌入式应用场景对低功耗、小内存的需求，TEngine在内存也做了深度优化，轻松运行极度消耗内存的VGG16网络，并且通过增加缓存利用率减少不必要的内存访问，从而降低功耗。


# TEngine开放生态

TEngine平台不仅性能指标非常优秀，而且使用非常方便。TEngine可以以wrapper的形式支持Caffe API，原本基于caffe开发的应用只需要简单的重新编译就能在TEngine上二次部署，后续TEngine还将支持Tensorflow API。

作为一款开放计算平台，TEngine的扩展能力非常强，除了支持自带的HCL库以外，还支持业界常见的Openblas, Arm Compute Library库。客户还可以很容易拓展自己的算子库，给用户自由的选择。除此之外，TEngine还能适配芯片上GPU等各类硬件资源甚至是DLA硬件加速器。只需要简单的在TEngine上注册设备并挂载上驱动，就能充分利用芯片上的硬件资源提升计算能力。

**Tengine**, developed by **OPEN** AI LAB, is a lite, high-performance, and modular inference engine for embedded device.

Tengine is composed of six modules: **core/operator/serializer/executor/driver/wrapper**.

- [**core**](core)  provides the basic components and functionalities of the system.
- [**operator**](operator)  defines the schema of operators, such as convolution, relu, pooling, etc. al. Here is the current support [operator list](doc/operator_ir.md). 
- [**serializer**](serializer)  is to load the saved model. The serializer framework is extensible to support different format, including the customized one. caffe/onnx/Tensorflow and MXNet models can be loaded directly by Tengine.
- [**executor**](executor)  implements the code to run graph and operators. Current version provides a highly optimized implementation for multi A72 cores.
- [**driver**](driver)  is the adapter of real H/W and provides service to device executor by HAL API. It is possible for single driver to create multiple devices.
- [**wrapper**](wrapper)  provides the wrapper of APIs for different frameworks. Both caffe API wrapper and Tensorflow API wrapper work now.


This version can load and run caffe/MXNet model of **mobilenet** and **squeezenet** directly.  For more details, please goto [**install**](doc/install.md).

`NOTE`: Old caffe model has to be upgraded using **upgrade_net_proto_binary/upgrade_net_proto_binary** from caffe's package.

## Performance

The data is collected on **1.8G A72** and on chip RK3399, by repeating calling the forward interface to get the average time cost (ms) per run.

- Single A72 core (1xA72)

|NN  |Caffe(Openblas)|Tengine|
|----|---------------|-------|
|squeezenet|147|91|
|mobilenet|306|122|

- Two A72 cores (2xA72)

|NN  |Caffe(Openblas)|Tengine|
|----|---------------|-------|
|squeezenet|102|51|
|mobilenet|232|65|


For details to run benchmark, please visit [**benchmark**](doc/benchmark.md) page.

## Build and Install
please refer to the [install](doc/install.md) page.


## Develop New Operator

It is easy to add new operator to Tengine. Here is the guide on [new operator](doc/operator_dev.md).

## Support New Model Format

Tengine can be extended to support new serialization format, by building new serializer module. 

[How to build new serializer module](doc/serializer_dev.md)

## Release History

### version 0.5.0 - 2018/6/15

**New features**

Support GPU: using ACL (Arm computing library) as a backend graph device

Support blas operator implementation: Tengine can run on x86 without caffe now

Support new NN: Inception-v3/vgg16/faster-rcnn/ssd/yolo-v2

Support Android build:  includes 32bit and 64bit

Support cross-compile on x86 (experimental): 
    debian example contributed by **mcharleb** and **Mani-Sadhasivam** @ Linaro

Support Tensorflow serializer: load inception-v3 and mobilenet TF model directly

Support Tensorflow wrapper: label_image.cpp from tensorflow repo

**Others**

 Single so file now and remove the etc/config according to feedback from field.
     
Tengine will automatically probe the CPU arch/part settings, and there is just one CPU driver now.

To assign cpu manually when necessary:
     
     export TENGINE_CPU_LIST=1,2 
     


Besides probing CPU, a few CPUs are defined in cpu_predefined.cpp, including rk3399/a63/kirin960/apq8096.
To use the predefined CPU, refers to below:

    const struct cpu_info * p_info=get_predefined_cpu("rk3399");
    create_cpu_device("rk3399",p_info);
    





### version 0.3.0 - 2018/2/6

Introduce the driver/device model to support MT(Multi-Thread)

Support new NN: Inception-v4

Caffe Wrapper examples: squeezenet/mobilenet/mtcnn

MXNet model load examples: squeezenet/mobilenet


### version 0.2.0 - 2018/1/24

Support new operator: Eltwise, PReLU, Slice

Support new NN: mtcnn, resnet and lighten_cnn 

Experimental caffe API wrapper: caffe based application just needs to recompile to use Tengine


### version 0.1.2 - 2017/12/30

Update documents, as well a few fixes.

### version 0.1.0 - 2017/12/29

Initial release of single A72 support




