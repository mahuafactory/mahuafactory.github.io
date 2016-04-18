---
layout: post
comments: true
categories: 动画
---
### 一 概述###
  属性动画从API 11引入，比视图动画更加强大，用官方的话说，属性动画可以改变任何事物。而视图动画仅仅是动画。视图动画基类是`android.view.animation.Animation`，而属性动画基类是`android.animation.Animator`

  如下是属性动画的继承关系：
  
  ![Animator](/icons/animator_relationship.png)

  属性动画中，属性值的计算过程如下图：

  ![Animator](/icons/property_value.jpg)
  
  由图可见，要将时间变量转换成最终改变的属性值，需要三步。

  第一：计算出动画时间的分数，根据消耗的时间占用总时间的比，得到0到1的一个分数。

  第二：计算插值(动画完成率),根据上一个值，计算出动画的完成度，可以大于1，也可以小于0。

  第三：计算需要的属性值，使用合适的`TypeEvaluator`计算出需要的属性值。


  完成了上面3步计算，最后使用得到的属性值来改变需要改变的对象，就可以得到想要的动画了。

  首先，先来了解属性动画中两个常用类`ValueAnimator`以及`ObjectAnimator`的用法。

### 二 ValueAnimator###

  `ValueAnimator`并不会直接操作任何属性，而仅仅是回调，让开发者自己做处理。

	ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
    animation.setDuration(1000);
    animation.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
       @Override
       public void onAnimationUpdate(ValueAnimator animation) {
            //...
       }
    });
    animation.setInterpolator(new LinearInterpolator());
    animation.start();

  如上，在`onAnimationUpdate`这个回调方法中，可以做出相应的操作，该方法会在每一个动画帧的时候回调。对属性等做出改变，就能产生对应的动画。

  常用的两个方法如下：
    public static ValueAnimator ofInt(int... values)  
    public static ValueAnimator ofFloat(float... values)  
  可以看到接收的是可变参数，意思是说，值可以在多个值之间变动，而不仅仅局限于从一个变化到另一个。

  另外的常用方法如下：

| 方法名      | 描述     |
| --------- |:--------:|
|setDuration(long duration)	|设置动画时长，单位毫秒|
|getAnimatedValue()	|返回动画的值|
| start()  	|开始动画|
|setRepeatCount(int value) 	|重复次数，INFINITE代表无限|
|setRepeatMode(int value)|重复模式，有RESTART(从头开始)和REVERSE(倒序开始)|
|cancel() 	|取消动画|

  `ValueAnimator`拥有两种监听器，其中`AnimatorUpdateListener`监听动画的实时变化，在上面的代码中也有提现，另外还有一个监听器`AnimatorListener`，主要用于监听动画的`start`，`end`，`cancle`，`repeart`状态。

   值得特别注意的是`ofObject(TypeEvaluator evaluator, Object... values)`方法，该方法接收一个`TypeEvaluator`以及可变参数`Object`，从上面的属性动画属性值计算过程图可以看出，经过加速器（插值器）的计算后，就需要`TypeEvaluator`将值转换成需要的值，那么这里就需要它返回对应的对象。这里需要通过自定义`TypeEvaluator`来实现。

    public interface TypeEvaluator<T> {

    public T evaluate(float fraction, T startValue, T endValue);

    }

  继承`TypeEvaluator`，并指定合适的泛型，随后通过`evaluate`方法返回指定对象。同时通过`ValueAnimator`的`setEvaluator(TypeEvaluator value)`方法指定自定义的`TypeEvaluator`即可。



