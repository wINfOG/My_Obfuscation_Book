# 反混淆：花指令/间接跳转



花指令或者中间跳转是常见的基础混淆。虽然强度不高，但是其实这是一个很不错的混淆技术：没什么性能损耗，能直接干扰F5反编译（外行一看就感觉有效果），甚至可以在源码层实现不用动中间语言。

常见的混淆场景有以下三种：

1. 最基础也是最常见的，函数头部的花指令或者间接跳转，一般来说他们成片的出现。
2. 不太常见，间接跳转在函数体中间，是上面的加强版，通常和上面一起出现
3. 最少见的，混淆将会把函数中几乎所有的直接跳转语句变成间接跳转，这种情况已经接近深度混淆了，但是由于“跳转”的局限性，只要能够找到方法处理控制流，就能解决。



## 场景1. 函数头部的花指令

最常见的，就是在IDA中按F5后，变成了下面这样

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

他的代码大概会是下面这样个逻辑，有的时候IDA的常量传播优化能够直接计算出目标地址。

```
func sub_sample() {
    //一大堆对reg_xx的计算。。。
    br reg_xx
    //可能会出现的干扰垃圾指令
    //正在的函数起始部分
}
```

对于间接跳转，最有效的两个通用对抗方案是：

1. 通过观察规律找到BR指令前的汇编计算模式，在脚本中计算真实跳转后替换指令。
2. 混合模拟执行器，直接模拟执行出目标地址。

我不太推荐第一种观察规律的方案，对于静态还原观察规律的脚本很难再次复用，而且现在稍微复杂一些的混淆模式不太能观察出有效的规律。

第二种，以看我的项目中的IDA反混淆sample，依赖flare\_emu（本质上也是Unicorn）

> [https://github.com/wINfOG/IDA\_Easy\_Life/blob/main/emu\_fast\_br.py](https://github.com/wINfOG/IDA_Easy_Life/blob/main/emu_fast_br.py)





## 场景2. 函数中间的中间跳转

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



可以看出IDA能够分析出跳转的目标。

最主要的问题是不是很好模拟的开头，如果以basic-block作为开头，业务逻辑代码混合在中间跳转的逻辑中；缺少了上下文构造的环境，比较容易出现内存异常，通用性难做的好。



## 场景3. 完全的中间跳转函数



将所有的分支跳转都替换为间接跳转语句，分支语句。



## 其它场景
