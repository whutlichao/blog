---
title: android怎么确定view的大小的
tags: [android]
---

  在原生android应用中，我们知道所有的布局元素都继承自View，我们可以在布局文件中定义一个视图的大小，通常有三种类型：
  1、填充父控件
``` xml
  android:layout_width = "match_parent"
```
  2、使用固定大小
``` xml
  android:layout_width = "50dp"
```
  3、内容包裹
``` xml
  android:layout_width = "warp_content"
```
  那么问题来了，android是怎么来判断和确定控件的大小的呢？
<!--more-->

## 模式与大小
  写过自定义控件的小伙伴们应该知道，在重写onMeasure方法时我们一上来基本就会获取组件的宽、高的模式和大小
``` java
  int widthMode = MeasureSpec.getMode(widthMeasureSpec);
  int heightMode = MeasureSpec.getMode(heightMeasureSpec);
  int widthSize = MeasureSpec.getSize(widthMeasureSpec);
  int heightSize = MeasureSpec.getSize(heightMeasureSpec);
```
  这里的模式和大小分别指什么呢，并且这些大小信息是怎么缓存在内存中的呢，我们在万恶的View超类中找到了这么一个内部类，控件的测量信
  息就是存放在这个内部类实例中：
``` java
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK = 0x3 << MODE_SHIFT;

    /**
     * 下面三个静态变量就是对控件显示模式的定义
     */
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;

    public static final int EXACTLY = 1 << MODE_SHIFT;

	public static final int AT_MOST = 2 << MODE_SHIFT;

    public static int makeMeasureSpec(int size, int mode) {
        if (sUseBrokenMakeMeasureSpec) {
            return size + mode;
        } else {
            return (size & ~MODE_MASK) | (mode & MODE_MASK);
        }
    }

    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }

    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
    ......
}
```
  我们来一一分析下这个类：
  1、MODE_SHIFT：位运算的的规模，也就是位运算多少位。
  2、MODE_MASK：模式位运算基 1100 0000 0000 0000 0000 0000 0000 0000
  3、控件显示模式
  a)放任自流模式，你想多大给你多大，不指定大小
  UNSPECIFIED：0000 0000 0000 0000 0000 0000 0000 0000
  b)精确模式，完全由父控件约整子控件的大小，match_parent
  EXACTLY：0100 0000 0000 0000 0000 0000 0000 0000
  c)伸缩不自如模式，当控件定义的大小超过父控件，则使用父控件的大小，否则使用自己定义的大小，warp_content/具体大小
  AT_MOST：1000 0000 0000 0000 0000 0000 0000 0000

  在View的这个方法中，我们也可以充分理解下这三种模式
``` java
/**
 * @param  size 控件想要的大小，也就是你布局文件里想要定义的大小
 * @param  measureSpec 父控件的约束大小
 */
public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
    final int specMode = MeasureSpec.getMode(measureSpec);
    final int specSize = MeasureSpec.getSize(measureSpec);
    final int result;
    switch (specMode) {
        case MeasureSpec.AT_MOST: // 父控件大小或自定义的大小，完全取决与哪个小
            if (specSize < size) {
                result = specSize | MEASURED_STATE_TOO_SMALL;
            } else {
                result = size;
            }
            break;
        case MeasureSpec.EXACTLY:// 就是父控件的大小
            result = specSize;
            break;
        case MeasureSpec.UNSPECIFIED: // 定义的多大就给你多大
        default:
            result = size;
    }
    return result | (childMeasuredState & MEASURED_STATE_MASK);
}
```
  到此我们就会发现，android是将模式和控件大小放到一个32位的二进制数中，前两位代表的模式，后30位是大小。
  4、makeMeasureSpec、getMode、getSize
  生成测测信息、获取模式和获取大小，这三个方法就是一堆与或运算，这里没什么好讲的。
