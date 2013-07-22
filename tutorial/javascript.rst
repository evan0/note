提交表单
=========
如果表单中带有提交按钮，在点击 ``Enter`` 时就会提交表单。
如果没有提交按钮， ``Enter`` 不会提交表单。
一个例外是 ``textarea`` ，在框里回车是换行。

所谓提交按钮，也就是
``input[type=submit]`` 和 ``input[type=image]`` 和 ``button[type=submit]`` 。

:code:`form.submit()` 不会触发 :code:`submit` 事件。






定时器
=======
js 里有 :code:`setTimeout` 和 :code:`setInterval` 两种定时器，
前者是超时调用，后者是间歇调用，这个不用多说。

那么，下面的代码，两者的区别在哪里？

.. code:: javascript

    var doSomething = function() {};

    setTimeout(function repeat() {
        doSomething();
        setTimeout(repeat, delay);
    }, 10);

    setInterval(function repeat() {
        doSomething();
    }, 10);

两者看起来都是每 10 毫秒调用函数一次，但在具体执行的时候，还是存在细微的差异。

首先，我们知道 js 在是单线程的。
这里我们忽略调度的细节，把要执行的函数想象成一个任务队列。


:code:`setTimeout` 在 10 毫秒后调用函数，也就是把函数加入了任务队列。
如果队列中没有其他代码在执行或等待，结果就和预期的一样。
如果队列中有很多其他函数正在等待，那么这个超时调用就要慢慢排队，
等待的时间就有可能超过 10 毫秒。
执行后，会再次尝试在 10 毫秒后调用函数，也就是重复前面的过程。

那么，假设函数本身要执行 11 毫秒，
实际的执行间隔可能是这样的： `21(32),42(53),63(74),92(103)` ，
括号里面是函数执行完毕的时间，
可能两次正好就隔 10 毫秒，可能要排队多等一会儿。


:code:`setInterval` 则是每隔 10 毫秒，就尝试调用函数一次。
也就是，10 毫秒时试一次，20 毫秒时试一次，30 毫秒时试一次……
如果没碰上排队，就这样了。
如果碰上要排队的情况，也就是没能调用函数，函数就加入任务队列，等待执行。
特别的是，可能出现超过时间间隔，函数还在排队的情况。
如果 10 毫秒加入队列的函数在 20 毫秒时还没执行，
20 毫秒时的函数是不会加入队列的，也就是说，
:code:`setInterval` 要调用的函数，不会在队列中出现两次。

一样假设函数执行要 11 毫秒，
实际情形可能是这样： `21(32),40(51),60(71),88(99),100(111)` ，
比较一下还是能看出区别的。

**总结** 就是，两者的区别在于对这个间隔的处理。
其实就像函数名暗示的那样，
一个是结束之后多少毫秒加入队列，一个是每多少毫秒加入队列。





new
====
我们使用一个函数作为构造函数（constructor），来 :code:`new` 一下。
下面解释下 :code:`new` 的时候，都干了什么。

.. code:: javascript

    function Example(args) {
        this.blahblah = args;
    }

    example = new Example('wtf');

在上面这个例子里，构造函数 :code:`Example` 没有 :code:`return` 语句，
而且里面引用了 :code:`this` ，那么 :code:`example` 到底是什么呢。

实际上， :code:`new` 会先构造一个空对象（ :code:`{}` ），
在这个空对象上执行构造函数（就是把这个对象绑定到构造函数的 :code:`this` 上），
最后返回这个对象。

.. code:: javascript

    example = {};
    Example.call(example, 'wtf');

就是上面这种感觉吧。

如果这个构造函数带有 :code:`return` 语句会怎么样？

.. code:: javascript

    function Ex() {
        return 'wtf';
    }

    ex = new Ex();
    console.log(ex);

    ex2 = {}
    console.log(Ex.call(ex2));

看了上面的代码，估计也能猜出来了一点。
使用 :code:`new` 的时候，返回值是被无视的，返回的是新生成的那个对象。

最后， :code:`new A` 和 :code:`new A()` 的效果是一样。
只能说，:code:`new` 和构造函数以及括号，三者是个整体，
如果插入括号改变运算优先级，会改变整个语句的语义。