---
layout: post
title: "3 camera show and change position for each oter"
date: 2018-09-04 16:04:32
categories: android camera change position
---
公司要实现三个摄像头可以切换位置，并实现推流。开始的想法是摄像头在3个位置变化就行，试了之后发现摄像头在解绑定，重新绑定，在预览之后，造成切换画面延时太严重。后面想到一个取巧的办法，直接改变预览画面预览param不是就大功告成了。点击摄像头预览view之后，它只能和长宽param最大的view交换位置，即它们的LayoutParam交换。下面是切换的关键代码：
{% highlight ruby %}
     private void changeCamera(int index) {
        if (surfaceViewList.get(index).getLayoutParams() == mMainLayoutParams) {
            return;
        }

        BaseSurfaceView surfaceView = getMainParamsView(index);
        surfaceView.setLayoutParams(surfaceViewList.get(index).getLayoutParams());
        surfaceViewList.get(index).setLayoutParams(mMainLayoutParams);

        surfaceView.bringToFront();
        surfaceView.getParent().requestLayout();
        surfaceView.getParent().bringChildToFront(surfaceView);
        surfaceView.invalidate();
    }
再说一下我踩到的坑，开始我是用surfaceview来做显示，切换之后总是少一个摄像头view，但是可以点击，我觉得是显示的问题，摸索了很久，选择用textvure来做显示，终于可以把问题解决了，从我所描述的大家应该清楚了surfaceview跟textvure的一个区别

github项目源码地址
[项目源码地址][jekyll-docs]

[jekyll-docs]: https://github.com/highway33/3camera.git
