# ScrollView-Nested-Problems
解决Android中出现ScrollView嵌套 GridView 、WebView 、ListView、RecyclerView出现的高度问题。

开篇语：最近开始想写一些技术总结了，一方面分享给其他同学，另一方面也作为自己的技术积累。
今天我分享的是日常遇到的问题，ScrollView组件里面嵌套GridView、WebView、ListView等本身具有滑动的组件时，所引发的高度显示不全的问题。

对于GridView、WebView、ListView这三类组件来说，大家都知道通常情况下这三类组件本身是具有滑动特性的，其实际占用的高度也只是屏幕内显示的高度，当嵌套在ScrollView组件里时就造成了冲突。
所以解决的思路可以从其onMeasure方法入手，onMeasure方法是重写自定义View中用到的一个非常重要的方法，onMeasure方法的作用就是计算出自定义View的宽度和高度，这个计算的过程参照父布局给出的大小，以及自己特点算出结果。onMeasure方法是在父视图计算子视图大小时被调用的，其中的细节就不在此详细讲述。

  @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    
        int mExpandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, mExpandSpec);
        
    }
    
    
上面方法的2个参数，来自于父视图发过来给子视图的限制条件,这涉及到一个知识点MeauseSpec这个类.

一.MeasureSpec的构成

MeasureSpec代表一个32位的int值，前俩位代表SpecMode，后30位代表SpecSize.其中：SpecMode代表测量的模式，SpecSize值在某种测量模式下的规格大小。

共有三种测量模式： 

1.EXACTLY: 父容器已经检测出子View所需要的精确大小，这个时候view的大小即为SpecSize的大小，他对应于布局参数中的MATCH_PARENT,或者精确大小值；

2.AT_MOST: 父容器指定了一个大小，即SpecSize，子view的大小不能超过这个SpecSize的大小；

3.UNSPECIFIED: 父容器对子View的大小没有任何要求,子View想多大都可以。


二.如何创建MeasureSpec
MeasureSpec内部提供了创建MeasureSpec的方法：

public static int makeMeasureSpec(int size, int mode) {

            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
            
        }

private static final int MODE_SHIFT = 30;

private static final int MODE_MASK  = 0x3 << MODE_SHIFT; 二进制 1100....00 32位

public static final int UNSPECIFIED = 0 << MODE_SHIFT;   二进制 0000....00 32位

public static final int EXACTLY     = 1 << MODE_SHIFT;   二进制 0100....00 32位

public static final int AT_MOST     = 2 << MODE_SHIFT;   二进制 1000....00 32位
  
MeasureSpec代表一个32位的int值，前俩位代表SpecMode，后30位代表SpecSize.通过巧妙的位运算，即可通过MeasureSpec来得到SpecSize,SpecMode.
public static int getMode(int measureSpec) {

            return (measureSpec & MODE_MASK);  
        }
  
public static int getSize(int measureSpec) {

            return (measureSpec & ~MODE_MASK);
        }

---------------------------------------------------------------------------------------------
对于RecyclerView来说,重写onMeasure()方法就不管用了。
1.一种解决办法是在RecyclerView的外部套上一层RelativeLayout
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:descendantFocusability="blocksDescendants">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/menuRv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="@dimen/margin_16"
            android:layout_marginRight="@dimen/margin_16"/>

</RelativeLayout>

android:descendantFocusability属性的值有三种： 

beforeDescendants：viewgroup会优先其子类控件而获取到焦点 

blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点 

afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点

但是这个方案recyclerView时有卡顿的问题
原因还是滑动冲突的问题，可以重写LinearLayoutManager，设置让其不可滑动，外部滑动靠ScrollView,这样就解决了滑动时卡顿的问题 

代码如下：
    public class ScrollLinearLayoutManager extends LinearLayoutManager {
        private boolean isScrollEnabled = true;

        public ScrollLinearLayoutManager(Context context) {
            super(context);
        }

        public ScrollLinearLayoutManager(Context context, int orientation, boolean reverseLayout) {
            super(context, orientation, reverseLayout);
        }

        public ScrollLinearLayoutManager(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
            super(context, attrs, defStyleAttr, defStyleRes);
        }

        public void setScrollEnabled(boolean flag) {
            this.isScrollEnabled = flag;
        }

        @Override
        public boolean canScrollVertically() {
            return isScrollEnabled && super.canScrollVertically();
        }
    }

简单使用：
    ScrollLinearLayoutManager scrollLinearLayoutManager = new ScrollLinearLayoutManager(this);
    
    scrollLinearLayoutManager.setScrollEnabled(false);
    mRecyclerView.setLayoutManager(scrollLinearLayoutManager);

2.完美方案是这样的：首先在xml布局中将你的ScrollView替换成android.support.v4.widget.NestedScrollView，并在java代码中设置recyclerView.setNestedScrollingEnabled(false);












