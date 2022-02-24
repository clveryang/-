# 如何通过开源代码编译labelimg

## 1 下载labelimg源码

GitHub： https://github.com/tzutalin/labelImg

## 2 在pycharm打开并选择需要更改的内容

在此可以增加特定类别的颜色

![image-20220218194853632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220218194853632.png)

在此可以修改选定框以及覆盖的颜色

![image-20220218195018324](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220218195018324.png)

## 3通过pyinstaller 编译python源码生成  labelimg.exe

1. 通过以下命令在labelimg源码路径下编译 **注意！ 注意！一定要用cmd或者powershell，千万不要使用conda虚拟环境编译源码**

```python
pip install PyQt5

pip install pyinstaller

			-D #生成一个文件夹   -c #不带控制台 
pyinstaller -F #生成一个exe     -w #带win控制台    labelimg.py

```

2. 通过上一步的命令生成 build 与dist 以及labelimg.spec

3. 删除build与dist，并按照如下修改dist

   ![image-20220218200716483](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220218200716483.png)

4. 运行命令 pyinstaller -F labelimg.spec （为了生成的labelimg.exe可以关联predefined_classes.txt）

## 4然后就可以正常打开与使用我们自己编译的labelimg了









