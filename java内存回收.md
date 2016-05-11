下面我从一段代码来分析整个过程，并结合模型图来简易讲解，希望能让大家对彻底明白。

在正式回答这个问题之前，先简单说说 Java运行时内存区，划分为线程私有区和线程共享区：
•	（1）线程私有区：
o	程序计数器，记录正在执行的虚拟机字节码的地址；
o	虚拟机栈：方法执行的内存区，每个方法执行时会在虚拟机栈中创建栈帧；
o	本地方法栈：虚拟机的Native方法执行的内存区；
•	（2）线程共享区：
o	Java堆：对象分配内存的区域，这是垃圾回收的主战场；
o	方法区：存放类信息、常量、静态变量、编译器编译后的代码等数据，另外还有一个常量池。当然垃圾回收也会在这个区域工作。
有兴趣进一步了解的，可查看我的博文 Jvm内存模型

---------------------------------------------分割线 进入正题----------------------------------------------------

目前虚拟机基本都是采用可达性算法，为什么不采用引用计数算法呢？下面就说说引用计数法是如何统计所有对象的引用计数的，再对比分析可达性算法是如何解决引用技术算法的不足。先简单说说这两个算法：
•	引用计数算法（reference-counting） ：每个对象有一个引用计数器，当对象被引用一次则计数器加1，当对象引用失效一次则计数器减1，对于计数器为0的对象意味着是垃圾对象，可以被GC回收。
•	可达性算法(GC Roots Tracing)：从GC Roots作为起点开始搜索，那么整个连通图中的对象便都是活对象，对于GC Roots无法到达的对象便成了垃圾回收的对象，随时可被GC回收。
采用引用计数算法的系统只需在每个实例对象创建之初，通过计数器来记录所有的引用次数即可。而可达性算法，则需要再次GC时，遍历整个GC根节点来判断是否回收。

下面通过一段代码来对比说明：
 public class GcDemo {

    public static void main(String[] args) {
        //分为6个步骤
        GcObject obj1 = new GcObject(); //Step 1
        GcObject obj2 = new GcObject(); //Step 2

        obj1.instance = obj2; //Step 3
        obj2.instance = obj1; //Step 4

        obj1 = null; //Step 5
        obj2 = null; //Step 6
    }
}

class GcObject{
    public Object instance = null;
}

很多文章以及Java虚拟机相关的书籍，都会告诉你如果采用引用计数算法，上述代码中obj1和obj2指向的对象已经不可能再被访问，彼此互相引用对方导致引用计数都不为0，最终无法被GC回收，而可达性算法能解决这个问题。

但这些文章和书籍并没有真正从内存角度来阐述这个过程是如何统计的，很多时候大家都在相互借鉴、翻译，却也都没有明白。或者干脆装作讲明白，或者假定读者依然明白。 其实很多人并不明白为什么引用计数法不为0，引用计数到底是如何维护所有对象引用的，可达性是如何可达的？ 接下来结合实例，从Java内存模型以及数学的图论知识角度来说明，希望能让大家彻底明白该过程。

情况（一）：引用计数算法
如果采用的是引用计数算法：
<img src="https://pic1.zhimg.com/b9e5ecd5d1cfb1d045541c1f6e25e92c_b.png" data-rawwidth="592" data-rawheight="438" class="origin_image zh-lightbox-thumb" width="592" data-original="https://pic1.zhimg.com/b9e5ecd5d1cfb1d045541c1f6e25e92c_r.png"> 

再回到前面代码GcDemo的main方法共分为6个步骤：
•	Step1：GcObject实例1的引用计数加1，实例1的引用计数=1；
•	Step2：GcObject实例2的引用计数加1，实例2的引用计数=1；
•	Step3：GcObject实例2的引用计数再加1，实例2的引用计数=2；
•	Step4：GcObject实例1的引用计数再加1，实例1的引用计数=2；
执行到Step 4，则GcObject实例1和实例2的引用计数都等于2。

接下来继续结果图：
<img src="https://pic2.zhimg.com/3c1246661a548f933cd886070f6c1af1_b.png" data-rawwidth="592" data-rawheight="438" class="origin_image zh-lightbox-thumb" width="592" data-original="https://pic2.zhimg.com/3c1246661a548f933cd886070f6c1af1_r.png"> 

•	Step5：栈帧中obj1不再指向Java堆，GcObject实例1的引用计数减1，结果为1；
•	Step6：栈帧中obj2不再指向Java堆，GcObject实例2的引用计数减1，结果为1。
到此，发现GcObject实例1和实例2的计数引用都不为0，那么如果采用的引用计数算法的话，那么这两个实例所占的内存将得不到释放，这便产生了内存泄露。

情况（二）：可达性算法
这是目前主流的虚拟机都是采用GC Roots Tracing算法，比如Sun的Hotspot虚拟机便是采用该算法。 该算法的核心算法是从GC Roots对象作为起始点，利用数学中图论知识，图中可达对象便是存活对象，而不可达对象则是需要回收的垃圾内存。这里涉及两个概念，一是GC Roots，一是可达性。

那么可以作为GC Roots的对象（见下图）：
•	虚拟机栈的栈帧的局部变量表所引用的对象；
•	本地方法栈的JNI所引用的对象；
•	方法区的静态变量和常量所引用的对象；
关于可达性的对象，便是能与GC Roots构成连通图的对象，如下图：
<img src="https://pic3.zhimg.com/22f72b18415405c3e0207925a8de74fa_b.png" data-rawwidth="592" data-rawheight="438" class="origin_image zh-lightbox-thumb" width="592" data-original="https://pic3.zhimg.com/22f72b18415405c3e0207925a8de74fa_r.png"> 

从上图，reference1、reference2、reference3都是GC Roots，可以看出：
•	reference1-> 对象实例1；
•	reference2-> 对象实例2；
•	reference3-> 对象实例4；
•	reference3-> 对象实例4 -> 对象实例6；
可以得出对象实例1、2、4、6都具有GC Roots可达性，也就是存活对象，不能被GC回收的对象。
而对于对象实例3、5直接虽然连通，但并没有任何一个GC Roots与之相连，这便是GC Roots不可达的对象，这就是GC需要回收的垃圾对象。

到这里，相信大家应该能彻底明白引用计数算法和可达性算法的区别吧。

再回过头来看看最前面的实例，GcObject实例1和实例2虽然从引用计数虽然都不为0，但从可达性算法来看，都是GC Roots不可达的对象。

总之，对于对象之间循环引用的情况，引用计数算法，则GC无法回收这两个对象，而可达性算法则可以正确回收。
