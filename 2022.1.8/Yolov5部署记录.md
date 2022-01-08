# Yolov5部署记录

**主要包含以下部署方式**

- ncnn部署
- libtorch部署
- openvino部署
- tensorRT部署

## Export

1. 安装依赖包

   ```sh
   pip install onnx, onnxruntime 
   ```

2. 修改程序

   ```python
   # 1. 修改export.py中的--img-size为自己设定的尺寸，注意需要是32的倍数，如：
   parser.add_argument('--img-size', nargs='+', type=int, default=[960, 960], help='image size')
   
   # 2. 修改export.py中导出最后一层的设置为True
   model.model[-1].export = True  # not opt.grid  # set Detect() layer grid export
   ```

3. 执行导出命令

   ```sh
   python models/export.py --weights xx/yy.pt --batch 1 --device 0
   ```

4. 简化

   ```sh
   pip install onnx-simplifier
   
   cd weights
   python -m onnxsim yy.onnx yy-sim.onnx
   ```

   

## NCNN部署

1. 下载安装VS2017 （使用VS2015会报错，无法完成安装）

2. 下载安装protobuf

   - 从 https://github.com/google/protobuf/archive/v3.4.0.zip下载protobuf-3.4.0，解压到指定目录
   - 依次运行以下命令

   ```sh
   cd <protobuf-root-dir>
   mkdir build_vs2017
   cd build_vs2017
   
   cmake cmake -G"NMake Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=%cd%/install -Dprotobuf_BUILD_TESTS=OFF -Dprotobuf_MSVC_STATIC_RUNTIME=OFF ../cmake
   or
   cmake cmake -G"NMake Makefiles" -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=%cd%/install -Dprotobuf_BUILD_TESTS=OFF -Dprotobuf_MSVC_STATIC_RUNTIME=OFF ../cmake
   
   nmake
   nmake install
   ```

   cmake命令中的Release可以修改为Debug，根据自己需要进行选择

3. 下载安装vulkan

   - 从https://vulkan.lunarg.com/sdk/home下载最新版vulkan，安装到指定目录
   - 配置环境变量（可选），一个例子如下：
     - Vulkan_INCLUDE_DIR = C:\VulkanSDK\1.1.106.0\Include
     - Vulkan_LIBRARY = C:\VulkanSDK\1.1.106.0\Lib
     - VULKAN_SDK = C:\VulkanSDK\1.1.106.0
   - 安装vulkan_NVIDIA驱动 https://developer.nvidia.com/vulkan-driver。采用精简安装即可。

4. 下载安装ncnn

   - 依次运行以下命令（注意替换其中的路径为个人的路径）

   ```sh
   cd <ncnn-root-dir>
   git clone https://github.com/Tencent/ncnn.git
   cd ncnn
   mkdir -p build
   cd build
   
   cmake -G"NMake Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=%cd%/install -DProtobuf_INCLUDE_DIR=<protobuf-build-dir>/install/include -DProtobuf_LIBRARIES=<protobuf-build-dir>/install/lib/libprotobuf.lib -DProtobuf_PROTOC_EXECUTABLE=<protobuf-build-dir>/install/bin/protoc.exe -DNCNN_VULKAN=OFF -DOpenCV_DIR=E:/opencv/build .. 
   or
   cmake -G"NMake Makefiles" -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=%cd%/install -DProtobuf_INCLUDE_DIR=<protobuf-build-dir>/install/include -DProtobuf_LIBRARIES=<protobuf-build-dir>/install/lib/libprotobufd.lib -DProtobuf_PROTOC_EXECUTABLE=<protobuf-build-dir>/install/bin/protoc.exe -DNCNN_VULKAN=ON -DOpenCV_DIR=E:/opencv/build .. 
   
   nmake
   nmake intall
   ```

   - 若报错：

   ```sh
   CMake Error at CMakeLists.txt:223 (message):
     The submodules were not downloaded! Please update submodules with "git
     submodule update --init" and try again.
   ```

   则运行：

   ```sh
   git submodule update --init
   ```

