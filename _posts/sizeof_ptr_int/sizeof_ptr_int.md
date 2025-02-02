# 不同平台下int类型、指针类型的数据大小
对于int类型数据和指针类型数据的大小，是非常基础的问题。  

在一个具体的平台上，确定他们最好的办法就是使用sizeof(type)对其进行判断，返回当前数据类型的大小。  

在不同的平台下，int类型和指针类型的数据类型大小时怎样的呢？如果要给出一个统一的答案，自然不可能集齐每个平台，一个个地去试，我们必须从底层进行分析。  


## 数据总线和地址总线
计算机内的数据总线是CPU与外设进行数据交换的通路，而地址总线则是CPU用于寻址的通路。  

数据总线的位数决定了CPU与外设一次可以传输的字节数。  

而地址总线则决定了CPU可以寻址的范围，如果是32位地址线，每根地址线传输一位，可表示的范围为0~2^32-1，由于内存中的基本单位是byte，所以也就对应0~2^32-1 byte，也就是0~4GB，对应于内存的可取范围。  

总线的位数一般也用总线宽度来表示。  

*** 
一个常见的误区是：地址总线和数据总线的宽度总是保持一致的。  

这个错误想法的缘由就是没有弄清楚地址总线和数据总线的工作原理，我们可以这样理解：

在一个很大的仓库中，地址总线代表一张仓库地图的坐标，而数据总线则是取货的推车，地图坐标系的大小(也就是地址总线的宽度)决定了它可以记录多大的仓库面积，而推车的大小(即数据总线的宽度)决定了一次可以取多少货。  

仓库管理员根据地图找到货物存放的地址，然后使用推车来存取货物。  

在这个模型中，仓库的面积可以非常大，只需要将地图坐标绘得足够大，而取货的推车可以很小，只是取大件货物的时候多跑几次。  

同样的，管理员可以买个足够大的推车，而仓库面积并不大，只是通常在取货的时候，推车不能被装满，存在一些空间浪费问题。

*** 
所以说，地址总线和数据总线在宽度上没有太大的联系，但是，芯片设计者为了平衡时间效率和空间效率，同时考虑到硬件上的影响，通常使用同样的地址总线宽度和数据总线宽度。  

举一个数据总线和地址总线宽度不同的例子：在经典的8086计算机中，数据总线为16位，地址总线则是20位。  

## 芯片的位数
通常，我们所说的一个芯片是多少位的，到底是看它的数据总线宽度还是地址总线宽度呢？  

答案是：决定一个芯片多少位，是由这个芯片一次能处理多少位数据决定的，等于片内寄存器的宽度，同时可以看成是数据总线的宽度。  

为什么这里说的是芯片位数可以看成是数据总线的宽度而不是等于数据总线的宽度呢？按照目前的情况而言，几乎所有的芯片位数都等于数据总线宽度。

但是从严格意义上来说，数据总线是用来传输数据的，芯片位数指的是处理数据的宽度，数据的传输和处理并非同一个概念，传输和处理数据的宽度是可以不一样的，只是实际情况下数据传输的宽度和处理的宽度是一样的，当然，这种区分有点吹毛求疵，所以，芯片位数等于数据总线宽度也是一种可接受的答案(当然，过去都是一致不代表未来也是一致，概念还是要分清)。  

同时，在编程时，我们通常碰到一个叫做"字长"的概念，字长通常等于数据总线的宽度。

## int型数据的大小
常见的第二个误区是：int型数据的大小，也就是sizeof(int)的大小完全跟随硬件平台的位数。  

这个误区的产生是源于我们初学C语言时的教程：在16位芯片上int型类型大小为16位，即两字节，而在32位机器上，int型为32位，即四字节。 以此类推，由此我们就建立的一个模糊且错误的概念：int型数据的大小是跟随于平台的位数。  

事实上，正确的答案是：int型数据的大小和硬件平台位数无关，它是由C语言标准和编译器共同决定的。  

为此，博主查阅了C99 spec标准，它是这么说的：

    Sizes of integer types <limits.h>
    The values given below shall be replaced by constant expressions suitable for use in #ifpreprocessing directives.
    ...
    ...
    minimum value for an object of type int
    INT_MIN -32767 // −(215 − 1)           //这只是其中一个示例，不同平台可能有不同定义
    — maximum value for an object of type int
    INT_MAX +32767 // 215 − 1
翻译过来就是，int类型的大小是由limits.h文件中INT_MIN和INT_MAX两个宏定义来决定的，而limits.h文件在编译器库文件中可以找到。 

int类型对应平台的大小是这样的：
* 16位系统中，int型为16位大小，两字节
* 32位系统中，int型为32位大小，四字节
* 64位系统中，int型为32位大小，四字节

事实上，除了int类型，还有一个类型在不同平台中有不同的表现，那就是long型:
* 16位系统中，long型为32位大小，4字节
* 32位系统中，long型为32位大小，4字节
* 64位系统中，long型为64位大小，8字节

## 指针的大小
对于指针变量的大小，我想听得最多的一个概念就是：在32位系统下指针类型为32位，在64位系统下指针类型为64位。  

但是不得不遗憾地说，这个说法其实是错误的，至少说是不严谨的。  

指针本质上是变量，它的值是内存中的地址，既然需要通过指针能够访问当内存当中所有的数据，那么这个指针的类型至少要大于等于地址总线的宽度。打个比方一个芯片的地址总线是32位，那么内存地址的范围就是0~4G，那么这个指针类型的宽度至少需要32位，才能保证访问到内存中每个字节。  

但是，实际上的情况是：芯片的位数由芯片一次能处理的数据宽度决定，可看成是数据总线的宽度，但是地址总线和数据总线的宽度有时候并不一致。  

所以在经典的32位系统中，同时也是32位地址总线，自然而然的，指针的长度为32位。  

但是对早期的8086而言，这是16位芯片，但是它的地址总线却扩展到了20位，同时因为数据对齐的原因，它的指针大小应该是16+16位=32位，但是出于效率上的优化，8086提供了远指针、近指针，在访问本段内的地址时，采用16位指针，如果有段地址跳转，就使用32位的指针。 

至少从这个示例可以知道，指针的大小完全由实际使用的地址总线的宽度(+数据对齐)来决定，而并非由芯片位数来决定。  

所以，有时候，我们可能会在64位系统中碰到指针大小为4字节的情况，也可能在16位系统中碰到指针大小为4字节的情况。  

***当然，需要特别注意的是，在64位系统中地址大小为4字节的情况下，并非一定是芯片的地址总线是32位，很可能是CPU运行在只使用部分地址总线的模式下，又或者是使用32位兼容的编译器所致，这一部分较为复杂，暂不赘述。***


## 总结
int和long类型数据大小并非由硬件平台的位数决定，而是由C标准和编译器共同决定。  

同时，指针即sizeof(ptr)的大小也并非由硬件平台的位数决定，而是由实际上所使用的地址总线宽度决定的。  

好了，关于int型和指针的大小的讨论就到此为止啦，如果朋友们对于这个有什么疑问或者发现有文章中有什么错误，欢迎留言

***原创博客，转载请注明出处！***

祝各位早日实现项目丛中过，bug不沾身.


