# 寒武纪 NPU 部署示例

Paddle Lite 已支持寒武纪思元 NPU （MLU370）在 X86 服务器上进行预测部署。目前支持子图接入方式，其接入原理是在线分析 Paddle 模型，将 Paddle 算子先转为统一的 NNAdapter 标准算子，再通过 Cambricon MagicMind 组网 API 进行网络构建，在线生成并执行模型。

## 支持现状

### 已支持的芯片

- MLU 370

### 已支持的设备 ？ 

- Atlas 300I 推理卡（型号：3000/3010)
（待补充）

### 已支持的 Paddle 模型

#### 模型
- 图像分类
  - [ResNet-50](https://paddlelite-demo.bj.bcebos.com/NNAdapter/models/PaddleClas/ResNet50.tgz)
- 目标检测

#### 性能

- 测试环境
  - 编译环境
    - Ubuntu 16.04, GCC 5.4 

  - 硬件环境
    - CPU: Intel(R) Xeon(R) Gold 6230 CPU @ 2.10GHz
    - NPU: MLU370-S4

- 测试结果

  性能表格待补充
  

### 已支持（或部分支持）的 Paddle 算子

您可以查阅[ NNAdapter 算子支持列表](https://github.com/PaddlePaddle/Paddle-Lite/blob/develop/lite/kernels/nnadapter/converter/all.h)获得各算子在不同新硬件上的最新支持信息。

## 参考示例演示

测试设备 （xxxx 待添加）

<img 图片待补充>

### 准备设备环境

安装驱动的步骤。。。。。
- Atlas 300I 推理卡[规格说明书](https://e.huawei.com/cn/products/cloud-computing-dc/atlas/atlas-300-ai)

- 安装Atlas 300I 推理卡的驱动和固件包（Driver 和 Firmware)

- 驱动和固件包下载：https://www.hiascend.com/hardware/firmware-drivers?tag=commercial

  - 驱动：A300-3010-npu-driver_21.0.1_ubuntu18.04-x86_64.run（x86）

  - 固件：A300-3000-3010-npu-firmware_1.77.22.6.220.run

- 安装驱动和固件包：

```shell
# 增加可执行权限
$ chmod +x *.run
# 安装驱动和固件包
$ ./A300-3010-npu-driver_21.0.1_ubuntu18.04-x86_64.run --full
$ ./A300-3000-3010-npu-firmware_1.77.22.6.220.run --full
# 重启服务器
$ reboot
# 查看驱动信息，确认安装成功
$ npu-smi info
```

- 更多系统和详细信息见[昇腾硬件产品文档](https://www.hiascend.com/document?tag=hardware)

### 准备本地编译环境

- 为了保证编译环境一致，建议参考[ Docker 环境准备](../source_compile/docker_environment)中的 Docker 开发环境进行配置；

### 运行图像分类示例程序

- 下载示例程序[ PaddleLite-generic-demo.tar.gz ](https://paddlelite-demo.bj.bcebos.com/devices/generic/PaddleLite-generic-demo.tar.gz)，解压后清单如下：

  ```shell
    - PaddleLite-generic-demo
      - image_classification_demo
        - assets
          - images
            - tabby_cat.jpg # 测试图片
            - tabby_cat.raw # 经过 convert_to_raw_image.py 处理后的 RGB Raw 图像
          - labels
            - synset_words.txt # 1000 分类 label 文件
          - models
            - resnet50_fp32_224 # Paddle non-combined 格式的 resnet50 float32 模型
              - __model__ # Paddle fluid 模型组网文件，可拖入 https://lutzroeder.github.io/netron/ 进行可视化显示网络结构
              - bn2a_branch1_mean # Paddle fluid 模型参数文件
              - bn2a_branch1_scale
              ...
        - shell
          - CMakeLists.txt # 示例程序 CMake 脚本
          - build.linux.amd64 # 已编译好的，适用于 amd64
            - image_classification_demo # 已编译好的，适用于 amd64 的示例程序
          - build.linux.arm64 # 已编译好的，适用于 arm64
            - image_classification_demo # 已编译好的，适用于 arm64 的示例程序
            ...
          ...
          - image_classification_demo.cc # 示例程序源码
          - build.sh # 示例程序编译脚本
          - run.sh # 示例程序本地运行脚本
          - run_with_ssh.sh # 示例程序 ssh 运行脚本
          - run_with_adb.sh # 示例程序 adb 运行脚本
      - libs
        - PaddleLite
          - android
            - arm64-v8a
            - armeabi-v7a
          - linux
            - amd64
              - include # Paddle Lite 头文件
              - lib # Paddle Lite 库文件
                - cambricon_mlu # 寒武纪思元 NPU DDK、NNAdapter 运行时库、device HAL 库
                	- libnnadapter.so # NNAdapter 运行时库
                	- libcambricon_mlu.so # NNAdapter device HAL 库
                        - libmagicmind.so # 寒武纪 DDK
                        - libmagicmind_runtime.so # 寒武纪 DDK
                        - libLLVM-12.so # 寒武纪 DDK
                        - libMLIR.so # 寒武纪 DDK
                        - libcnrtc.so # 寒武纪 DDK
                        - libcndrv.so # 寒武纪 DDK
                        - libcnrt.so # 寒武纪 DDK
                        - libcnlight.so # 寒武纪 DDK
                        - libcnpapi.so # 寒武纪 DDK
                - libiomp5.so # Intel OpenMP 库
                - libmklml_intel.so # Intel MKL 库
                - libmklml_gnu.so # GNU MKL 库
                - libpaddle_full_api_shared.so # 预编译 Paddle Lite full api 库
                - libpaddle_light_api_shared.so # 预编译 Paddle Lite light api 库
            - arm64
              - include
              - lib
            - armhf
            	...
        - OpenCV # OpenCV 预编译库
      - ssd_detection_demo # 基于 ssd 的目标检测示例程序
  ```

- 进入 `PaddleLite-generic-demo/image_classification_demo/shell/`；

- 执行以下命令比较 resnet50_fp32_224 模型的性能和结果；

  ```shell
  运行 resnet50_fp32_224 模型
  	
  For amd64
  (intel x86 cpu only)
  $ ./run.sh resnet50_fp32_224 linux amd64
      warmup: 1 repeat: 5, average: 109.829399 ms, max: 124.794998 ms, min: 103.552002 ms
      results: 3
      Top0  tabby, tabby cat - 0.739717
      Top1  tiger cat - 0.130993
      Top2  Egyptian cat - 0.101076
      Preprocess time: 0.991000 ms
      Prediction time: 109.829399 ms
      Postprocess time: 0.097000 ms
  (intel x86 cpu + cambricon mlu)
  $ ./run.sh resnet50_fp32_224 linux amd64 cambricon_mlu
      warmup: 1 repeat: 5, average: 5.749600 ms, max: 5.821000 ms, min: 5.711000 ms
      results: 3
      Top0  tabby, tabby cat - 0.739199
      Top1  tiger cat - 0.130901
      Top2  Egyptian cat - 0.101006
      Preprocess time: 0.751000 ms
      Prediction time: 5.749600 ms
      Postprocess time: 0.294000 ms

### 更新支持寒武纪思元 NPU 的 Paddle Lite 库

- 下载 Paddle Lite 源码

  ```shell
  $ git clone https://github.com/PaddlePaddle/Paddle-Lite.git
  $ cd Paddle-Lite
  $ git checkout <release-version-tag>
  ```

- 编译并生成 PaddleLite+NNAdapter+CambriconMLU for amd64 的部署库

  - For amd64
    - full_publish 编译
      ```shell
      $ ./lite/tools/build_linux.sh --arch=x86 --with_extra=ON --with_log=ON --with_exception=ON --with_nnadapter=ON --nnadapter_with_cambricon_mlu=ON --nnadapter_cambricon_mlu_sdk_root=/usr/local/neuware/ full_publish
      ```

    - 替换头文件和库
      ```shell
      # 清理原有 include 目录
      $ rm -rf PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/include/
      # 替换 include 目录
      $ cp -rf build.lite.linux.x86.gcc/inference_lite_lib/cxx/include/ PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/include/
      # 替换 NNAdapter 运行时库
      $ cp build.lite.linux.x86.gcc/inference_lite_lib/cxx/lib/libnnadapter.so PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/lib/cambricon_mlu/
      # 替换 NNAdapter device HAL 库
      $ cp build.lite.linux.x86.gcc/inference_lite_lib/cxx/lib/libcambricon_mlu.so PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/lib/cambricon_mlu/
      # 替换 libpaddle_full_api_shared.so
      $ cp build.lite.linux.x86.gcc/inference_lite_lib/cxx/lib/libpaddle_full_api_shared.so PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/lib/
      # 替换 libpaddle_light_api_shared.so
      $ cp build.lite.linux.x86.gcc/inference_lite_lib/cxx/lib/libpaddle_light_api_shared.so PaddleLite-generic-demo/libs/PaddleLite/linux/amd64/lib/
      ```

- 替换头文件后需要重新编译示例程序

## 其他说明

- 我们正在持续扩展算子和模型。
