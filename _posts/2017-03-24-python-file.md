---
layout: post
title:  "文件操作"
categories: python
tags: python 基础知识
---

* content
{:toc}
文件读写的一些基础知识



## 读写文件
* **文件读写方式**
	* read([size])

		读取文件(读取size个字节，默认读取全部)
    * readlines([size])

		读取一行
    * readlines([size])

		读取完文件,返回每一行所组成的列表
        
    * write(str)

		把字符串   写入文件
        
    * writelines([sequence_of_strings])

		写入一行文件, [sequence_of_strings]是有一个可迭代的字符串参数, 如字符串, list, dict, tuple,元素必须是字符串型的
        
* **文件写是有写缓存的**
	* Buff

		通过io.DEFAULT_BUFFER_SIZE可以知道默认的缓冲区的大小
        
* **文件是有文件指针的**
	* io.SEEK_CUR
		相对于文件的当前位置
        
	* io.SEEK_SET
相对于文件的起始位置
        
	* io.SEEK_END
相对于文件结尾位置
        
	* file.tell()
		返回当前文件在哪个位置
        

## 读文件操作
```python
f = open("test.txt") #如果没指定模式，那么默认就是r模式
f.read() #把文件的内容读取进内存里面,然后返回整个文件的内容，如果文件的内容特别特别大，则会报内存不足
f.read() #再读，已经没有内容了,因为上面的操作读完后，文件指针已经指向了末尾    
f.close() #关闭文件

f = open("test.txt") #打开文件，然后把指针指向开始处
f.read(1) #读取一个字节，文件指针指向下一个字符
f.read(2) # 从文件指针的位置开始再次读取
f.close()

f = open("test.txt") # 这个文件有1G
for content in iter(f) : #一次读完所有到内存是会报异常的，那么通过迭代器则可以完整读完
	print content 
    
f.close()

f = open("test.txt") 
f.readline() #读取一行

# 当设置了读取大小就要注意了
# 如果每行的长度要大于设置的size，那么只返回指定size的内容
# 如果每行的长度要小于size,那么就返回一整行
f.readline(100) #size比行的长度要大，那么返回一整行
f.readline(1) # size比行长度要小, 只返回1个字符
f.close()

f = open("test.txt") #这个文件有1G
l = f.readlines()  # 这样会报内存错误,文件太大
l = f.readlines(1) # 设置读取一行，但是默认会读取io.DEFAULT_BUFFER_SIZE相近的一个数值

```

## 写文件操作
写文件里面存在一个缓存机制
写操作是这么个过程
**open("test", "w") -> python解释器-> 系统内核-> 缓存区 ->磁盘**
```python
f = open("test.txt", "w")
f.write("123456") # 此时已经把内容写入缓存区了，但是是还没有在文件上面能看见的
f.close() # 调用了close或者flush的时候才会把缓存区的内容写入到文件里面去


f = open("test.txt", "w")
for i in range(10000) :
	f.write('write ' + str(i) + '\n') 

#这里没有调用f.close(), 然后你去查看文件的时候，已经有内容了，但是可以看到内容并没有完全写入，这是因为如果写入的内容要比缓冲区要大的话，那么当写满了缓冲区之后，系统会把缓冲区的内容同步到磁盘，然后重新接受新的内容写入缓冲区

#此时再close,那么剩下的内容就会同步到磁盘了
f.close() 
```


## 文件的限制
**1.linux系统中每个文件打开的个数是有限制的**

**2.如果打开文件的个数达到了限制，再次打开就会报错了**





查看进程的打开文件限制


打开两个ssh, 一个输入python, 另外一个查看这个python的pid
```shell
ps aux | grep python
root      1515  0.0  0.2 121784  4244 pts/0    S+   00:57   0:00 python
root      1536  0.0  0.0 103312   872 pts/1    S+   00:57   0:00 grep python

cat /proc/1515/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            10485760             unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             7413                 7413                 processes
Max open files            1024                 4096                 files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       7413                 7413                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us

#这个就代表最大的打开限制，软限制为1024, 硬限制为4096
Max open files            1024                 4096                 files
```

实验代码
```python
files = []
for i in range(1025) :
	files.append(open("test.txt", "w"))
	print "%d : %d" % (i, files[i].fileno()) 

# 上面的打印结果表明当到了1023的时候，就会报系统错误, IOError: [Errno 24] Too many open files: 'test.txt', 因为限制是1024个文件可以被打开, 如果想打开更多的文件，可以通过unlimit -n 2000来设置, 所以打开文件就必须关闭
```


## 文件的指针

实验
```python
f = open("test.txt", "r+")
f.write("0123456789abcdef")
f.close()


f = open("test.txt", "r+")
f.tell() # 当前为第一个位置
f.seek(os.SEEK_CUR, os.SEEK_END) # 从当前的位置移动到最后
f.tell() # 17L
f.seek(os.SEEK_END, os.SEEK_SET) # 从最后的位置移动到开始的位置
f.seek(-5, os.SEEK_END) # 从最后的位置移向前移动5个位置
```

## 常用文件的属性
* 文件的描述符
	file.fileno()
    
* 文件的mode
	file.mode
* 文件的编码
	file.encoding
    

实验标准输入输出错误
```python
f = open("test.txt")
f.fileno() # 文件描述符
file.mode # 文件读写方式
file.encoding # 返回空白就是默认的asii吗
file.closed # 查看是否已经关闭了

# 标准输入输入出与文件错误
# sys.sysin 输入
# sys.sysout 输出
# sys.syserr 错误

type(sys.sysin) # 可以看到这是个文件对象
sys.sysin.fileno() # 0 代表打开了一个输入文件
a = raw_input(':') # 实际上就是调用了sys.sysin.read()

type(sys.sysout) # 也是个文件对象
sys.sysout.fileno() # 1 ,标识打开了一个输入和一个输出的文件
print '1234' # 实际上就是调用sys.sysout.write("1234") 的方法

type(sys.syserr) # 也是个文件
sys.syserr.fileno() # 2, 打开了一个输入，输出和error的文件
```

实验文件编码
```python
f = open("test.txt", "w")
f.write(u"中国") #由于 test.txt的是一个asii码的文件，这里写入的是一个unicode文件，所以报错
f.close()

#转换编码
f = open("test.txt", "w")
content = unicode.encoding(u"中国", 'utf-8')
f.write(content)
#f.write(u"中国".encode('utf-8')) 这样也是可以的
f.close()

#直接新建一个utf-8的文件
import codecs
f = codecs.open("d:/test.txt", "w", "utf-8")
f.write(u"中国")
f.close()
```


## os模块常用方法， os是使用底层的方法来操作的
* os.open
	打开一个文件，返回一个文件描述符(fd)
    
* os.write
	写入内容到文件，根据描述符
    
* os.read
	读取文件
    
* os.close
	关闭文件
    
* os.access
	查看文件是否可读，可写，可执行
    
* os.remove
	删除文件
    
* os.removedirs
	删除目录树
    
* os.mkdir
	删除目录
    
* os.mkdirs
	建立多个数目录
    
* os.rename
	重命名
* os.rmdir
	删除目录

## os.path模块常用方法
* os.path.exists
	当前路径是否存在
* os.path.isdir
	是否是一个目录
* os.path.isfile
	是否是一个文件
* os.path.getsize
	返回文件大小
* os.path.dirname
	返回路径的目录
* os.path.basename
	返回路径文件名