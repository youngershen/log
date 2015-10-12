python中的formatter的详细用法
=============================

今天抽空学习了一下python中的string service中的formatter的相关用法，主要是为了让自己的代码看起来更加和谐，因为很多java或者c语言过来的开
发者都不怎么爱使用python的原生的字符串格式化工具，似乎大家都爱用下面的格式化工具::    

    info = 'my name is %s I really enjoy %s' % ('younger', 'python')
    
现在我要学习使用更加python化的字符串格式化风格。

python的buildin字符串服务模块 `string <file:///Users/youngershen-mac-book-pro/Downloads/python-2.7.8-docs-html/library/string.html>`_         身提供了一些字符串操作的工具类方法，其中包括一些经常会使用到的常量，和比较复杂的Formatter类，Template类，这里我要重点学习的就是string.

Formatter类
-----------

`string.Formatter <file:///Users/youngershen-mac-book-pro/Downloads/python-2.7.8-docs-html/library/string.html#string-formatting>`_ 类中的方法::

    format(format_string, *args, **kwargs)
    
format方法是string.Formatter类中的主要方法，它的参数是一个你需要去格式化的目标字符串，和一组需要去填充目标字符串的序列，比如字典和元组，format方法是对vformat方法的封装。
    
上面是我照文档的说明写的，其实Formatter.format方法和str.format并没有什么不同，他们的语法是通用的，只要学会一种就都会了，哪个更方便就用哪个::

    vformat(format_string, args, kwargs)
    parse(format_string)
    get_field(field_name, args, kwargs)
    get_value(key, args, kwargs)
    
上面的这几个方法是互相调用的，所以一放在一起研究，其中Formatter.format 最终调用的是Formatter.vformat方法，测试程序如下::
    
    class Person(object):
        name  = 'default name'
        def __init__(self, name = ''):
            self.name = name

    me = Person('younger')
    data = [me]
    s = "my name is {person[0].name:^30}"
    formatter = String.Formatter()
    formatter.format(s, data)
    
输出结果::

    >>> formatter.format(s, person = data)
    'my name is            younger            '
    
现在用Formatter.vformat方法::

    formatter.vformat(s, (), {'person' : data})

输出结果::

    >>> formatter.format(s, person = data)
    'my name is            younger            '

2个方法的结果是完全一样的，只是一个包装了另一个，再Formatter.vformat方法中必须有4个参数，中间的空元组和最后的空字典必须存在，因为Formatter.vformat的参数不是 (*args, **kwargs) 而是 (args, kwargs), 这是完全不一样的，没看清楚会悲剧的。
    
现在继续说上面4个方法的调用顺序， get_value调用了get_field, get_field调用parse, vformat调用了  get_value, 一般情况下我们只需要调用format就足够了，上面的4个方法都是给需要继承Formatter创建自己的格式化语法的时候来覆盖掉的，不过我们可以从这4个方法分析出很多东西。
    
我现在执行下面的程序::

    for i, v in enumerate(formatter.parse(s, start = 0)):
        print i, v

结果会输出::

    0 ('my name is ', 'person[0].name', '^30', None)

上面是返回的第1个编号为0的元组， 这4个值分别是::

    (literal_text, field_name, format_spec, conversion)

我们没有canversion, 所以第4项是None, 你也可以写上一个!r或者!s这样的话字符串就变成了::

    s = "my name is {person[0].name!r:^30}"

前两项都很好理解，直接看字面就懂什么意思，第三项的意思是说格式化的时候的站位符，我这里用的是空，你也可以用星号，现在把s变成这样::

    s ="my name is {person[0].name!r:*^30}"

输出的结果是::

    "my name is **********'younger'***********"
    *号充当了占位符的作用
    
现在执行下面的程序::

    for i ,v in enumerate(formatter.parse(s)):
        temp = v
    
    formatter.get_field(temp[1], (), {'person' : data})

输出结果为::

    ('younger', 'person')
    
用文档中的话来说就是 'object'和'used_key', 这个key和get_value中的key是相同的所以要调用get_value必须先调用get_field下面继续执行代码::

    ret = formatter.get_value(formatter.get_field(temp[1], (), {'person':data})[1], (), {'person':data})

这个返回的 ret就是得到的对象，一个Person类型的list ,里面只有一个对象就是最初我们填充的那个，到此为止所有的方法都跑了一遍，如果我们要改写自己的format语法，那就直接继承这个类，覆盖这么几个方法就行了，其余的2个方法很容易理解，可以直接看文档。
    
Formatter.format的语法
----------------------

这里就不采用文档里的论道的方法来说明了，直接以我的理解来说好了只有keyword类型的format string
    
最简单的::

    "my name is {name}".format(name = 'younger')
    
带有多个组合条件的::

    "my name is {person[0].name!r:*^30}".format(person = data_list)
    
上面的意思是说传入的是一个list，list中有person， keyword是person[0].name, 很好理解，就是第0个对象的name属性， 这样用起来很方便， 非常好记， 从!r开始的奇怪语法是 Format Specification Mini-Language , 其中对一些数据类型，比如百分数，正负数,大数，时间，小数点的位数，以及格式化format string的站位符等进行了定义，用法都和我写的例子一样，没有什么复杂的，这里例子已经算是比较复杂的例子了。
    
只有position类型的format string::

    "my name is {0.name}".format(person)

同时有position和keyword的formart string::

    "my name is {0.name}, I am living at {area[0].city}".format(person, area_list)
    
这里要注意的就是position的必须写再前面，不然是不能使用的，推荐大家一个格式化字符串里只用以个方式去写。
    
    
    
    