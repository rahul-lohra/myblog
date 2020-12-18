---
layout: post
title:  "Flutter: Create Circular Reveal Animation"
date:   2019-05-19 01:13:01 +0530
categories: Animation
---

![Demo Application](https://cdn-images-1.medium.com/max/2400/1*3hH6MTj1WF2O8IekzHNUdQ.gif)

Breaking the animation:
1. The radius of the circle is increasing : ***AnimationController***
2. Drawing of the circle : ***CustomPainter***
3. There is also a fade out animation : ***AnimationController***

Tap Behaviour (For simplicity: I will consider very minimal case)
1. Tap down: Radius starts to increase
2. Tap Up/Tap Cancel : Alpha is going from 40 to 0. (default alpha is 40 and alpha is of the reveal background)

## Classes & their responsibilities:

### RevealPaint: 
1. Draw circle on our widget
2. Draw background with **ARGB** colour.

### RevealAnimationController:
1. Manage expand animation and fadeOut animation
2. Send callback to ***State (required to frequently draw on UI)***

### State class:
1. It will have reference of the Paint class, and it will give the **arguments** such as **alpha** value and **radius** to the paint class, and Paint class will just draw stuff.
2. It will obviously manage **tapUp and tapDown behaviou**r- which will tell when to start **expand animation** and when to start **fade out animation**
> NOTE: The code is bit **huge**, I will only explain the very basic and core-concepts:
> Full source code: [https://github.com/rahul-lohra/circular_reveal_widget](https://github.com/rahul-lohra/circular_reveal_widget)

## Code:
```
class RevealPaint extends CustomPainter {
  double fraction = 0.0;
    double radius = 0;
    Paint mPaint;
    Offset offset;
    int alpha = RevealAnimationController.MAX_ALPHA;
    String revealColor = "F2F3F4";

    RevealPaint() {
      mPaint = Paint();
      mPaint.color = HexColor("#$alpha$revealColor");
      mPaint.style = PaintingStyle.fill;
      offset = Offset(0, 0);
    }

    setAlpha(int alpha){
      this.alpha = alpha;
      mPaint.color = HexColor("#${alpha}$revealColor");
    }

    @override
    void paint(Canvas canvas, Size size) {
      double finalRadius = (radius * fraction);
      canvas.drawCircle(offset, finalRadius, mPaint);
    }

    @override
    bool shouldRepaint(RevealPaint oldDelegate) {
      return true;
    }
}
```

Drawing is happening on `paint()` function. **Radius** and **fraction** values will be initialised dynamically — we will see later.

We need code to ***start and stop*** the **expand and fade** animation. So we have `RevealAnimationController` here, it manages them

```
class RevealAnimationController{
  Animation<double> tweenExpand;  //contain animated value
  Animation<double> tweenFade;    //contain animated value
  VoidCallback tweenFadeCallback; //they are listeners to the animation
  VoidCallback tweenExpandCallback;
  AnimationController expandAnimationController;    
  AnimationController fadeAnimController;
  RevealAnimationController(
      TickerProviderStateMixin mixin, ControllerCallback callback) {
  expandAnimationController = AnimationController(
      duration: const Duration(milliseconds: 800), vsync: mixin);
  fadeAnimController = AnimationController(
      duration: const Duration(milliseconds: 300), vsync: mixin);

  tweenFade =
      Tween<double>(begin: MAX_ALPHA.toDouble(), end: MIN_ALPHA.toDouble())
          .animate(fadeAnimController);

  tweenExpand = Tween<double>(begin: MIN_TWEEN_VALUE, end: MAX_TWEEN_VALUE)
      .animate(animaController);
  }
  startAnimation() {
      expandAnimationController.forward();
  }
  startFadeOutAnim() {
      fadeAnimController.forward();
  }
  resetExpandAnim() {    
      expandAnimationController.reset();
  }
  _prepareTweenCallback() {
    tweenFadeCallback = () {
        int val = tweenFade.value.toInt();
        callback.onAlphaUpdate(val);    
    };

    tweenExpandCallback = () {
      fraction = tweenExpand.value;
        if (tweenExpand.value == MAX_TWEEN_VALUE) {
          callback.onAnimationComplete(fraction);
        } else {
          callback.onAnimationUpdate(fraction, alpha);
        }
    };
  }
}
```
Now we need a way to tell out `Paint` class to draw circle with some **colours and alpha** value

Now we have a `ControllerCallback`(custom made) **interface, it will basically give updates to our State class. And this State class will** force the Paint class to re-draw stuffs on our widget.

```
class RevealState extends State
    with TickerProviderStateMixin, ControllerCallback {
  Widget child;
  RevealAnimationController controller;
  RevealPaint revealPaint;
  RevealState(Widget child) {
    this.child = child;
    controller = RevealAnimationController(this, this);
  }
  @override
  Widget build(BuildContext context) {
    List<Widget> widgetList = List();
    widgetList.add(child);

    if (enableReveal) {
      revealPaint = RevealPaint();
      revealPaint.radius = max(renderBoxSize.height, renderBoxSize.width) + 10;
      revealPaint.fraction = controller.fraction;
      revealPaint.setAlpha(controller.alpha);
      revealPaint.offset = offset;

      widgetList.add(revealWidget(renderBoxSize, revealPaint));
    }

    return getTapWidget(widgetList);
  }
  Widget revealWidget(Size _size, CustomPainter paint) {
    return ClipRRect(
      borderRadius: BorderRadius.zero,
      child: CustomPaint(
        size: _size,
        painter: paint,
      ),
    );
  }
}
```
Inside the build functions we are **setting arguments our `Paint` class**

In the above class we have some callbacks

```
  @override
  void onAnimationUpdate(double fraction, int alpha) {
    setState(() {
      controller.fraction = fraction;
    });
  }

  @override
  void onAnimationComplete(double fraction) {
    setState(() {
      controller.fraction = fraction;
    });
  }

  @override
  void onAlphaUpdate(int alpha) {
    setState(() {
      controller.alpha = alpha;
    });
  }
```

We are calling `setState` — It will force to re-render the `widget`, means the `build` function will be called again and again.

Now how to **start or stop** expand and fade **animations**, — We will `GestureDetector` for this

```
Widget getTapWidget(List<Widget> widgetList) {
  return GestureDetector(
    child: Stack(children: widgetList),
    onLongPressMoveUpdate: (e) {
      handleFingerMove(e.globalPosition.distance);
    },
    onHorizontalDragUpdate: (e){
      handleFingerMove(e.globalPosition.distance);
    },
    onVerticalDragUpdate: (e){
      handleFingerMove(e.globalPosition.distance);
    },
    onTapUp: (e){
      handleTapUp();
    },
    onLongPressUp: (){
      handleTapUp();
    },
    onTapDown: (e) {
      handleTapDown(e);
    },
  );
}
void handleFingerMove(double pos) {
  //Codes are removed for simplicity
    handleTapUp();
}

void handleTapDown(TapDownDetails e) {
  setState(() {
  //Codes are removed for simplicity
    startAnimation();
  });
}
void handleTapUp() {
  controller.stopAnimation();
}

void startAnimation() {
  controller.startAnimation();
}
```   

That’s it. check the readme of the library. Its very simple

**Finally, this is our library’s widget**

```
class RevealWidget extends StatefulWidget{
  Widget child;
  RevealWidget(this.child);
  @override
  State<StatefulWidget> createState() {
    return RevealState(child);
  }
}
```

### **That’s how we use it.**

```
class ButtonWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RevealWidget(Container(
        color: Colors.red,
        width: 100,
        height: 60,
        child: Align(
          alignment: Alignment.center,
          child: Text(
            "Hello",
            style: TextStyle(fontSize: 20),
          ),
        )));
  }
}
```

> Full source code: [https://github.com/rahul-lohra/circular_reveal_widget](https://github.com/rahul-lohra/circular_reveal_widget)

**Package: [https://pub.dev/packages/circular_reveal](https://pub.dev/packages/circular_reveal)**