5. onnx转ncnn

   - 依次运行以下命令：

   ```sh
   cd tools/onnx
   onnx2ncnn your-yolov5s-path/yolov5-sim.onnx  the-path-to-save/yolov5-sim.param the-path-to-save/yolov5-sim.bin
   ```

   如出现以下输出，属于正常情况：

   ```sh
   Unsupported slice step !
   Unsupported slice step !
   Unsupported slice step !
   Unsupported slice step !
   ...
   ```

   - 上述输出的原因是Split、Crop和Concat层在ncnn中未定义，我们需要删除这些层，然后自己定义实现。打开yolov5-sim.param删除这些层，过程示例如下：

   ```sh
   # 删除前
   7767517
   309 350
   Input            images                   0 1 images
   Split            splitncnn_input0         1 4 images images_splitncnn_0 images_splitncnn_1 images_splitncnn_2 images_splitncnn_3
   Crop             Slice_4                  1 1 images_splitncnn_3 227 -23309=1,0 -23310=1,2147483647 -23311=1,1
   Crop             Slice_9                  1 1 227 232 -23309=1,0 -23310=1,2147483647 -23311=1,2
   Crop             Slice_14                 1 1 images_splitncnn_2 237 -23309=1,1 -23310=1,2147483647 -23311=1,1
   Crop             Slice_19                 1 1 237 242 -23309=1,0 -23310=1,2147483647 -23311=1,2
   Crop             Slice_24                 1 1 images_splitncnn_1 247 -23309=1,0 -23310=1,2147483647 -23311=1,1
   Crop             Slice_29                 1 1 247 252 -23309=1,1 -23310=1,2147483647 -23311=1,2
   Crop             Slice_34                 1 1 images_splitncnn_0 257 -23309=1,1 -23310=1,2147483647 -23311=1,1
   Crop             Slice_39                 1 1 257 262 -23309=1,1 -23310=1,2147483647 -23311=1,2
   Concat           Concat_40                4 1 232 242 252 262 263 0=0
   Convolution      Conv_41                  1 1 263 264 0=64 1=3 11=3 2=1 12=1 3=1 13=1 4=1 14=1 15=1 16=1 5=1 6=6912
   Swish            Mul_43                   1 1 264 266
   
   # 删除后，YoloV5Focus是自定义的层，用以替代删除的层
   7767517
   300 350
   Input            images                   0 1 images
   YoloV5Focus      focus					  1 1 images 263
   Convolution      Conv_41                  1 1 263 264 0=64 1=3 11=3 2=1 12=1 3=1 13=1 4=1 14=1 15=1 16=1 5=1 6=6912
   Swish            Mul_43                   1 1 264 266
   ```

   - 修改yolov5-sim.param中的Reshape层。因为Reshape 层把输出grid数写死了，根据 ncnn Reshape 参数含义，把写死的数量改为 -1，便可以自适应。

   ```sh
   # 修改前
   Convolution      Conv_403                 1 1 567_splitncnn_0 632 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=65280
   Reshape          Reshape_417              1 1 632 650 0=6400 1=85 2=3
   Permute          Transpose_418            1 1 650 output 0=1
   Convolution      Conv_419                 1 1 599_splitncnn_0 652 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=130560
   Reshape          Reshape_433              1 1 652 670 0=1600 1=85 2=3
   Permute          Transpose_434            1 1 670 671 0=1
   Convolution      Conv_435                 1 1 631 672 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=261120
   Reshape          Reshape_449              1 1 672 690 0=400 1=85 2=3
   Permute          Transpose_450            1 1 690 691 0=1
   
   # 修改后
   Convolution      Conv_403                 1 1 567_splitncnn_0 632 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=65280
   Reshape          Reshape_417              1 1 632 650 0=-1 1=85 2=3
   Permute          Transpose_418            1 1 650 output 0=1
   Convolution      Conv_419                 1 1 599_splitncnn_0 652 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=130560
   Reshape          Reshape_433              1 1 652 670 0=-1 1=85 2=3
   Permute          Transpose_434            1 1 670 671 0=1
   Convolution      Conv_435                 1 1 631 672 0=255 1=1 11=1 2=1 12=1 3=1 13=1 4=0 14=0 15=0 16=0 5=1 6=261120
   Reshape          Reshape_449              1 1 672 690 0=-1 1=85 2=3
   Permute          Transpose_450            1 1 690 691 0=1
   ```

   注意记录Permute层的输出层数，如这里，分别是：output, 671, 691，这三层分别对应stride 8/16/32的输出。

   - 压缩（可选），用 ncnnoptimize 过一遍模型，转为 fp16 存储减小模型体积

   ```sh
   ./ncnnoptimize yolov5-sim.param yolov5-sim.bin yolov5-sim-opt.param yolov5-sim-opt.bin 65536
   ```

   - 生成头文件 

   ```sh
   # 不建议通过头文件加载模型，头文件太大了，会把电脑卡崩的
   ./ncnn2mem yolov5-sim.param yolov5-sim.bin xx.id.h xx.mem.h
   ```

   

   

