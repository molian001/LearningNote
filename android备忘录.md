#Android知识备忘录

##android异步消息处理机制
[http://blog.csdn.net/guolin_blog/article/details/9991569](http://blog.csdn.net/guolin_blog/article/details/9991569)

##View的事件分发机制

###点击事件传递规则

主要是三个方法：

public boolean dispatchTouchEvent(MotionEvent ev)

用来进行事件的分发。如果事件能传递到当前view，此方法一定会被调用，返回结果受当前view的onTouchEvent和下级view的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

public boolean onInterceptTouchEvent(MotionEvent ev)

在上述方法内部调用，用来判断是否拦截某个事件，如果当前view拦截了某个事件，那么在同一个事件序列中，此方法不会再调用，返回结果表示是否拦截当前事件。

public boolean onTouchEvent(MotionEvent ev)

在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前view无法再次接收到事件。

它们之间的关系可以用以下伪代码表示：
	
	public boolean dispatchTouchEvent (MotionEvent ev){
		boolea consume=false;
		if(onInterceptTouchEvent(ev)){
			consume=TouchEvent(ev);
		}
		else{
			childView.dispatchEvent(ev);
		}
		return consume
	}
当一个view需要处理事件时，如果它设置了onTouchListener，那么OnTouchListener中的onTouch方法会回调。此时事件如何处理还要看onTouch的返回值，如果返回false，则当前View的onTouchEvent会被调用；返回true，那么onTouchEvent方法将不会调用。

给view设置的OnTouchListener，其优先级比onTouchEvent要高。在onTouchEvent中如果当前设置有OnClickListener，那么它的OnClick方法会被调用。OnClickListener，其优先级最低，处于事件传递的尾端。


如果一个view的 OnTouchEvent返回false，那么它的父容器的onTouchEvent将会被调用，以此类推，若所有元素都不处理这个事件，那么这个事件将会最终传递给Activity处理。

###总结
1. 同一事件序列指以down事件开始，中间含有多个move，最终以up事件结束
2. 正常情况下，一个事件序列只能被一个view拦截且消耗。因为一个view一旦决定拦截，那么这一个事件序列都有它来处理（如果事件序列能传递给他的话），并且它的onInterceptTouchEvent不会再被调用（系统会把同一个事件序列内的其他方法都直接交给他处理，因为就不用调用onInterceptTouchEvent去询问它是否要拦截了）
3. 一个view一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一事件序列中的其他事件都不会再交给他处理，并且事件将重新交给它的父元素去处理，即父元素的onTouchEvent会被调用。
4. 如果view不消除除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent不会调用，并且当前View可以持续收到后续事件，最终这些消失的点击事件会传递给Activity处理。
5. ViewGroup默认不拦截任何事件。Android源码中ViewGroup的OnInterceptTouchEvent方法默认返回false。
6. View没有OnInterceptTouchEvent，一旦有点击事件传递给它，它的onTouchEvent就会调用。
7. View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认都为false，clickable要分情况，比如Button默认为true，而TextView的clickable默认属性为false。
8. view的enable属性不影响onTouchEvent的返回值。哪怕一个View是disable状态，只要它的clickable和longClickable有一个为true，它的onTouchEvent就返回true


##View工作原理

###MeasureSpec

MeasureSpec代表一个32位int值，高2位代表SpecMode，低30位代表SpecSize，SpecMode是指测量模式，而SpecSize是指在某种测量模式下的规格大小。

SpecMode有三类：

**UNSPECIFIED**  
父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态。

**EXACTLY**  
父容器已经检测出View所需的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式。

**ATMOST**      
父容器指定一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么值要看不同View的具体实现。它对应于LayoutParams中的wrap_content。

对于DecorView，其MeasureSpec由窗口的尺寸和其自身的LayoutParams来共同确定；对于普通View，其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定，MeasureSpec一旦确定后，onMeasure中就可以确定View的测量宽/高。

##View的工作流程

view的工作流程 主要指measure，layout，draw这三大流程，即测量，布局和绘制。其中measure确定view的测量宽/高，layout确定View的最终宽/高和四个顶点的位置，而draw则将View会知道屏幕上。

###measure

**view的measure过程**  
View的measure过程由其measure方法来完成，measure方法时final类型方法，不能重写。measure方法内部会调用view的onMeasure方法。View的onMeasure方法如下：
	
	protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec){
		setMeasureDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
	}

