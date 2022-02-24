# C++ 如何编动态库与静态库



# 静态库编写

静态库的编写相对较为简单

1. 首先创建一个应用程序选择正常的静态库

   ![image-20220107180733669](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220107180733669.png)

2. 创建.cpp 与.h文件

   将需要的输出内容放入其中（可以创建多个.cpp与多个.h）,在输出静态库的过程中只要在同一个根命名空间中，就会统一输出一个lib包含所有的内容函数。

3. 在一个解决方案之下创建另外一个测试工程

   ![image-20220107182201382](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220107182201382.png)

4. 在测试工程下创建.cpp

   ```c++
   #include "Test.h"
   #include <stdio.h>
   #include "../Static_link_library2/add.h"         //注意包含.h
   #pragma comment(lib, "Static_link_library2.lib") //注意包含lib
   #include <iostream>
   #include "../Static_link_library2/one_hundred.h"
   
   int main()
   {
   
   	printf("add from one to hundred: %d", one_add_hundred());
   		
   	system("pause");
   	return 1;
   	
   }
   
   
   ```

   除了上述这种包含方式还可以直接在项目属性页中包含

   如下图所示（红色斜杠处包含lib的目录与lib名称）**注意c++文件相对路径包含系统需要使用“ ..\ ”来返回上一级 **

   ![image-20220107190906754](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220107190906754.png)



## 动态库的编写

动态库的编写要稍微复杂一些，其中调用过程分为两种方式隐式调用与显式调用

1. 创建一个应用程序向导

   ![image-20220107192641978](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220107192641978.png)



2. 创建一个.cpp与对应的.h

   其中.h的函数调用如下图模板所示

   ```c++
   
   #pragma once
   
   #include <vector>
   #include <iostream>
   #include "net.h"
   #include "postprocess.h"
   #include "pre_deal.h"
   #include "Draw.h"
   
   
   #ifndef _DLLAPI
   	#define DLLAPI _declspec(dllexport)
   #else
   	#define DLLAPI _declspec(dllexport)
   #endif
   
   
   
   extern "C" DLLAPI int detect_shufflenetv2(const cv::Mat& bgr, std::vector<float>& cls_scores, ncnn::Net *shufflenetv2);
   extern "C" DLLAPI int print_topk(const std::vector<float>& cls_scores, int topk);
   extern "C" DLLAPI ncnn::Net * load_Moudle(const char * Path_Param, const char * Path_Bin);
   extern "C" DLLAPI cv::Mat load_Pic(cv::Mat m, const char * PIC);
   extern "C" DLLAPI unsigned int correct_label(cv::Mat m);
   
   
   
   ```

   

3. 建立一个调用函数

   其中显示调用与上述静态库调用比较类似就不展开了

   其中隐式调用动态库的方式与模板如下 其中需要注意的是 构建函数指针去回调dll中的函数值

   ```c++
   #include "test.h"
   #include "../WBC-C/WBC_C.h"
   #include <iostream>
   #include <windows.h>
   //#pragma comment(lib, "WBC-C.lib") //这个不是真正的静态库， 隐式调用动态库
   
   typedef int(*SHUFFLENET)(const cv::Mat& bgr, std::vector<float>& cls_scores, ncnn::Net *shufflenetv2);
   typedef int(*PRINT_TOPK)(const std::vector<float>& cls_scores, int topk);
   typedef ncnn::Net* (*LOAD)(const char * Path_Param, const char * Path_Bin);
   typedef cv::Mat (*PIC)(cv::Mat m, const char * PIC);
   typedef unsigned int(*CORRECT)(cv::Mat m);
   
   int test()
   {
   	HMODULE hDLL = LoadLibrary(L"../x64/Release/WBC-C.dll");
   	if (hDLL == NULL)
   	{
   
   		printf("加载DLL文件失败.\n");
   		return 0;
   
   	}
   	SHUFFLENET shufflenet = (SHUFFLENET)GetProcAddress(hDLL, "detect_shufflenetv2");
   	PRINT_TOPK print_topk = (PRINT_TOPK)GetProcAddress(hDLL, "print_topk");
   	LOAD load = (LOAD)GetProcAddress(hDLL, "load_Moudle");
   	PIC pic = (PIC)GetProcAddress(hDLL, "load_Pic");
   	CORRECT correct = (CORRECT)GetProcAddress(hDLL, "correct_label");
   
   	std::vector<float> cls_scores;
   	cv::Mat dst;
   	ncnn::Net *shufflenetv2;
   
   	dst = pic(dst, "c:\\Users\\Administrator\\Pictures\\1 (5)1.jpg");
   	shufflenetv2 = load("WBC-C.param", "WBC-C.bin");
   
   	shufflenet(dst, cls_scores, shufflenetv2);
   	int label = print_topk(cls_scores, 18);
   
   	if (label == 3)
   	{
   
   		label = correct(dst);
   
   	}
   
   
   	return 1;
   
   }
   
   
   int main()
   {
   	test();
   	return 0;
   	//cout << label << endl;
   }
   ```

   当然也可以使用属性链接dll,其中需要注意的是如果程序找不到dll,可以如下操作

   1. 右键单击“解决方案资源管理器”中的“MathClient”节点，然后选择“属性”以打开“属性页”对话框
   2. 在“配置”下拉框中，选择”所有配置“(如果尚未选择)

   3. 在左窗歌中，选择”配置属性“ ”生成事件后生成事件“

   4. 在属性窗格中在”命令行“字段中选择编辑控件，如果按照指示将客户端项目置于DLL项目的单独解决方案之中则输入如下

      ```c++
      xcopy /y /d "..\..\MathLibrary\$(IntDir)MathLibrary.dll" "$(OutDir)"
      ```

      ![image-20220107194116895](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220107194116895.png)

