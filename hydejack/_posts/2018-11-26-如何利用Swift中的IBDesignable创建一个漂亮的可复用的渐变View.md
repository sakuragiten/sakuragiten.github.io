---
layout: post
title: 如何利用Swift中的IBDesignable创建一个漂亮的可复用的渐变View
description: >
excerpt_separator: <!--more-->
---

### 如何利用Swift中的IBDesignable创建一个漂亮的可复用的渐变View

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxi32ztfg7j30sg0lcdgm.jpg)

接下来讲述如何用swift4创建一个通用的并且带有`@IBDesignable`的gradient View类, 你可以在stroyboard中直接拖拽CAGradientView并且能及时看到效果，或者你也可以通过代码的形式添加。然后设置开始和结束两个点对应的色值已经颜色渐变的方向，当然，这些属性可以在XIB中直接控制。



#### 为什么需要这个

不可否认的是设计师喜欢渐变图层，类似的还有毛玻璃背景以及阴影，时尚流行的风格不断变化，现如今，他们对这些更加敏感，所以他们需要大量的调试来使效果变得更好。

创建一个渐变图层需要一定量的工作，不断的调试，直到设计师满意为止。这是一个很费时间的过程，所以，本文将告诉你如何通过storyboard直接拖拽一个渐变图层，并且调试完直接显示出效果。

这样的话，你将省掉很多重复编译运行的时间。



### 接下来要做什么
我们先列举需要注意的事情

* 需要继承 UIView
* 需要实现 `@IBDesignable` 功能，这样的话就可以通过storyboard直接调试并显示出效果
* 需要兼容纯代码或者XIB两种创建方式


### [获取Demo工程](https://github.com/leedowthwaite/LDGradientView)

运行工程，在storyboard中找到ViewController对应的XIB，你将可以Inspector中编辑对应的属性值，效果就像上面放了一张图片一样。

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxi53aki1ej30xx0oudjc.jpg)


