# python opencv



## 图像读取、显示、与保存



### 图像的读取共有三种 matplotlib(recommend)、PIL、opencv

#### 1.matplotlib

```sh
# 1、显示图片
import matplotlib.pyplot as plt #plt 用于显示图片
import matplotlib.image as mpimg #mpimg 用于读取图片
import numpy as np
lena = mpimg.imread('lena.png') #读取和代码处于同一目录下的lena.png
# 此时 lena 就已经是一个 np.array 了，可以对它进行任意处理
lena.shape #(512, 512, 3)
plt.imshow(lena) # 显示图片
plt.axis('off') # 不显示坐标轴
plt.show()

# 2、显示图片的第一个通道
lena_1 = lena[:,:,0]
plt.imshow('lena_1')
plt.show()

#3、将 RGB 转为灰度图
def rgb2gray(rgb):
    return np.dot(rgb[...,:3], [0.299, 0.587, 0.114])
    
#4、对图像进行放缩
from scipy import misc
lena_new_sz = misc.imresize(lena, 0.5) # 第二个参数如果是整数，则为百分比，如果是tuple，则为输出图像的尺寸
plt.imshow(lena_new_sz)
plt.axis('off')
plt.show()

#5、保存 matplotlib 画出的图像
plt.savefig('lena_new_sz.png')
```



#### 2.PIL

```sh
#1、显示图片
from PIL import Image
im = Image.open('lena.png')
im.show()

#2、将 PIL Image 图片转换为 numpy 数组
im_array = np.array(im)
# 也可以用 np.asarray(im) 区别是 np.array() 是深拷贝，np.asarray() 是浅拷贝

#3、保存 PIL 图片
#直接调用 Image 类的 save 方法

#4、将 numpy 数组转换为 PIL 图片
#这里采用 matplotlib.image 读入图片数组，注意这里读入的数组是 float32 型的，范围是 0-1，而 PIL.Image 数据是 uinit8 型的，范围是0-255，所以要进行转换：
import matplotlib.image as mpimg
from PIL import Image
lena = mpimg.imread('lena.png') # 这里读入的数据是 float32 型的，范围是0-1
im = Image.fromarray(np.uinit8(lena*255))
im.show()

#5、RGB 转换为灰度图、二值化图
from PIL import Image
I = Image.open('lena.png')
I.show()
L = I.convert('L')   #转化为灰度图
L = I.convert('1')   #转化为二值化图
L.show()

```



#### 3. opencv 读入的时候一般是BGR plt.imshow plt.show的时候为RGB

```python
import cv2
image = cv2.imread('C:/Users/Administrator/Pictures/1 (192)0.jpg',flags=1)

# cv.imread()函数中共有两个参数 path 与 flag
# path 中无中文字符 只用用 \\ 或 / 去分隔地址
# flags = 1 代表使用 CV_LOAD_IMAGE_UNCHANGED 保持原始格式的方式读取图像
# flags = 2 代表使用 CV_LOAD_IMAGE_GRAYSCALE 以灰度图像格式
# flags = 3 代表使用 CV_LOAD_IMAGE_COLOR 即用BGR格式读取图像， 无论原图格式都转换为BGR三通道图像

```

<font color= Red>！！注意</font>使用 **imread** 函数的返回值通道为 **B G R** 三色通道  使用cv2.imshow() 时并没有什么问题，但使用matplotlib时会出现通道参数不一致的问题。



## 图像显示cv.imshow()函数

``` python
import cv2
image = cv2.imread('C:/Users/Administrator/Pictures/1 (192)0.jpg',flags=1)
cv2.imshow('Example', image)
cv2.waitkey(0)
```

<font color=red>!!注意</font>使用cv.imread()返回值为B G R三通道顺序



## 图像保存cv2.imwrite()函数

```python
import cv2
image = cv2.imread('C:/Users/Administrator/Pictures/1 (192)0.jpg',flags=1)
image = np.asarray(img).astype(float)
cv2.imwrite('C:/Users/Administrator/Pictures/1 (192)0.jpg',img)

# cv2.imwrite()函数中的第一个形参是期望存储地址, 第二个值为图片格式



```

















## os.path.join()函数的使用

```python
import os 

path1 = 'my'
path2 = 'hometown'
path3 = 'is'
path4 = 'in'
path5 = 'xuzhou'

Path = os.path.join(path1, path2, path3, path4, path5)


```

**输出结果** : my\\hometown\\is\\in\\xuzhou



值得注意的是如果最后一个输入为空''则最后一个输入后会补上一个\\

```python

path1 = 'my'
path2 = 'hometown'
path3 = 'is'
path4 = ''

Path = os.path.join(path1, path2, path3, path4)

```

**输出结果** : my\\hometown\\is\\in\\





