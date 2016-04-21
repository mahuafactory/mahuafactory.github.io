---
layout: post
comments: true
categories: 动画
---
### 一 概述###

  LayoutAnimation和LayoutTransition都是运用于容器的，作用于他们的child。

  这里于LayoutAnimation类似的还有gridLayoutAnimation。

### 二 layoutAnimation和gridLayoutAnimation###

  他们是API 1就加入的，用于将动画运用在容器item上，因此它们使用的也是view animation（同样是API 1加入的视图动画），他们使用上可以通过xml以及代码方式实现。

  先看layoutanimation实现的效果：

  ![animation](/icons/animation/layoutanimation.gif)

  从上面的图中可以看出，listview在初始化并显示的时候，其初始条目有进入动画，但是后期加入的数据则没有。这是layoutAnimation的一个特点。

  xml实现代码：

        <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/btn_add_data"
        android:layoutAnimation="@anim/layout_animation"/>

  layout_animation.xml

	<?xml version="1.0" encoding="utf-8"?>
	<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                 android:animation="@anim/left"
                 android:animationOrder="normal"
                 android:delay="1"/>

  left.xml

    <?xml version="1.0" encoding="utf-8"?>
	<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="-50%p"
    android:toXDelta="0"
    android:duration="500"/>

  首先是在listview的xml中，配置layoutanimation的xml文件，随后在layoutanimation的xml中配置条目的相关动画即可。

  值得说明的是layoutAnimation中的两个属性
    android:animationOrder="normal"
    android:delay="1"

  animationOrder 代表了item运动的顺序,normal(正序)、reverse(倒序)、random(随机).
  delay 代表了每个Item开始动画的延迟时间，取值为animtaion中动画的duration倍数，这里就是500毫秒。

  当然，以上通过xml实现的效果也可以通过代码实现：

        mListView = (ListView) findViewById(R.id.list_view);
        mAarrayAdapter = new ArrayAdapter<>(this, android.R.layout.simple_expandable_list_item_1, getStrs());
        TranslateAnimation translateAnimation = new TranslateAnimation(-500, 0, 0, 0);
        translateAnimation.setDuration(1000);
        LayoutAnimationController controller = new LayoutAnimationController(translateAnimation);
        controller.setDelay(1);
        mListView.setLayoutAnimation(controller);
        mListView.setAdapter(mAarrayAdapter);
  
  在代码中layoutAnimation需要用LayoutAnimationController代替。
  
  
  gridLayoutAnimation的效果如下：

  ![animation](/icons/animation/gridLayoutAnimation.gif)

  效果和layoutAnimation基本类似，新添加的数据也不会有效果，这里就不做演示。不过不同的地方在于，它不仅仅有行，还有列，所以他的属性要多一点。先看xml实现：

    <gridLayoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                     android:animation="@anim/left_in"
                     android:columnDelay="70%"
                     android:direction="top_to_bottom"
                     android:directionPriority="none"
                     android:rowDelay="60%"/>

  其中有些属性需要单独说说
  rowDelay： 每一行动画开始的延迟，这里使用的是百分比，也就是animation中指定动画的时间比例。
  columnDelay：每一列动画开始的延迟，同样使用的是百分比。
  directionPriority：这里是none，代表行列同时开始，row，column分别代表行列优先。
  direction：方向，默认是从左到右，从上到下，所以这里设置了从上到下也没有实际意义。可以调整方向。

  同样的，也可以使用纯代码的方式实现，代码中需要使用`GridLayoutAnimationController`,其余的和上面的LayoutAnimation基本一致。这里就不写示例了。


### 二 LayoutTransition###

  和属性动画一起在API 11的时候加入。它使用的是属性动画。主要负责容器内item添加删除时候的效果。
  先看一下效果：

  ![animation](/icons/animation/layouttransitiondefault.gif)

  这里是layoutransition的默认效果，仅仅是通过xml开启了容器内item增删的动画。
  
  xml中如下：
 
	    <LinearLayout
        android:animateLayoutChanges="true"
        android:id="@+id/linear"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/btn_add"
        android:orientation="vertical"/>

  重点在于第一句` android:animateLayoutChanges="true"`指定开启了layout变化的动画，可以看到，默认的动画有透明度变化，以及其他条目的移动效果。

  如果不想使用默认效果，那么就需要使用Layouttransition来自定义。
  先看一个自定义的效果，这里button加入的时候，有一个rotation效果。并且其他条目有一个平移效果。

  ![animation](/icons/animation/layouttransitioncustom.gif)

  代码如下：

    LayoutTransition layoutTransition = new LayoutTransition();
    ObjectAnimator animator = ObjectAnimator.ofFloat(mLinear, "rotationX", 0, 360);
    layoutTransition.setAnimator(LayoutTransition.APPEARING, animator);
    PropertyValuesHolder holder1 = PropertyValuesHolder.ofInt("left", 0, 0);
    PropertyValuesHolder holder2 = PropertyValuesHolder.ofInt("top", 0, 0);
    PropertyValuesHolder holder3 = PropertyValuesHolder.ofInt("right", 0, 0);
    PropertyValuesHolder holder4 = PropertyValuesHolder.ofInt("bottom", 0, 0);
    PropertyValuesHolder holder5 = PropertyValuesHolder.ofFloat("translationX", 0, 500,0);
    Animator animator1 = ObjectAnimator.ofPropertyValuesHolder(mLinear, holder1,holder2,holder3,holder4,holder5);
    layoutTransition.setAnimator(LayoutTransition.CHANGE_APPEARING, animator1);
    mLinear.setLayoutTransition(layoutTransition);  

  使用CHANGE_APPEARING来指定了其他条目在新条目加入时的动画，不过跟指定APPEARING动画不同的是，这类动画必须使用PropertyValuesHolder来构建，并且left，top必须要指定，否则无效。这是需要注意的一点。