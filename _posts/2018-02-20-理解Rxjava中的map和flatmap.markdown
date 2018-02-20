---
layout: post
title: "rxjava map和flatmap使用"
date: 2018-02-20 16:04:32
categories: Rxjava map flatmap
---
本文所举列子代码参考链接https://www.jianshu.com/p/52cd2d514528
打印学生选的课程
map实现
{% highlight ruby %}
List<Student> students = new ArrayList<Student>();
        students.add...
        ...

        Action1<List<Course>> action1 = new Action1<List<Course>>() {
            @Override
            public void call(List<Course> courses) {
                //遍历courses，输出cuouses的name
                 for (int i = 0; i < courses.size(); i++){
                    Log.i(TAG, courses.get(i).getName());
                }
            }
        };
        Observable.from(students)
                .map(new Func1<Student, List<Course>>() {
                    @Override
                    public List<Course> call(Student student) {
                        //返回coursesList
                        return student.getCoursesList();
                    }
                })
                .subscribe(action1);

{% endhighlight %}

flatmap实现
{% highlight ruby %}
List<Student> students = new ArrayList<Student>();
        students.add...
        ...

        Observable.from(students)
                .flatMap(new Func1<Student, Observable<Course>>() {
                    @Override
                    public Observable<Course> call(Student student) {
                        return Observable.from(student.getCoursesList());
                    }
                })
                .subscribe(new Action1<Course>() {
                    @Override
                    public void call(Course course) {
                        Log.i(TAG, course.getName());
                    }
                });
{% endhighlight %}
使用map订阅者接收到的是List<Course>，而使用flatmap订阅者接受到的course，从源码来分析为什么会这样
map中的onnext方法
{% highlight ruby %}
actual.onNext(v);
{% endhighlight %}
flatmap中的onnext方法
{% highlight ruby %}
subscribeInner(p);
{% endhighlight %}
map只是简单把学生的课程传给下一个要处理的观察者的onnext方法
而flatmap把学生的课程给打印课程的订阅者重新订阅了一遍
