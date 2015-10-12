为什么0[0]在javascript中不报语法错
==============================================

偶然在 `一篇帖子 <http://stackoverflow.com/questions/29250950/why-is-00-syntactically-valid>`_  中看到了这个问题，所以打算记录一下::

    var a = 0[0];
    console.log(a);

    output:
    a is undefined

上面这段代码就是问题的题干，非常简单的2行代码，问题的难点是是否能搞清楚 0[0] 这行代码到底干了什么。对于不熟悉js的人来说,会把 0[0]当成是数组索引，按照java或者c的经验，如果去索引一个 integer 类型的 0 ,是必然不行的，但是在js当中，没有任何异常发生，非常平滑的输出了 undefined, 其实这是一个js中的语法糖，但是又有一点障眼法的感觉，分析如下.

当进行 0[0] 运算的时候，js解析器会把第一个 0 当做一个 Number类的对象(自动装箱)，然后去尝试获取方括号中的名称为  0 的属性，然而  Number 0 中必然是没有名字叫做 0 的属性的，所以得出的结果就是 undefined, 并且没有丝毫的语法错误.说它是障眼法，是因为根据惯例，如果我们要取一个对象中的属性的话，都是去用 . 这个符号进行索引某个对象的属性，但是在js当中，使用 [] 与使用 . 是一样的效果，所以说它是一个语法糖::

    var a = 0.0;
    console.log(a);

上面的代码和开篇的代码运行结果完全相同，js的这个特性可以和ruby进行类比，在ruby之中也是可以这么做的，这种语法叫做完全面向对象，貌似ES6中正在加强这种特性，理论上来说这也是在自动进行装箱操作，不过具体是如何解析代码的，就必须要去读源码才能知晓.

ES5.1中的获取对象属性的语法说明::

    Literal ::
        NullLiteral
        BooleanLiteral
        NumericLiteral
        StringLiteral
        RegularExpressionLiteral

    PrimaryExpression :
        this
        Identifier
        Literal
        ArrayLiteral
        ObjectLiteral
        ( Expression )

    MemberExpression :
        PrimaryExpression
        FunctionExpression
        MemberExpression [ Expression ]
        MemberExpression . IdentifierName
        new MemberExpression Arguments    

所以我们可以这样获取属性::

    MemberExpression [ Expression ]
    PrimaryExpression [ Expression ]
    Literal [ Expression ]
    NumericLiteral [ Expression ]

根据5.1的文档，我们这样使用 0[0] 也就是完全说的通的，完全没有任何语法错误::

    NumericLiteral [ NumericLiteral ]
    数字字面量[数字字面量]

关于js中的自动装箱，可以参见 `这篇文当 <https://javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/>`_ ,这篇文当主要讲述的就是js中的一般字面量在进行属性索引操作的时候，是如何自动装箱的，关于ES5的文档，可以参见 `这里 <http://www.ecma-international.org/ecma-262/5.1/>`_ ,

下面的代码也可以从侧面说明问题::

    0['toString']
    output: function toString() { [native code] }

(全文完)