# ConstraintLayout (code)
## OverView
![](http://i.imgur.com/q9lTep6.png)

*A ConstraintLayout is a ViewGroup which allows you to position and size widgets in a **flexible** way.*

ConstraintLayout和其他布局一样，继承自ViewGroup，但是不同点在于它调整控件的位置和大小时更加得灵活，功能更加强大。

## 支持版本
ConstraintLayout是一个Support库，意味着向前兼容，它可以兼容至API 9，也就是Android 2.3，鉴于现在市场上手机基本都是2.3及以上的，所以如果不是特殊情况，开发者可以不用考虑版本问题。下面是官网文档引文：
> Note: ConstraintLayout is available as a support library that you can use on Android systems starting with API level 9 (Gingerbread). As such, we are planning in enriching its API and capabilities over time. This documentation will reflects the changes.

## 新特性
* Relative positioning		
* Margins					
* Centering positioning		
* Visibility behavior		
* Dimension constraints		
* Chains					
* Virtual Helpers objects	

### Relative positioning
Relative positioning是在ConstraintLayout中创建布局的最基本构建块，也就是一个控件相对于另一个控件进行定位，可以从横向、纵向添加约束关系：

* Horizontal Axis: Left, Right, Start and End sides
* Vertical Axis: top, bottom sides and **text baseline**

例：下面按钮B要定位在按钮A的右边一样：

![](http://i.imgur.com/3WWNP2Y.png)

    <Button android:id="@+id/buttonA" ... />
       <Button android:id="@+id/buttonB" ...
          	   app:layout_constraintLeft_toRightOf="@+id/buttonA" />

这样系统就会知道按钮B的左侧被约束在按钮A的右侧，这里的约束可以理解为边的对齐。

![](http://i.imgur.com/Jb4OxEf.png)

上图是相对定位的约束，图中每一条边（top、bottom、baseline、left、start、right、end）都可以与其他控件形成约束，罗列这些边形成的相对定位关系如下：

    layout_constraintLeft_toLeftOf
    layout_constraintLeft_toRightOf
    layout_constraintRight_toLeftOf
    layout_constraintRight_toRightOf
    layout_constraintTop_toTopOf
    layout_constraintTop_toBottomOf
    layout_constraintBottom_toTopOf
    layout_constraintBottom_toBottomOf
    layout_constraintBaseline_toBaselineOf
    layout_constraintStart_toEndOf
    layout_constraintStart_toStartOf
    layout_constraintEnd_toStartOf
    layout_constraintEnd_toEndOf

上面的这些属性需要结合id才能进行约束，这些id可以指向控件也可以指向父容器（也就是ConstraintLayout），比如：

    <Button android:id="@+id/buttonB" ...
     app:layout_constraintLeft_toLeftOf="parent" />

### Margins
![](http://i.imgur.com/iabz76K.png)

这个我们经常使用，这里就不再赘述了。我们经常使用的margin属性如下：

    android:layout_marginStart
    android:layout_marginEnd
    android:layout_marginLeft
    android:layout_marginTop
    android:layout_marginRight
    android:layout_marginBottom

下面有一个新属性：GONE MARGIN

这里的gone margin指的是B向A添加约束后，如果A的可见性变为GONE，这时候B的gone margin属性就会生效。gone margin属性如下：

    layout_goneMarginStart
    layout_goneMarginEnd
    layout_goneMarginLeft
    layout_goneMarginTop
    layout_goneMarginRight
    layout_goneMarginBottom

举个具体例子：ButtonB 左边相对于 ButtonA 右边对齐，ButtonA 左边相对于父容器左边对齐。如果 ButtonA 的状态为 GONE（不可见的），则 ButtonB 就相对于父容器左边对齐了。如果有个需求是，当 ButtonA 不可见的时候， ButtonB 和父容器左边需要一个边距 16dp。 这个时候就需要使用上面的 layout_goneMarginLeft 或者 layout_goneMarginStart 属性了，如果设置了这个属性，当 ButtonB 所参考的 ButtonA 可见的时候，这个边距属性不起作用；当 ButtonA 不可见（GONE）的时候，则这个边距就在 ButtonB 上面起作用了。

### Centering positioning and bias
假设我们要设置一个控件居中怎么办？很简单，利用上面介绍过的属性就可以办到。

    <android.support.constraint.ConstraintLayout ...>
     <Button android:id="@+id/button" ...
     app:layout_constraintLeft_toLeftOf="parent"
     app:layout_constraintRight_toRightOf="parent/>
     </>

如下图所示，这种情况就感觉像是控件两边有两个反向相等大小的力在拉动它一样，所以才会产生控件居中的效果。

![](http://i.imgur.com/mLN9tbR.png)

#### Bias
在这种约束是同向相反的情况下，默认控件是居中的，但是也可以像拔河一样，让两个约束的力大小不等，这样就产生了倾向，其属性是：

    layout_constraintHorizontal_bias
    layout_constraintVertical_bias

下面这段代码就是让左边占30%，右边占70%（默认两边各占50%），这样左边就会短一些，如下图所示，此时代码是这样的：

![](http://i.imgur.com/dbAXNNI.png)

    <android.support.constraint.ConstraintLayout ...>
     <Button android:id="@+id/button" ...
     app:layout_constraintHorizontal_bias="0.3"
     app:layout_constraintLeft_toLeftOf="parent"
     app:layout_constraintRight_toRightOf="parent/>
     </>

通过设置bias,可以更好的进行屏幕适配。

### Visibility behavior
ConstraintLayout对可见性被标记View.GONE的控件（后称“GONE控件”）有特殊的处理。一般情况下，GONE widgets是不可见的，且不再是布局的一部分，而且他们的尺寸大小也不会改变。
但是在布局计算上，ConstraintLayout仍然会将GONE widgets进行计算：

* GONE widgets的尺寸会被认为是0（或者说当做点来处理）
* 假如其他控件相对该GONE widgets有约束的话，该控件仍然会被计算在内，只是其所有的margin属性都会按0计算

![](http://i.imgur.com/LrTRifj.png)

这种特殊的设计优点就是在一个widget设置为Gone的时候不会破坏原始的布局结构，这点对于做简单的布局动画也非常有用。
看下面一个例子：
两个button，A水平居中，B在A的右侧约束。

![](http://i.imgur.com/XVRGOP7.png)

然后A 的可见性设为Gone，这时A可以看做一个点（若有margin属性的话都会被置为0），此时B的位置变为如下图所示：

![](http://i.imgur.com/8N0Z6Cr.png)

### Dimensions constraints
ConstraintLayout本身可以定义自己的最小尺寸

    android:minWidth 设置布局的最小宽度
    android:minHeight 设置布局的最小高度
这些最小尺寸当ConstraintLayout被设置为WRAP_CONTENT时有效。

#### Widgets dimension constraints
控件的尺寸可以通过android:layout\_width和android:layout\_height来设置，有三种方式：

* 使用固定值(128dp)
* 使用WRAP_CONTENT
* 使用0dp（相当于MATCH_CONSTRAINT）

![](http://i.imgur.com/lnW7Hhb.png)

前两个属性的使用方法和其他Layout一样，最后一个是通过填充约束来计算widget的尺寸，（(a) is wrap_content, (b) is 0dp），假如设置了margin的话就会如（C）所示((c) with 0dp).
> MATCH\_PARENT 属性在ConstraintLayout包含的子View中不再支持，如果widget是相对于“parent”约束的话，设为0dp(MATCH\_CONSTRAINT)就可以实现MATCH\_PARENT的效果。

### Ratio
这里的比例指的是宽高比，通过设置比例，让宽高的其中一个随另一个变化。若要使用该属性，需要让widget的宽或高至少一个设置为0dp（也就是MATCH\_CONSTRAINT），然后设置layout\_constraintDimensionRatio属性，如下所示，将会设置控件width 和 height相等：

    <Button android:layout_width="wrap_content"
       android:layout_height="0dp"
       app:layout_constraintDimensionRatio="1:1" />

layout_constraintDimensionRatio可以设置两种类型的值：

* float： "width/height"
* "width:height"

还有一种情况就是width和height都设置为MATCH_CONSTRAINT（0dp），这种情况下，系统会将宽高设置为既满足所有约束又满足宽高比的最大值。为了让一条边相对于另一条来进行布局，可以在ratio前面加上“W/H”参数，这样受约束的一方将根据不受约束的一方，按照比例计算自己的尺寸。

    <Button android:layout_width="0dp"
       android:layout_height="0dp"
       app:layout_constraintDimensionRatio="H,16:9"
       app:layout_constraintBottom_toBottomOf="parent"
       app:layout_constraintTop_toTopOf="parent"/>

上面的 layout_constraintDimensionRatio 取值为 H,16:9，前面的 H 表明约束高度，所以结果就是 Button 的宽度和父容器宽度一样，而高度值符合 16:9 的比率。

### Chains
Chains 为同一个方向（水平或者垂直）上的多个子 View 提供一个类似群组的概念。其他的方向则可以单独控制。
#### 创建一个 Chain
多个 View 相互在同一个方向上双向引用就创建了一个 Chain。什么是双向引用呢？ 比如在水平方向上两个 Button A 和 B，如果 A 的右边位于 B 的左边，而 B 的左边位于 A 的右边，则就是一个双向引用。如下图：

![](http://pic.goodev.org/wp-files/2017/02/cl/chains.png)

> 注意： 在 Android Studio 编辑器中，先把多个 View 单向引用，然后用鼠标扩选多个 View，然后在上面点击右键菜单，选择 “Center Horizontally” 或者 “Center Vertically” 也可以快速的创建 Chain。

#### Chain heads
Chain 的属性由该群组的第一个 View 上的属性所控制（第一个 View 被称之为 Chain head）.

![](http://pic.goodev.org/wp-files/2017/02/cl/chains-head.png)

水平群组，最左边的 View 为 head， 垂直群组最上面的 View 为 head。

#### Margins in chains
可以为 Chain 中的每个子 View 单独设置 Margin。对于 spread chains， 可用的布局空白空间是扣除 margin 后的空间。下面会详细解释。

#### Chain Style
下面几个属性是控制 Chain Style 的：

    * layout_constraintHorizontal_chainStyle
    * layout_constraintHorizontal_weight
    * layout_constraintVertical_chainStyle
    * layout_constraintVertical_weight

chainStyle 是设置到 Chain Head 上的，指定不同的 style 会改变里面所有 View 的布局方式，有如下四种 Style：

* CHAIN\_SPREAD 这个是默认的 Style， 里面的所有 View 会分散开
* Weighted chain，在 CHAIN\_SPREAD 模式下，如果有些 View 的尺寸设置为 MATCH\_CONSTRAINT（0dp）,则这些 View 尺寸会占据所有剩余可用的空间，和 LinearLayout weight 类似。
* CHAIN\_SPREAD\_INSIDE 和 CHAIN\_SPREAD 类似，只不过两端的两个 View 和 父容器直接不占用多余空间，多余空间在 子 View 之间分散
* CHAIN\_PACKED 这种模式下，所有的子 View 都 居中聚集在一起，但是可以设置 bias 属性来控制聚集的位置。

![](http://pic.goodev.org/wp-files/2017/02/cl/chains-styles.png)

如果多个子View尺寸设置为 MATCH\_CONSTRAINT（0dp），则这些 View 会平均的占用多余的空间。通过 layout_constraintXXX\_weight 属性，可以控制每个 View 所占用的多余空间的比例。例如，对于只有两个 View 的一个水平 Chain，如果每个View 的宽度都设置为 MATCH\_CONSTRAINT， 第一个 View 的 weight 为 2；第二个 View 的 weight 为 1，则第一个 View 所占用的空间是 第二个 View 的两倍。
## Virtual Helper objects
这里“辅助工具”的原文是Virtual Helper objects，对于ConstraintLayout，其辅助工具目前是Guidline。
### Guideline
Guideline 是 ConstraintLayout 中一个特殊的辅助布局的类。相当于一个不可见的 View，使用 ConstraintLayout 可以创建水平或者垂直的参考线，其他的 View 可以相对于这个参考线来布局。

* 垂直 Guideline 的宽度为 0， 高度为 父容器（ConstraintLayout）的高度
* 水平 Guideline 的高度为 0， 宽度为 父容器（ConstraintLayout）的宽度

参考线的位置是可以移动的。

* layout\_constraintGuide\_begin 可以指定距离左（或者上）边开始的固定位置
* layout\_constraintGuide\_end 可以指定距离右（或者下）边开始的固定位置
* layout\_constraintGuide\_percent 可以指定位于布局中所在的百分比，比如距离左边 2% 的位置

下面是一个使用垂直 Guideline 的示例, Button 相对于 guideline 布局：

    <android.support.constraint.ConstraintLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
    
	    <android.support.constraint.Guideline
		    android:layout_width="wrap_content"
		    android:layout_height="wrap_content"
		    android:id="@+id/guideline"
		    android:orientation="vertical"
		    app:layout_constraintGuide_percent="0.4" />
	    
	    <Button
		    android:text="Button"
		    android:layout_width="wrap_content"
		    android:layout_height="wrap_content"
		    android:id="@+id/button"
		    app:layout_constraintLeft_toRightOf="@+id/guideline"
		    app:layout_constraintTop_toTopOf="parent"
		    app:layout_constraintBottom_toBottomOf="parent" />
	    
	    <Button
		    android:id="@+id/button14"
		    android:layout_width="wrap_content"
		    android:layout_height="wrap_content"
		    android:text="Button"
		    app:layout_constraintRight_toLeftOf="@+id/guideline"
		    app:layout_constraintTop_toTopOf="parent"
		    app:layout_constraintBottom_toBottomOf="parent" />
    
    </android.support.constraint.ConstraintLayout>

效果图如下：

![](http://i.imgur.com/OwOSmHb.png)

## 通过代码来设置 ConstraintLayout 属性

上面只是 XML 布局文件中使用的属性，只能在 XML 布局文件中使用，但是现在针对 UI 做动画的时候，需要通过代码来动态设置 View 的布局属性。下面就来看看如何通过代码来设置这些属性。
### ConstraintSet
ConstraintSet 是用来通过代码管理布局属性的集合对象，可以通过这个类来创建各种布局约束，然后把创建好的布局约束应用到一个 ConstraintLayout 上，可以通过如下几种方式来创建 ConstraintSet：

* 手工创建：

    `c = new ConstraintSet(); c.connect(....);`

* 从 R.layout.* 对象获取

    `c.clone(context, R.layout.layout1);`

* 从 ConstraintLayout 中获取

    `c.clone(clayout);`

然后通过 applyTo 函数来应用到ConstraintLayout 上


    import android.content.Context;
    import android.os.Bundle;
    import android.support.constraint.ConstraintLayout;
    import android.support.constraint.ConstraintSet;
    import android.support.transition.TransitionManager;
    import android.support.v7.app.AppCompatActivity;
    import android.view.View;
    
    public class MainActivity extends AppCompatActivity {
	    ConstraintSet mConstraintSet1 = new ConstraintSet(); // create a Constraint Set
	    ConstraintSet mConstraintSet2 = new ConstraintSet(); // create a Constraint Set
	    ConstraintLayout mConstraintLayout; // cache the ConstraintLayout
	    boolean mOld = true;
	    
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
		    super.onCreate(savedInstanceState);
		    Context context = this;
		    mConstraintSet2.clone(context, R.layout.state2); // get constraints from layout
		    setContentView(R.layout.state1);
		    mConstraintLayout = (ConstraintLayout) findViewById(R.id.activity_main);
		    mConstraintSet1.clone(mConstraintLayout); // get constraints from ConstraintSet
	    }
	    
	    public void foo(View view) {
		    TransitionManager.beginDelayedTransition(mConstraintLayout);
		    if (mOld = !mOld) {
		    	mConstraintSet1.applyTo(mConstraintLayout); // set new constraints
		    }  else {
		    	mConstraintSet2.applyTo(mConstraintLayout); // set new constraints
		    }
	    }
    }

## 总结
1. 布局灵活性更高，功能更加强大
2. 适用于配合UI编辑器编写，更加方便
3. 可以有效的解决布局嵌套的问题
4. 可以设计出更具有适配性的布局

## 参考链接
[ConstraintLayout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html "ConstraintLayout")

[Guideline](https://developer.android.com/reference/android/support/constraint/Guideline.html)

[ConstraintLayout 终极秘籍](http://blog.chengyunfeng.com/?p=1030&utm_source=tuicool&utm_medium=referral)