### 关于渐变图层
__注意：__ 这不是`CAGradientLayer`的文档，如果你想了解更多基础的介绍，可以参考这篇文章[ Mastering CAGradientLayer in Swift](https://appcodelabs.com/ios-gradients-how-use-cagradientlayer-swift)

iOS中实现渐变效果的方式有很多，本文主要使用`CALayer`的子类`CAGradientLayer`，这也是核心动画中视图层级结构中的一个重要对象。在iOS中，UIView的内容显示主要是layer层来控制，每一个UIView对象都一个对应的layer，同时，就像每一个UIView对象都可以有多个subView对象一样，每一个layer对象也可以有多个sublayer对象。

而实际应用过程中，我们就是将这些错综复杂的sublayers添加到响应的View上。在深入了解Core Animation的时候，开发者同样需要知道怎样在layer层实现与View层同样的效果。通常，Views和layers的区别主要在事件响应上，而一般app都需要这些具备交互功能性的控件，比如说 UILabel， UIButton。

当我们创建一些精致的图形时，layer的层级关系就会变得很复杂，我们最好是避免让这些图层变得很复杂，毕竟这些layers不能用故事版来操作，只能通过纯代码，所以理论上来讲，处理图层将会变得很复杂。。这里将引导你将一个简单 `CAGradientLayer `对象作为sublayer添加到view的layer层上，这里只是一对一的关系,所以，也可以在storyboard里将layer添加到view里的效果展示出来。


### 定义View的子类
这里将创建`LDGradientView `作为UIView的子类，定义方式如下:

```

@IBDesignable class LDGradientView: UIView { 
  // ... 
}


```

用`@IBDesignable`标记， 表明可以直接在storyboard里编辑


而渐变层本身将作为类的私有属性

```
// the gradient layer 
private var gradient: CAGradientLayer?

```

这个成员变量将通过下面的函数创建。这里设置了gradient的`frame`为对应View的bounds，让其填充整个View，这样就可以保证图层与视图界面的一对一关系。

```
// create gradient layer 
private func createGradient() -> CAGradientLayer { 
  let gradient = CAGradientLayer() 
  gradient.frame = self.bounds
  return gradient 
}

```
然后将创建的gradient layer作为子视图添加到对应View的layer层上。

```
// Create a gradient and install it on the layer 
private func installGradient() { 
  // if there's already a gradient installed on the layer, remove it
  if let gradient = self.gradient {
    gradient.removeFromSuperlayer()
  } 
  let gradient = createGradient()
  self.layer.addSublayer(gradient)
  self.gradient = gradient
}
```

这些都是私有函数，因为view的layer层级应当由它自己来处理。

如果你在一个很复杂的图层上创建gradient或者父视图使用了约束，那么每次在设置view的frame的时候都需要刷新一下它里面的子图层，代码如下：

```
override var frame: CGRect {
  didSet {
    updateGradient()
  }
}
override func layoutSubviews() {
  super.layoutSubviews()
  // this is crucial when constraints are used in superviews
  updateGradient()
}
// Update an existing gradient
    private func updateGradient() {
        if let gradient = self.gradient {
            let startColor = self.startColor ?? UIColor.clear
            let endColor = self.endColor ?? UIColor.clear
            gradient.colors = [startColor.cgColor, endColor.cgColor]
            let (start, end) = gradientPointsForAngle(self.angle)
            gradient.startPoint = start
            gradient.endPoint = end
            gradient.frame = self.bounds
        }
    }

```

最后，我们也需要初始化方法来创建渐变图层，而实现初始化方法主要通过代码和故事版两种创建方式。

```
// initializers 
required init?(coder aDecoder: NSCoder) {
  super.init(coder: aDecoder)
  installGradient() 
} 
override init(frame: CGRect) {
  super.init(frame: frame)
  installGradient() 
}
```

### 定义一个渐变层 Gradient

现在只是有了一个能创建`CAGradientLayer`的UIView的子类，这还不能满足我们所有的需求，所以就需要定义一下gradient来满足需求。

`CAGradientLayer`主要有两个重要的属性需要外界控制。

* 渐变层的颜色
* 渐变层的方向

### 定义颜色
先给`CAGradientLayer`添加一个colors属性

```
// An array of CGColorRef objects defining the color of each gradient stop. Animatable.
var colors: [Any]?
```

### 渐变层的控制点
控制颜色变化的点称为颜色站点(gradient stops)。 渐变层可以有很多个颜色站点来支持复杂的图形变化，
而颜色站点添加的过程会十分的繁琐，而站点的数量又不定，所以解决起来很困难。在代码里直接一点点的重复调试作用效果不明显。也正是因为这样的原因，这里将创建一个开始为一种颜色结束为另一种颜色的"简单"图层。当然你也可以自己添加其它的颜色站点。

所以 colors属性的实现很简单:

```
// the gradient start colour 
@IBInspectable var startColor: UIColor? 
// the gradient end colour 
@IBInspectable var endColor: UIColor?

```
这些属性也可以再XIB上控制

### 定义方向
渐变层的变化方向主要由`CAGradientLayer`的两个属性控制

```
// 结束点
var endPoint: CGPoint
// 开始点
var startPoint: CGPoint
```

gradient的开始结束点定义在渐变空间单元(__unit gradient space__)

这就意味着无论给定的`CAGradientLayer`对象的大小是多少，我们都可以用下面的草图来表示。我们假定左上角的坐标为(0, 0), 右下角的坐标为(1, 1)

<div align=center>![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxla4iddwzj308c08ca9u.jpg)
<div align=left>

用XIB来控制graident方向是一件很费劲的事情，因为 `@IBInspectable` attibuetes不支持CGPoint类型，但也不意味着完全没有数据来支持UI，只是我们的选择会有限制。最终在XIB上我们使用弧度来代替CGPoint。

```
//逆时针方向 从0开始
@IBInspectable var angle: CGFloat = 270
```

这里是270作为`CAGradientLayer`的默认渐变方向，方向为从下往上。如果想要设置为水平方向，那么angle的值可以设置成0或180

### 将angle转换成CGPoint



```
private func gradientPointsForAngle(_ angle: CGFloat) -> (CGPoint, CGPoint) {
// 获取方向的开始点
  let end = pointForAngle(angle)
  let start = oppositePoint(end)
  // convert to gradient space
  let p0 = transformToGradientSpace(start)
  let p1 = transformToGradientSpace(end)
  return (p0, p1)
  }

```

弧度指定了颜色渐变的方向从0度开始，0度为向右的方向，按逆时针方向递增，如图所示
<div align=center>![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxlb04cl6xj309t09cglq.jpg)
<div align=left>

获取颜色渐变方向的结束点

```
private func pointForAngle(_ angle: CGFloat) -> CGPoint {
  // 将度数转换成弧度
  let radians = angle * .pi / 180.0
  var x = cos(radians)
  var y = sin(radians)
  // (x,y) 在单位圆内. 
  if (fabs(x) > fabs(y)) {
    // 假设x为单位长度
    x = x > 0 ? 1 : -1 y = x * tan(radians)
  } else {
    // 假设y为单位长度
    y = y > 0 ? 1 : -1
    x = y / tan(radians)
  } 
  return CGPoint(x: x, y: y) 
}
```

这么做看起来很复杂，主要通过sine和cosine函数，swift中的三角函数和其它语言中一样，需要的是弧度而不是角度，假设我们知道弧度，那么`x = cos(radians)`， `y = sin(radians)`。接下来只要知道我们关心的点是否在我们的单位圆圈里，而如图所示，无论是0，90，180，270，点都在圈内

<div align=center>![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxlfo8quifj30ax09w3yq.jpg)
<div align=left>

在我们指定单位矩形内拿到结束点以后，找到开始点就很容易了，只需要把结束的点沿着y=x翻转一下就可以了

```
private func oppositePoint(_ point: CGPoint) -> CGPoint {
  return CGPoint(x: -point.x, y: -point.y) 
}
```

现在有了开始和结束的点，剩下的就是将它们转换到渐变空间里，单位渐变空间里有它自己的Y轴，这根Core Animation 里的维度一样。所以，上面所说的坐标点(0, 0)在新的坐标系里就是(0.5, 0.5)

```
private func transformToGradientSpace(_ point: CGPoint) -> CGPoint {
  // 输入的点从: (-1,-1) 到 (1,1)
  return CGPoint(x: (point.x + 1) * 0.5, y: 1.0 - (point.y + 1) * 0.5) 
}
```

### 支持界面调试
剩下的就是实现`prepareForInterfaceBuilder()`函数，这个方法仅仅通过XIB调试需要重新渲染界面的时候执行

```
override func prepareForInterfaceBuilder() { 
  super.prepareForInterfaceBuilder()
  installGradient()
  updateGradient() 
}

```