6. C++调用模型预测

   - 直接从ncnn官网下载yolov5的示例https://github.com/Tencent/ncnn/blob/master/examples/yolov5.cpp。修改其中的一些内容即可运行：

   ```C++
   // 修改模型路径
   yolov5.load_param("yolov5s.param");
   yolov5.load_model("yolov5s.bin");
   
   // 修改图像尺寸
   const int target_size = 640;
   
   // 修改stride层和anchor，stride的层数是前面Permute中定义的，anchor的尺寸是根据模型的yaml文件定的，如：yolov5s.yaml
   // stride 8
   {
       ncnn::Mat out;
       ex.extract("output", out);
   
       ncnn::Mat anchors(6);
       anchors[0] = 10.f;
       anchors[1] = 13.f;
       anchors[2] = 16.f;
       anchors[3] = 30.f;
       anchors[4] = 33.f;
       anchors[5] = 23.f;
       ...
   }
   
   // stride 16
   {
       ncnn::Mat out;
       ex.extract("671", out);	// 781
   
       ncnn::Mat anchors(6);
       anchors[0] = 30.f;
       anchors[1] = 61.f;
       anchors[2] = 62.f;
       anchors[3] = 45.f;
       anchors[4] = 59.f;
       anchors[5] = 119.f;
       ...
   }
   
   // stride 32
   {
       ncnn::Mat out;
       ex.extract("691", out);	// 801
   
       ncnn::Mat anchors(6);
       anchors[0] = 116.f;
       anchors[1] = 90.f;
       anchors[2] = 156.f;
       anchors[3] = 198.f;
       anchors[4] = 373.f;
       anchors[5] = 326.f;
       ...
   }
   
   // 修改class
   static const char* class_names[] = {"person", "bicycle", "car", ...};
   ```

   - 运行

   ```sh
   ./yolov5 image-path
   ```

   

## 检测模型的评价指标(object detection)

**mAP(mean average precision) ** 

1. **IoU(Intersection over Union)**交并比 IoU是预测框与ground truth的交集和并集的比值。这个量也被称为Jaccard指数，并于20世纪初由Paul Jaccard首次提出。为了得到交集和并集，我们首先将预测框与ground truth放在一起

 ![img](https://img-blog.csdnimg.cn/20181117142046935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hpdHppaml5aW5nY2Fp,size_16,color_FFFFFF,t_70)

对于每个类，预测框和ground truth重叠的区域是交集，而横跨的总区域就是并集。其中horse类的交集和并集如下图所示（这个例子交集比较大）：

![img](https://img-blog.csdnimg.cn/20181117142128360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hpdHppaml5aW5nY2Fp,size_16,color_FFFFFF,t_70)

其中蓝绿色部分是交集，而并集还包括橘色的部分。那么，IoU可以如下计算：

![img](https://img-blog.csdnimg.cn/20181117142226610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hpdHppaml5aW5nY2Fp,size_16,color_FFFFFF,t_70)

2. **鉴别正确的检测结果并计算precision和recall **

   为了正确的计算precision和recall  我们通常需要 Ture Positives(真正例) False Positive(假正例) True Negative(真负例) False Negatives(假负例)

   

   Accuracy(Acc 准确率)：
   $$
   Acc = \frac{TP+TN}{TP+FN+FP+FN}
   $$
   

   如下图所示(假设IoU的交并比阈值为0.5),为了获取 Ture Positives(真正例) False Positive(假正例) 我们通常需要考虑IoU交并比.
   $$
   IoU=
   \begin{cases}
   \leq0.5 , False Positives(FP)\\
   >0.5, Ture Positive(TP)\\
   \end{cases}
   $$
   各个类别的Precision(又称查准率):
   $$
   Precision = \frac{TP}{TP+FP}
   $$
   计算recall(又称查全率)的时候，我们通常需要知道Negatives的数量， FalseNegatives(即我们模型所漏检的物体)。在得到Ture Positives(正确预测值数量)与False Negatives(漏检物体数)，据此可以计算出Recall:
   $$
   Recall=\frac{TP}{TP+FN}
   $$
   ![img](https://img-blog.csdnimg.cn/20200324221740538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2t1d2VpY2Fp,size_16,color_FFFFFF,t_70#pic_center)

3. **计算AP（Average Precision）**

查准率和查全率是一对矛盾的度量，一般而言，查准率高时，查全率往往偏低；而查全率高时，查准率往往偏低。我们从直观理解确实如此：我们如果希望好瓜尽可能多地选出来，则可以通过增加选瓜的数量来实现，如果将所有瓜都选上了，那么所有好瓜也必然被选上，但是这样查准率就会越低；若希望选出的瓜中好瓜的比例尽可能高，则只选最有把握的瓜，但这样难免会漏掉不少好瓜，导致查全率较低。通常只有在一些简单任务中，才可能使查全率和查准率都很高。所以为了更全面的衡量模型的性能提出了 AP。
![img](https://img-blog.csdnimg.cn/2020032421535733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2t1d2VpY2Fp,size_16,color_FFFFFF,t_70#pic_center)

AP表示的是检测器在各个Recall情况的平均值，即为PR曲线下的面积，从离散的角度如下式表示：
$$
AP=\frac{\sum{P_{ri}}}{\sum{r}}
$$
其中$$\sum p_{ri}$$表示PR曲线上r-i所对应的P值， 而$$\sum r=1$$

显然AP是针对某一个类别来说，比如马这一个单一类别。

4. mAP
   $$
   mAP=\frac{AP}{num_classes}
   $$
   





















