---
layout: post
title:  "Shadow Constraint Layout with Round Corners"
date:   2019-11-02 01:13:01 +0530
categories: Android
---
`ConstraintLayout` with its own shadow and we will apply round corners around it just like `CardView`

Few Good things: 
1. No need to put `ConstraintLayout` inside `CardView` anymore 
2. You can draw custom shadows as well
3. Use same technique with any view

**Prequisites** (Good if you know these things otherwise don’t worry):
1. `Canvas` x,y corrdinate
2. `PorterDuffXfermode`
3. `Paint` class
4. `Path` class

**Things we need to do**
1. Draw shadow
2. Draw a border around the view
3. Clip the view with RoundRorners
4. Fill your view with white color: so you can see the border & shadow & you must draw this **over your shadow paint**

## Draw shadow around your view

Classes required
1. `Paint` : to paint with shadow color
2. `Path` : To give a path to shadow and that is around the view like a RECTANGLE

<script src="https://gist.github.com/rahullohra2903/9bdebadd1a56ceedc8249a5378f4db07.js" charset="utf-8"></script>

```
val porterDuffXfermode = PorterDuffXfermode(PorterDuff.Mode.SRC)
val blurMaskFilter = BlurMaskFilter(blurRadius, BlurMaskFilter.Blur.NORMAL)

shadowPaint.xfermode = porterDuffXfermode
shadowPaint.maskFilter = blurMaskFilter
```

The shadow is blurry that’s why we have used blurMaskFilter. And blurMaskFilter will only blur your color
The shadow should not overlap the content of your constraint layout that’s why we used PorterDuffXfermode

Now we setting up paint is completed. We have to give it a path and that’s a rectangle

So the we are going to write something like this. 
1. Goto to start point
2. Make a rectangle from there and at last come back from where you started

>1. shadowPath.goToStartPosition()
>2. shadowPath.move_to_top_left_from_top_right
>3. shadowPath.move_to_bottom_left_from_top_left
>4. shadowPath.move_to_bottom_right_from_bottom_left
>5. shadowPath.move_to_top_right_from_bottom_right

```
"->" denotes : From

shadowPath.moveTo(width, 0f)                    //Top Right
shadowPath.lineTo(0, 0)                         // TR -> TL
shadowPath.lineTo(0,height)                     // TL -> BL 
shadowPath.lineTo(width,height)                 // BL -> BR
shadowPath.lineTo(width, shadowStartY)          // BR -> TR            
```

Cool the paint and path are set. We just need to pass commands to draw it on canvas
```
canvas.drawPath(shadowPath, shadowPaint)
```
That’s it.. This is what you will see

<p align="center">
  <img src="https://cdn-images-1.medium.com/max/2000/1*IjOlpc-QYdzQgSwwGghiSg.png" />
  <br>
  Rectangular shadow
</p>

The toughest part is done, now only comes the easy part

## Draw a border around the view

<script src="https://gist.github.com/rahullohra2903/2f280df11927891caa9491a5d60bcc3d.js" charset="utf-8"></script>

We are using ***`canvas.drawRoundRect(rect,paint)` to draw. It requires a `Rect` and `paint` and we have provided both***

## Clip the view with RoundRorners

<script src="https://gist.github.com/rahullohra2903/248133957438cf7a069464cca2cb7f79.js" charset="utf-8"></script>

At the end we doing
```
canvas.clipPath(clipPath)
```
This means we want to clip this part(defined via clipPath) of canvas. And out future drawing commands must happen within the bounds of this *clipPath*

In real app we will do something like this

```
class MyLayout:View {
  fun override fun dispatchDraw(canvas: Canvas) {
    clipRoundCorners(canvas)
    super.dispatchDraw(canvas)
}
```

Our job is to draw round corners around our view and also ensure your child views should not be drawn on that area. So we should write this function inside dispatchDraw. We can change the canvas before a child is drawn : [https://developer.android.com/reference/android/view/ViewGroup.html#dispatchDraw(android.graphics.Canvas)](https://developer.android.com/reference/android/view/ViewGroup.html#dispatchDraw(android.graphics.Canvas))

## Fill your view with white color

<script src="https://gist.github.com/rahullohra2903/77a196b4bd12fe94e03d4355f73023ce.js" charset="utf-8"></script>

Code should look pretty straight forward.

```
canvas.drawRoundRect(rectBackgroundRectF, cornerRadius, cornerRadius, rectPaint)
```
The `paint` object has `rectPaint.xfermode = porterDuffXfermode`, to ensure the paint should come on top of shadow

Final code should be like this

```
class MyLayout:View {
    override fun dispatchDraw(canvas: Canvas) {
       clipRoundCorners(canvas)
       super.dispatchDraw(canvas)
    }
    override fun onDraw(canvas: Canvas){
       drawShadow(canvas)
       drawBorder(canvas)
       drawRectBackground(canvas)
       super.onDraw(canvas)
    }
}
```

Full source code:

<script src="https://gist.github.com/rahullohra2903/5b5cbdb41cbc2fc94fc486fb9383554c.js" charset="utf-8"></script>

<p align="center">Full code snipet and how to use</p>

1 very importatnt thing you have to do this in your parent view

    android:clipChildren="false"
    android:clipToPadding="false"

From these two commands we are telling the Android that parent view must not clip the views of ShadowConstraintLayot if the views are getting drawn outside of its bouds.

This is the final result..

![Left one is Card View & Right one is ShadowConstraintLayout](https://cdn-images-1.medium.com/max/2000/1*iGOKfGtpeKS9I67PiCi5lQ.png)

Left one is Card View & Right one is ShadowConstraintLayout

You can play with `blurRadius` property and also set shadow `top`,`bottom`,`start`,`end` `offset` to suit your needs.