而在getDefaultSize方法中：
	
	public static int getDefaultSize(int size,int measureSpec){
		int result=size;
		int specMode=MeasureSpec.getMode(measureSpec);
		int specSize=MeasureSpec.getSize(measureSpec);

		switch(specMode){
			case MeasureSpec.UNSPECIFIED:
				result=size;
				break;
			case MeasureSpec.ATMOST:
			case MeasureSpec.EXACTLY:
				result=measureSpec;
				break;
		}
			return result;
	}

可以看出，在ATMOST,EXACTLY模式中，返回的就是specSize，即测量后的大小。而ATMOST,EXACTLY在默认情况下，这两个的效果是相等的，直接继承View的自定义控件需要重写onMeasure方法并设置wrapContent时的自身大小。

至于UnSPECIFIED这种情况，一般用于系统内部的测量过程，这种情况下大小为getSuggestedMinmumWith/Height这两个方法的返回值。源码:
	
	protected int getSuggestedMinimumWidth(){
		return(mBackground==null)?mMinWidth:max(mMinWidth,mBackground.getMinimumWidth());	
	}

从源码可看出，如果View没有设置背景，那么view的宽度为mMinWidth，mMinWidth是对应于android：minWidth 这个属性所指定的值，因此view的宽度为android：minWidth的值，如果没设定这个值，则默认为0；若View指定了背景，则取minWidth的值和背景Drawable的原始宽度的最大值。

	public int getMinimumWidth(){
		final int intrinsicWidth=getIntrinsicWidth();
		return intrinsicWidth>0?intrinsicWidth:0;
	}
可以看出，上面代码返回的就是Drawable的原始高度，如果没有原始高度，则返回0；

**ViewGroup的measure过程**

对于ViewGroup来说，除了完成自己的measure过程，还要遍历完成所有子元素的measure过程。和View不同的是，ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，而是提供了一个叫measureChildren的方法，如下

	protected void measureChildren(int widthMeasureSpec,int heightMeasureSpec{
		final int size=mChildrenCount;
		final View[] children=mChildren;
		for(int i=0;i<size;i++){
			final View child=children[i];
			if((child.mViewFlags&VISIBILITY MASK)!=GONE){
				mesureChild(child,widthMeasureSpec,heightMeasureSpec);
			}
		}
	}

而mesureChild的思想就是取出子元素的LayoutParams，然后通过getChildMeasureSpec来创建子元素的MeasureSpec，然后将MeasureSpec作为参数传递给子元素的measure方法来进行测量。子元素测量完毕后根据子元素的情况来测量自己的大小。

measure完成后，通过getMeasuredWith/height 方法就可以正确地获取到View的测量宽/高。但是，在某些情况下，系统可能需要多次measure才能确定最终的测量宽/高，在这种情况下，在onMeasure方法中拿到的宽高可能不准确。一个较好的习惯是在onLayout方法中获取View的最终宽高。

有一种情况，想在Activity已启动的时候就做一件任务，但是这一任务需要获取某个View的宽高。此时不能在onCreate或者onResume里面去获取这个View的宽高。因为View的measure过程和Activity生命周期不是同步的。

此时可以在  onWindowFocusChanged去获取，此方法是View已经初始化完毕后执行的回调方法。
此方法在Activity的窗口得到焦点和失去焦点的时候都会调用一次。

###layout

layout的作用是ViewGroup确定子元素的位置，当ViewGroup的位置确定后，它在onLayout中遍历所有的子元素并调用其layout方法，在layout中，onlayout又会被调用。layout相比较measure过程就简单多了，layout方法确定View本身的位置，而onLayout方法则会确定所有子元素的位置。

###draw

Draw过程比较简答，它的作用是将View绘制到屏幕上面。View的过程遵循以下几步:  

(1)绘制背景background.draw(cavas).

(2)绘制自己(onDraw)

(3)绘制children（dispatchDraw）dispatchDraw会遍历调用所有子元素的draw方法

(4)绘制装饰(onDrawScrollBars)

