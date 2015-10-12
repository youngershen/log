PYTHON中函数参数的pack与unpack
==============================

上周在使用django做开发的时候用到了mixin（关于mixin我还要写一个博客专门讨论一下，现在请参见这里），其中又涉及到了一个关于函数参数打包(pack)的问题，导致延误了开发时间，所以在这里记录一下，稍后会说到具体的背景。

背景交代
--------

具体情景是这样的，我需要一个view可以在查询的同时可以分页，又可以在返回的 queryset 上做更多的查询操作。为了解决这个问题，我自己写了一个mixin ::

    class  MultipleOjbectQueryPageMixin(object):
        '''
        query_params = [['filter , 'tags__label__contains, 'wow' ],[],[]]
        '''
        query_params = None
        paginate_by = None
        page_size_kwargs = 'size'
        
        #新加入的方法
        def get_paginate_by(self, queryset):
            if self.page_size_kwargs in self.request.kwargs:
                self.paginate_by = int(self.request.kwargs[self.page_size_kwargs])
                return self.paginate_by
         
        def do_query(self, queryset):
            if self.query_params:
                iterator = iter(self.query_params)
                try:
                    param = iterator.next()
                    try:
                        if params[1] is not None and params[0] is not None:
                            #函数参数解包的问题
                            queryset = getattr(queryset, params[0])(**{params[1] : params[2]})          
                            return queryset
                        else:
                            queryset = getattr(queryset,params[0])()
                            return queryset
                    except AttributeError as e:
                        print e
                except  StopIteration as e:
                    print e
            else:
                return queryset.all()       
                
        def get_queryset(self):
            if self.queryset is not None:
                queryset = self.queryset
            if isinstance(queryset, QuerySet):
                queryset = self.do_query(queryset)
            elif self.model is not None:
                queryset = self.do_query(self.model._default_manager)
            else:
                raise ImproperlyConfigured(
                    "%(cls)s is missing a QuerySet. Define"
                    "%(cls)s.model, %(cls)s.queryset, or override"
                    "%(cls)s.get_queryset()." % {
                        'cls' : self.__class__.__name__
                    }
                )
            ordering = self.get_ordering()
            if ordering:
                if isinstance(ordering, six.string_types):
                    ordering = (ordering, )
                    queryset = queryset.order_by(*ordering)
            return queryset

这个 mixin只自定义了2个方法，其中一个是从url中获取当前分页的页面大小，也就是page_size, 在MultipleObjectMixin类中这这个参数的名字叫做 paginate_by ,在 django.views.generic.ListView 中直接继承了这个mixin , 不过ListView只有基本的分页功能 (你可以直接在url中传入/?page=1,来进行分页，mixin中都写好了)，并没有可以从url中获取页面大小，我增加的方法可以直接从url中 获取要分页的页面大小，然后传入page参数就可以完成分页。

另外一个方法比较复杂，它拥有了基本的查询功能，我的初衷是要从url中得到博客的标签信息，然后根据标签查询到相应标签下的博客，所以就写了一个方法，这个方法由 get_queryset(self) 这个方法来调用， 

这样就可以比较方便的在已有代码的基础上去做查询，实际使用中我写了一个功能类似于BaseListView类的View类来继承该mixin， 然后重写自己的get方法就可以了，在myapp.views中使用起来非常方便，直接继承自己写的view类，然后在类中定义需要的属性，就可以 了，myapp.views中没有方法，看起来非常整洁，我认为这就是CBV的好处了。

如果你要说我直接自己写get方法，直接在request中获取tag的值，然后::

    pk = request.kwargs['id']
    tag = Tag.objects.get(pk = pk)
    blogs = tag.blogs.all()

那么我会说这完全是可以的，但是你以后需要再查询别的东西的时候，你必须一直写get方法，这样的写法，会导致CBV的优势荡然无存，那还不如直接去写FBV的好，大家都说FBV容易理解，其实我觉得让我写的话肯定是写CBV，不喜欢看到乱糟糟的代码。

函数参数的解包问题：
-------------------

由于已经很久很久没有写python了，所以不免会忘记一些东西，这次跳的一个小坑就是在函数参数上出了问题。
我想实现的是这样的效果::

    params = { 'tags__label__conatains' : wow'}
    Blog.objects.filter(params)

上面这样直接传入字典是不行的,因为filter中的参数是一个有默认值的tags__table__contains 参数，我的目的是想给它一个值，这样在func(*args, **kwargs)里是解析不到kwargs里的，实际上传入一个字典的结果都在args 参数里，因为我们传入的是一个对象，而不是kv, 只有传入test(a=1, b=2)这样的值才会被解析到kwargs中，现在我们这样写::

    Blog.objects.filter(params.keys()[0] = params.values()[0])

还是不行, 接着我们这样写::

    k = params.keys()[0]
    v = params.values()[0]
    Blog.objects.filter( k = v)

你会发现这样写可以，但是我们需要的参数变成k了，而不是 tags__label__contains所以这样也是不行的。然后我们又换了一种方法::

    Blog.objects.Filter({ k : v})

结果这个参数又跑到args里去了，最后正确的写法是这样的::

    Blog.objects.Filter(**{k:v})

这样先解包就可以把带变量的参数传到kwargs, 对于args来说是一样的::

    def test(*args, **kwargs):
        print args
        print kwargs
     
    s = (1,2,3,4,)
    test(s)
    输出：
    ((1,2,3,4),)

很明显系统把s这个元组当成了一个对象，如果你打算传入之后对，args进行遍历操作话，会发现args里只有一个对象，但是我明明传入了一个有4个元素的元祖。
正确的写法是::

    test(*s)

输出为::

    (1,2,3,4)

这个时候你就可以去遍历了，绝对没问题了