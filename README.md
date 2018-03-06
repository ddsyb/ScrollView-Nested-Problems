# ScrollView-Nested-Problems
解决Android中出现ScrollView嵌套 GridView 、WebView 、RecyclerView出现的高度问题。

开篇语：最近开始想写一些技术总结了，一方面分享给其他同学，另一方面也作为自己的技术积累。
今天我分享的是日常遇到的问题，ScrollView组件里面嵌套GridView、WebView、ListView等本身具有滑动的组件时，所引发的高度显示不全的问题。

对于GridView、WebView、ListView这三类组件来说，大家都知道通常情况下这三类组件本身是具有滑动特性的，其实际占用的高度也只是屏幕内显示的高度，当嵌套在ScrollView组件里时就造成了冲突。
所以解决的思路可以从其onMeasure方法入手，onMeasure方法是重写自定义View中用到的一个非常重要的方法，onMeasure方法的作用就是计算出自定义View的宽度和高度，这个计算的过程参照父布局给出的大小，以及自己特点算出结果。onMeasure()是在父视图计算子视图大小时被调用的，其中的细节就不在此详细讲述。

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


