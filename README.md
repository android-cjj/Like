#那些年，我们一起点过的赞

![](http://ww3.sinaimg.cn/mw690/7ef01fcagw1f25shwbp2mj20go0afgmw.jpg)

手在键盘敲很轻，欲说却不知从何吐槽......咳，刚装逼了，现在进入正题。几乎每个社交App都有点赞功能，但是就国内的app来说，你可能记得点赞的内容而压根忽视了点赞的效果。举个例子，就用户最多的微信、QQ来说，点赞也就是个心形和拇指的放大动画（自己去体验下），这里顺便吐槽下网易的点赞，动画做的不错，虽然我手机小小不流畅，可是不能取消赞是怎么回事？ 也许，现在你觉得无非就是个点赞效果，随便做个点击效果就好了，也许产品设计的人也是这样觉得的，也许用户根本就不在乎。我想说的是，友好的交互，会影响用户的体验，做好细节，会让App更成功。

####介绍下我认为点赞做的比较好的App及简单介绍如何实现。

###Periscope一款用户可以向其他人直播视音频的App,点赞效果让人眼前一亮。

大概在一年前，程序亦非猿就写了[一步一步教你实现Periscope点赞效果](http://www.jianshu.com/p/03fdcfd3ae9c)，我也在个人项目中用了，感觉良好。

![](http://ww3.sinaimg.cn/mw690/7ef01fcagw1f25od1so1kg205k07ittz.gif)

作者实现这个效果的思路是：
* 1.爱心出现在底部并且水平居中
* 2.爱心的颜色/类型 随机
* 3.爱心进入时候有一个缩放的动画
* 4.缩放完毕后,开始变速向上移动,并且伴随alpha渐变效果
* 5.爱心移动的轨迹光滑,是个曲线

其实，难点就是曲线运动，作者用了三次方贝塞尔曲线的公式：

![](http://e.hiphotos.baidu.com/baike/s%3D421/sign=9a6521eab8014a90853e47bf98763971/f603918fa0ec08fad54f8dff58ee3d6d55fbda1f.jpg)

P0,是爱心的起点,P3是终点,P1,P2是途径的两个点。在自定义TypeEvaluator，实现了动画曲线效果。具体你可以去看作者原文，因为他写了，我就不啰嗦了。我讲讲不够优美的地方，爱心出来移动太分散，源码中，作者提供P1,P2点时候，处理的很随意，所以这里是可以做优化的。
```java
   /**
     * 获取中间的两个 点
     *
     * @param scale
     */
    private PointF getPointF(int scale) {
        PointF pointF = new PointF()
        pointF.x = random.nextInt((mWidth - 100));//减去100 是为了控制 x轴活动范围,看效果 随意~~
        //再Y轴上 为了确保第二个点 在第一个点之上,我把Y分成了上下两半 这样动画效果好一些  也可以用其他方法
        pointF.y = random.nextInt((mHeight - 100)) / scale;
        return pointF;
    }
```
程序耦合性有点高，也就是不能自定义，这个也是可以做处理的。总体来说，写的很好。

小编采访作者为什么写这个？

[程序亦非猿](https://github.com/AlanCheen):"写了快一年了...跟风persicope啊，哈哈 , 效果帅 ,就想实现 ,如此而已"


###Twitter（中文称：推特）是国外的一个社交网络及微博客服务的网站,App点赞效果十分漂亮

我最早是看到[frogermcs/LikeAnimation](https://github.com/frogermcs/LikeAnimation)实现了，效果如下：

![](https://camo.githubusercontent.com/752e0e35b15b6b684cee90b6bf4309f387caa36f/687474703a2f2f66726f6765726d63732e6769746875622e696f2f696d616765732f32322f627574746f6e5f616e696d2e676966)

看起来也挺复杂的，作者用了很简单的实现方式，一步一步分出来。

1.先实现一个画圆的视图[CircleView](https://github.com/frogermcs/LikeAnimation/blob/master/app/src/main/java/frogermcs/io/likeanimation/CircleView.java)

![](http://frogermcs.github.io/images/22/circle_anim.gif)

看图知道主要在`ondraw()`函数画两个不断扩大圆就可以，内部圆提供的画笔path 设置：
 ```java
 maskPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.CLEAR));
```

2.接下来就是实现一个画四周点的视图[DotsView](https://github.com/frogermcs/LikeAnimation/blob/master/app/src/main/java/frogermcs/io/likeanimation/DotsView.java)

![](http://frogermcs.github.io/images/22/dots_anim.gif)

其实还是画圆，我们需要确定各个圆的圆心
```java
int cX = (int) (centerX + currentRadius1 * Math.cos(i * OUTER_DOTS_POSITION_ANGLE * Math.PI / 180));
int cY = (int) (centerY + currentRadius1 * Math.sin(i * OUTER_DOTS_POSITION_ANGLE * Math.PI / 180)); 
```
3.星星的缩放动画，这个很简单，作者没写个画星星的类，直接用图片代替了

![](http://frogermcs.github.io/images/22/touch_anim.gif)

4.最后写了个LikeButtonView继承FrameLayout，把以上三部分组合起来了

![](http://ww1.sinaimg.cn/mw690/7ef01fcagw1f25r8lk072j20b506m3yr.jpg)

看布局源码就知道了
```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

    <frogermcs.io.likeanimation.DotsView
        android:id="@+id/vDotsView"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center"/>

    <frogermcs.io.likeanimation.CircleView
        android:id="@+id/vCircle"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:layout_gravity="center"/>

    <ImageView
        android:id="@+id/ivStar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:src="@drawable/ic_star_rate_off"/>
</merge>
```
因为没有提供向外监听的方法，所以你是无法知道选择的state的，这点可以添加。动画效果还是挺流畅的，但是本人不是很喜欢这种实现方式，因为我觉得一个View也是可以实现这种效果的，不信？我接下来介绍的就是这样做的。

小编采访作者为什么写这个？

[frogermcs](https://github.com/frogermcs):"Hello, Chinese friends,我瞎掰了，呵呵"

###最新Twitter App 版本的点赞效果有了一点变化，还是很漂亮

作者[hanks-zy](https://github.com/hanks-zyh)实现了这种效果[SmallBang](https://github.com/hanks-zyh/SmallBang)

![](https://github.com/hanks-zyh/SmallBang/raw/master/screenshots/heart.gif)

hanks实现这个的思路是：
* 1.将当前的view变小,画圆半径从小到大,同时颜色渐变 (P1)
* 2.当半径到达MAX_RADIUS,开始画空心圆,空闲圆半径变大,画笔宽度从MAX_RADIUS变小
* 3.当空心圆半径达到 MAX_RADIUS - RINGWIDTH (P2), 此时变成圆环,在圆环上生成个DOT_NUMBER个小圆,均匀分布
* 4.空心圆继续变大,逐渐圆环消失; 同时小圆向外扩散,扩散过程小圆半径减小,颜色渐变;同时下面的view逐渐变大 (P3)

从源码上看，就一个view类，SmallBang继承了View,主要是在这个`bang(...)`函数做了位置的初始化和动画的逻辑,在`ondraw()`函数根据动画过程得到的progress和自己定义好的P1,P2,P3去绘制各个过程。

动画效果还是挺流畅，效果也挺棒。这里说说这个库的不足的地方。
```java
    public static SmallBang attach2Window(Activity activity) {
        ViewGroup rootView = (ViewGroup) activity.findViewById(Window.ID_ANDROID_CONTENT);
        SmallBang smallBang = new SmallBang(activity);
        rootView.addView(smallBang, new ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
        return smallBang;
    }
```
传入参数是activity，会导致如果是dialog使用，就必须改代码，这里写死了，可以优化。
因为此效果多半在列表中使用，所以测试中效果发生的位置会有偏差，这样是位置获取采用的方式所导致的。
还有就是作者也做取消的效果，在里面设置个isSelect的状态，并向外提供这个状态的方法。

小编采访作者为什么写这个？

[hanks](https://github.com/hanks-zyh):"使用 Twitter 的时候看到了，感觉动画很活泼，感觉实现起来也不难。"

###最后说说
看完这么多漂亮的点赞效果，我们应该是思考，为什么国外的App肯花费那么多时间去完成这个不起眼的小功能————点赞。我觉得，他们注重细节，注重交互。希望做产品的同学可以看到，其实，我们程序员可以完成很多绚丽的东西，请设计的时候，不要那么单调。那些年，我们一起点过的赞可能不是很漂亮，希望，以后的赞可以真的很赞。









