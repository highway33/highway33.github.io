---
layout: post
title: "a better way to change font size for android app"
date: 2018-03-12 16:04:32
categories: android fontsize setting
---
app在设置中一般都有改变字体大小这项需求。由rxjava订阅模式实现，借助BehaviorSubject消息源，当用户更改字体大小时，通过BehaviorSubject向所有订阅消息的TextSizeView发送缩放倍数，从而改变字体大小。
新建一个字体类
{% highlight ruby %}
public class TextSizeView extends android.support.v7.widget.AppCompatTextView implements Observer{
    Disposable mDisposable;

    float orignSize;
    public TextSizeView(Context context) {
        super(context);
        init();
    }

    public TextSizeView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public TextSizeView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    public void init(){
        //获得原始字体大小
        orignSize=getTextSize();
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        TextSizeChangeUtil.getInstance().subscribe(this);
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        if(mDisposable!=null){
        mDisposable.dispose();
        }
        mDisposable=null;
    }

    @Override
    public void onSubscribe(Disposable d) {
        mDisposable=d;
    }

    @Override
    public void onNext(Object o) {
        //以字体原始值为基数＊字体缩放倍数  重新设置字体大小
        setTextSize(TypedValue.COMPLEX_UNIT_PX,orignSize*(float)o);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
}
{% endhighlight %}

消息源单列类
{% highlight ruby %}
public class TextSizeChangeUtil {
    public static  BehaviorSubject mSubject;
    public static BehaviorSubject getInstance(){
        synchronized (BehaviorSubject.class){
            if(mSubject==null){
                mSubject=BehaviorSubject.createDefault((float)1.0);
            }
        }
        return mSubject;
    }
}
{% endhighlight %}

只要在需要改变字体大小的地方使用TextSizeView，把用户选择的缩放倍数调用TextSizeChangeUtil.getInstance().onNext（）发送给订阅者，即可实现app内改变字体大小的功能。


