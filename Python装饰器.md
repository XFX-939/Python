# Python装饰器

### **装饰器引入**

假如现在在一个公司，有A B C三个业务部门，还有S一个基础服务部门，目前呢，S部门提供了两个函数，供其他部门调用，函数如下：

```python
def f1():
  print('f1 called')
 
 
def f2():
  print('f2 called')
```

在初期，其他部门这样调用是没有问题的，随着公司业务的发展，现在S部门需要**对函数调用假如权限验证**，如果有权限的话，才能进行调用，否则调用失败。考虑一下，如果是我们，该怎么做呢？

#### 方案：

- 让调用方也就是ABC部门在调用的时候，先主动进行权限验证❌
  - 问题：将本不该暴露给外层的权限认证，暴露在使用方面前，同时如果有多个部门呢，要每个部门每个人都要周知到，你还不能确定别人一定会这么做，不靠谱。。。
- S部门在对外提供的函数中，首先进行权限认证，然后再进行真正的函数操作❌
  - 问题：看似看行，可是当S部门对外提供更多的需要进行权限验证方法时，每个函数都要调用权限验证，同样也实在费劲，不利于代码的维护性和扩展性

#### 为了解决以上问题，我们引入装饰器

```python
def w1(func):
  def inner():
    print('...验证权限...')
    func()
 
  return inner
 
 
@w1
def f1():
  print('f1 called')
 
 
@w1
def f2():
  print('f2 called')
 
 
f1()
f2()

输出结果：
...验证权限...
f1 called
...验证权限...
f2 called

可以通过代码及输出看到，在调用f1 f2 函数时，成功进行了权限验证，那么是怎么做到的呢？其实这里就使用到了装饰器，通过定义一个闭包函数w1，在我们调用函数上通过关键词@w1，这样就对f1 f2函数完成了装饰。
```

#### 装饰器原理

首先，开看我们的**装饰器函数w1，该函数接收一个参数func**，其实就是接收一个方法名，w1内部又定义一个函数inner，**在inner函数中增加权限校验，并在验证完权限后调用传进来的参数func**，同时w1的返回值为内部函数inner，其实就是一个闭包函数。

然后，再来看一下，在**f1上增加@w1**，那这是什么意思呢？**当python解释器执行到这句话的时候，会去调用w1函数，同时将被装饰的函数名作为参数传入**(此时为f1)，根据闭包一文分析，在执行w1函数的时候，此时直接把inner函数返回了，同时把它赋值给f1，此时的f1已经不是未加装饰时的f1了，而是指向了w1.inner函数地址。

接下来，**在调用f1()的时候，其实调用的是w1.inner函数**，**那么此时就会先执行权限验证，然后再调用原来的f1()，该处的f1就是通过装饰传进来的参数f1。**

这样下来，就完成了对f1的装饰，实现了权限验证。



#### **两个装饰器执行流程和装饰结果**

```python
def makeBold(fun):
  print('----a----')
 
  def inner():
    print('----1----')
    return '<b>' + fun() + '</b>'
 
  return inner
 
 
def makeItalic(fun):
  print('----b----')
 
  def inner():
    print('----2----')
    return '<i>' + fun() + '</i>'
 
  return inner
 
 
@makeBold
@makeItalic
def test():
  print('----c----')
  print('----3----')
  return 'hello python decorator'
 
 
ret = test()
print(ret)

输出结果：

----b----
----a----
----1----
----2----
----c----
----3----
<b><i>hello python decorator</i></b>
```

**可以发现，先用第二个装饰器(makeItalic)进行装饰，接着再用第一个装饰器(makeBold)进行装饰，而在调用过程中，先执行第一个装饰器(makeBold)，接着再执行第二个装饰器(makeItalic)。**



修饰和调用的顺序要注意区别，修饰是下面的装饰器先，调用是上面的先。

为什么呢，分两步来分析一下。

1. 装饰时机 通过上面装饰时机的介绍，我们可以知道，在执行到@makeBold的时候，需要对下面的函数进行装饰，此时解释器继续往下走，发现并不是一个函数名，而又是一个装饰器，这时候，@makeBold装饰器暂停执行，而接着执行接下来的装饰器@makeItalic，接着把test函数名传入装饰器函数，从而打印'b'，在makeItalic装饰完后，此时的test指向makeItalic的inner函数地址，这时候有返回来执行@makeBold，接着把新test传入makeBold装饰器函数中，因此打印了'a'。
2. 在调用test函数的时候，根据上述分析，此时test指向makeBold.inner函数，因此会先打印‘1‘，接下来，在调用fun()的时候，其实是调用的makeItalic.inner()函数，所以打印‘2‘，在makeItalic.inner中，调用的fun其实才是我们最原声的test函数，所以打印原test函数中的‘c‘，‘3‘，所以在一层层调完之后，打印的结果为<b><i>hello python decorator</i></b> 。



### **对有参函数进行装饰**

```python
def w_say(fun):
  """
  如果原函数有参数，那闭包函数必须保持参数个数一致，并且将参数传递给原方法
  """
 
  def inner(name):
    """
    如果被装饰的函数有行参，那么闭包函数必须有参数
    :param name:
    :return:
    """
    print('say inner called')
    fun(name)
 
  return inner
 
 
@w_say
def hello(name):
  print('hello ' + name)
 
 
hello('wangcai')

输出结果：
say inner called
hello wangcai
```





### 对带返回值的函数进行装饰

```python
def w_test(func):
  def inner():
    print('w_test inner called start')
    func()
    print('w_test inner called end')
  return inner
 
 
@w_test
def test():
  print('this is test fun')
  return 'hello'
 
 
ret = test()
print('ret value is %s' % ret)

输出结果为：

w_test inner called start
this is test fun
w_test inner called end
ret value is None
```

可以发现，此时，并没有输出test函数的‘hello',而是None，那是为什么呢，可以发现，在inner函数中对test进行了调用，但是没有接受不了返回值，也没有进行返回，那么默认就是None了，知道了原因，那么来修改一下代码：

```python
def w_test(func):
  def inner():
    print('w_test inner called start')
    str = func()
    print('w_test inner called end')
    return str
 
  return inner
 
 
@w_test
def test():
  print('this is test fun')
  return 'hello'
 
 
ret = test()
print('ret value is %s' % ret)

输出结果：
w_test inner called start
this is test fun
w_test inner called end
ret value is hello
```