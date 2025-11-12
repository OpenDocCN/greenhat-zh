# 第八章。传递参数和返回值

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

在本章中，你将了解许多传递参数和从方法返回值的效果（以及副作用）。首先，我将花一点时间总结一下你到目前为止所使用的方法类型。

# 总结实例、类和单例方法

实例方法是在类定义内部声明的，并旨在由特定的对象或类的“实例”使用，如下所示：

*methods.rb*

```
class MyClass
    # declare instance method
    def instanceMethod
        puts( "This is an instance method" )
    end
end

    # create object
ob = MyClass.new
    # use instance method
ob.instanceMethod
```

类方法可以在类定义内部声明，在这种情况下，方法名可能前面有类名，或者一个 `class << self` 块可以包含一个“正常”的方法定义。无论哪种方式，类方法都是为类本身使用，而不是为特定的对象使用，如下所示：

```
class MyClass
    # a class method
    def MyClass.classmethod1
        puts( "This is a class method" )
    end

    # another class method
    class << self
        def classmethod2
             puts( "This is another class method" )
        end
    end
end

    # call class methods from the class itself
MyClass.classmethod1
MyClass.classmethod2
```

单例方法是添加到单个对象中的方法，不能被其他对象使用。单例方法可以通过将方法名附加到对象名后跟一个点来定义，或者将一个“正常”的方法定义放在一个 *`ObjectName`* `<< self` 块中，如下所示：

```
# create object
ob = MyClass.new

    # define a singleton method
def ob.singleton_method1
    puts( "This is a singleton method" )
end

    # define another singleton method
class << ob
    def singleton_method2
        puts( "This is another singleton method" )
    end
end

    # use the singleton methods
ob.singleton_method1
ob.singleton_method2
```

# 返回值

在许多编程语言中，区分了返回值给调用代码的函数或方法和不返回值的函数或方法。例如，在 Pascal 中，一个 *函数* 会返回一个值，但一个 *过程* 不会。Ruby 中没有这样的区分。所有方法总是返回一个值，尽管当然你不必使用它。

当未指定返回值时，Ruby 方法返回最后评估的表达式的结果。考虑这个方法：

*return_vals.rb*

```
def method1
    a = 1
    b = 2
    c = a + b   # returns 3
end
```

最后评估的表达式是 `a + b`，它恰好返回 3，因此这是此方法返回的值。可能经常会有你不想返回最后评估的表达式的情况。在这种情况下，你可以使用 `return` 关键字指定返回值：

```
def method2
    a = 1
    b = 2
    c = a + b
    return b   # returns 2
end
```

方法不一定要进行任何赋值以返回一个值。如果简单数据恰好是方法中最后评估的内容，那么它将是方法返回的值。如果没有内容被评估，则返回 `nil`：

```
def method3
   "hello"     # returns "hello"
end

def method4
   a = 1 + 2
   "goodbye"   # returns "goodbye"
end

def method5
end            # returns nil
```

我的编程偏见是尽可能编写清晰且无歧义的代码。因此，每当我计划使用方法返回的值时，我更喜欢使用 `return` 关键字来指定它；只有在我不打算使用返回值时，我才省略这个关键字。然而，这并非强制性的——Ruby 将选择权留给了你。

# 返回多个值

但是，当你需要一种方法返回多个值时怎么办？在其他程序语言中，你可能可以通过传递引用（原始数据项的指针）而不是值（数据的副本）来实现“伪造”这一功能；当你改变“引用”参数的值时，你会改变原始值，而无需明确地将任何值返回给调用代码。

Ruby 不会区分“按引用”和“按值”，因此这项技术对你不可用（尽管你很快就会看到一些规则的例外）。然而，Ruby 能够一次性返回多个值，如下所示：

*return_many.rb*

```
def ret_things
    greeting = "Hello world"
    a = 1
    b = 2.0
    return a, b, 3, "four", greeting, 6 * 10
end
```

多个返回值被放入一个数组中。如果你要评估 `ret_things.class`，Ruby 会告诉你返回的对象是一个数组。然而，你可以显式地返回不同的集合类型，例如一个哈希表：

```
def ret_hash
    return {'a'=>'hello', 'b'=>'goodbye', 'c'=>'fare thee well'}
end
```

# 默认值和多个参数

Ruby 允许你为参数指定默认值。默认值可以在方法参数列表中使用常规赋值运算符进行分配：

```
def aMethod( a=10, b=20 )
```

如果将未分配的变量传递给该方法，则将分配默认值。但是，如果传递已分配的变量，则分配的值将优先于默认值。在这里，我使用 `p()` 方法来检查和打印返回值：

```
def aMethod( a=10, b=20 )
   return a, b
end

p( aMethod )            #=> displays: [10,  20]
p( aMethod( 1 ))        #=> displays: [1, 20]
p( aMethod( 1, 2 ))     #=> displays: [1, 2]
```

在某些情况下，一个方法可能需要能够接收不确定数量的参数——比如，例如处理可变长度项目列表的方法。在这种情况下，你可以通过在最后一个参数前加上星号来“清除”任何数量的尾随项：

*default_args.rb*

```
def aMethod( a=10, b=20, c=100, *d )
   return a, b, c, d
end

p( aMethod( 1,2,3,4,6 ) )     #=> displays: [1, 2, 3, [4, 6]]
```

# 赋值和参数传递

大多数情况下，Ruby 方法有两个访问点——就像进入和离开房间的门。参数列表提供了进入的方式；返回值提供了离开的方式。对输入参数所做的修改不会影响原始数据，简单的理由是当 Ruby 评估一个表达式时，该评估的结果会创建一个新的对象，因此对参数所做的任何更改只会影响新的对象，而不会影响原始数据。但这个规则也有例外，我现在会向你展示。

让我们从最简单的情况开始看起——一个接受一个命名参数并返回另一个值的方法：

*in_out.rb*

```
def change( x )
    x += 1
    return x
end
```

表面上看，你可能会认为你在这里处理的是一个单一的对象 `x`：对象 `x` 进入 `change` 方法，并且相同的对象 `x` 被返回。实际上并非如此。一个对象进入（参数），另一个不同的对象出来（返回值）。你可以通过使用 `object_id` 方法来轻松验证这一点，该方法显示一个唯一标识程序中每个对象的数字：

```
num = 10
puts( "num.object_id=#{num.object_id}" )
num = change( num )
puts( "num.object_id=#{num.object_id}" )
```

变量标识符 `num` 在调用 `change` 方法前后是不同的。这表明尽管变量名保持不变，但 `change` 方法返回的 `num` 对象与发送给它的 `num` 对象是不同的。

方法调用本身与对象的改变无关。你可以通过运行 *method_call.rb* 来验证这一点。这仅仅是将 `num` 对象传递给 `change` 方法并返回它：

*method_call.rb*

```
def nochange( x )
   return x
end
```

在这种情况下，`object_id` 在返回 `num` 之后与发送到方法之前相同。换句话说，进入方法的对象与再次出来的对象是同一个对象。这导致了一个不可避免的结论，那就是 `change` 方法（`x += 1`）中的 `assignment` 导致了新对象的创建。

但赋值本身并不是全部的解释。如果你只是将一个变量赋值给自己，不会创建新的对象：

*assignment.rb*

```
num = 10
num = num     # a new num object is not created
```

如果你现在显示 `num` 变量的 `object_id`，赋值前后数字相同，这证明了这确实是一个相同的对象。那么，如果你将对象赋值为它已经拥有的相同值呢？

```
num = 10
num = 10      # a new num object is not created
```

再次强调，赋值后 `object_id` 没有改变。这表明仅赋值本身并不一定会创建新对象。现在让我们尝试赋一个新的值：

```
num = 10
num += 1      # this time a new num object is created
```

这次，如果你在赋值前后显示 `num.object_id`，你会看到不同的数字——比如说赋值前是 21，赋值后是 23。实际的数字是由 Ruby 自动确定的，可能不同。重要的是要理解，不同的对象 ID 表示不同的对象。如果相同的变量在赋值时返回不同的 `object_id`，这意味着已经创建了新的对象。

大多数数据项被视为唯一的，所以一个字符串“hello”被认为与另一个字符串“hello,”不同，一个浮点数 10.5 被认为与另一个浮点数 10.5 不同。因此，任何字符串或浮点数的赋值都会创建一个新的对象。

但当与整数一起工作时，只有当赋值值与之前的值不同时，才会创建新对象。你可以在赋值号的右侧进行所有种类的复杂操作，但如果产生的值与原始值相同，则不会创建新对象。

```
num = 11
puts( "num.object_id=#{num.object_id}" )
num = (((num + 1 - 1) * 100) / 100)
puts( "num.object_id=#{num.object_id}" )
```

在前面的代码中，第一次赋值创建了一个具有整数值 11 的新 `num` 对象。即使下一个赋值使用了相当复杂的表达式的结果，这个值仍然是 11。由于 `num` 的值没有改变，因此没有创建新的对象，它的 `object_id` 保持不变：

```
num.object_id=23
num.object_id=23
```

# 整数是特殊的

在 Ruby 中，整数（或 Fixnum）有一个固定的身份。每个数字 10 的实例或被赋予值 10 的变量都将具有相同的 `object_id`。这不能适用于其他数据类型。例如，10.5 这样的浮点数或“hello world”这样的字符串的每个实例都将是一个具有唯一 `object_id` 的不同对象。请注意，当你将整数赋给变量时，该变量将具有整数的 `object_id`。但当你将其他类型的数据赋给变量时，即使每次赋值的数据本身相同，也会创建一个新的对象：

*object_ids.rb*

```
# 10 and x after each assignment are the same object
puts( 10.object_id )
x = 10
puts( x.object_id )
x = 10
puts( x.object_id )

# 10.5 and x after each assignment are 3 different objects!
puts( 10.5.object_id )
x = 10.5
puts( x.object_id )
x = 10.5
puts( x.object_id )
```

但为什么这一切都这么重要？

这很重要，因为有一些罕见的例外情况。正如我之前所说的，大多数时候，一个方法有一个明确的进入方式和一个明确的退出方式。一旦参数进入方法，它就进入了一个封闭的房间。任何在该方法之外的外部代码都无法了解对参数所做的任何更改，直到它以返回值的形态再次出现。这实际上是“纯”面向对象的一个深层次秘密。方法的具体实现细节应该，原则上，被隐藏起来，或者*封装*。这确保了对象之外的外部代码不能依赖于对象内部发生的事情。

# 单向进入，单向退出原则

在大多数现代面向对象的语言，如 Java 和 C#中，封装和信息隐藏并没有得到严格的强制执行。另一方面，在 Smalltalk——最著名和最有影响力的面向对象语言中——封装和信息隐藏是基本原理：如果你将变量`x`发送到方法`y`，并且`x`的值在`y`中被更改，你无法从方法外部获得`x`的更改后的值——*除非方法明确返回该值*。

封装或信息隐藏？

通常这两个术语是互换使用的。然而，要吹毛求疵的话，它们之间还是存在差异。*封装*指的是将对象的“状态”（其数据）和可能改变或查询其状态的运算（其方法）组合在一起。*信息隐藏*指的是数据被封闭起来，只能通过定义良好的进出路径访问——在面向对象术语中，这暗示了“访问器方法”来获取或返回值。在过程式语言中，信息隐藏可能采取其他形式；例如，你可能必须定义接口从代码“单元”或“模块”而不是从对象中检索数据。

在面向对象的概念中，封装和信息隐藏几乎是同义的——真正的封装必然意味着对象的内部数据是隐藏的。然而，许多现代面向对象的语言，如 Java、C#、C++和 Object Pascal，在强制执行信息隐藏的程度（如果有的话）上相当宽容。

通常，Ruby 遵循这一原则：参数进入方法，但方法内部所做的任何更改都无法从外部访问，除非 Ruby 返回更改后的值：

*hidden.rb*

```
def hidden( aStr, anotherStr )
    anotherStr = aStr + " " + anotherStr
    return anotherStr.reverse
end

str1 = "dlrow"
str2 = "olleh"
str3 = hidden(str1, str2)
puts( str1 )    #=> dlrow
puts( str2 )    #=> olleh
puts( str3 )    #=> hello world
```

在之前的代码中，第二个对象`str2`的字符串值被接收为`hidden`方法的`anotherStr`参数。该参数被分配了一个新的字符串值并反转。即便如此，原始变量`str1`或`str2`都没有改变。只有被分配返回值`str3`的变量包含了更改后的“hello world”字符串。

结果表明，在某些情况下，传递给 Ruby 方法的参数可以像其他语言的“按引用”参数一样使用（也就是说，在方法内部做出的更改可能会影响方法外部的变量）。这是因为一些 Ruby 方法修改了原始对象，而不是返回一个值并将其分配给新对象。

例如，有一些以感叹号结尾的方法会改变原始对象。同样，String 的追加方法`<<`将右侧的字符串连接到左侧的字符串，但在过程中不会创建新的字符串对象：因此，左侧字符串的值被修改，但字符串对象本身保留了其原始的`object_id`。

这种做法的后果是，如果你在方法中使用`<<`运算符而不是`+`运算符，你的结果将会改变：

*not_hidden.rb*

```
def nothidden( aStr, anotherStr )
    anotherStr = aStr << " " << anotherStr
    return anotherStr.reverse
end

str1 = "dlrow"
str2 = "olleh"
str3 = nothidden(str1, str2)
puts( str1 )      #=> dlrow olleh
puts( str2 )      #=> olleh
puts( str3 )      #=> hello world
```

在前面的代码中，`anotherStr`参数使用`<<`与`aStr`参数连接，并使用`<<`返回的结果字符串被反转。如果严格实施信息隐藏，这可能会产生与上一个程序相同的结果，即`str1`和`str2`将保持不变。然而，使用`<<`产生了深远的影响，因为它导致在`nothidden`方法内部对`aStr`参数所做的修改改变了方法外部的`str1`对象的值。

顺便说一下，如果将`nothidden`方法放入一个单独的类中，这种行为也会相同：

*nothidden2.rb*

```
class X
    def nothidden( aStr, anotherStr )
        anotherStr = aStr << " " << anotherStr
        return anotherStr.reverse
    end
end
ob = X.new
str1 = "dlrow"
str2 = "olleh"
str3 = ob.nothidden(str1, str2)
puts( str1 )    #=> dlrow olleh
```

这表明，在某些情况下，对象方法的内部实现细节可能会意外地改变调用它的代码。通常，隐藏实现细节更安全；否则，当类内部重写代码时，这些更改可能会对使用该类的代码产生副作用。

# 修改接收者并返回新对象

你可能还记得，在第四章中，我区分了修改其接收者的方法和不修改其接收者的方法。（记住，*接收者*是“拥有”方法的对象。）在大多数情况下，Ruby 方法不会修改接收者对象。然而，一些方法，如以`!`结尾的方法，确实会修改它们的接收者。

*str_reverse.rb* 示例程序应该有助于阐明这一点。这表明，当你使用`reverse`方法时，例如，不会对接收者对象（即`str1`这样的对象）做出任何更改。但是，当你使用`reverse!`方法时，对象（其字母顺序被反转）会发生变化。即便如此，也不会创建新对象：在调用`reverse!`方法之前和之后，`str1`仍然是同一个对象。

在这里，`reverse`方法像大多数 Ruby 方法一样工作：它返回一个值，为了使用这个值，你必须将它分配给一个新的对象。考虑以下情况：

*str_reverse.rb*

```
str1 = "hello"
str1.reverse
```

在这里，`str1` 调用 `reverse` 后不受影响。它仍然具有值“hello”和它的原始 `object_id`。现在看看这个：

```
str1 = "hello"
str1.reverse!
```

这次，`str1` 被改变了（变成了“olleh”）。即便如此，也没有创建新的对象：`str1` 仍然具有它开始时的相同的 `object_id`。那么，看看这个：

```
str1 = "hello"
str1 = str1.reverse
```

这次，`str1.reverse` 产生的值被分配给了 `str1`。产生的值是一个新对象，所以 `str1` 现在分配了反转的字符串（“olleh”），并且它现在有一个新的 `object_id`。

参考示例程序 *concat.rb*，了解字符串连接方法 `<<` 的示例，它就像以 `!` 结尾的方法一样，修改接收器对象而不创建新对象（再次强调，当你运行代码时实际的 `object_id` 数字可能不同）：

*concat.rb*

```
str1 = "hello"          #object_id = 23033940
str2 = "world"          #object_id = 23033928
str3 = "goodbye"        #object_id = 23033916
str3 = str2 << str1
puts( str1.object_id )  #=> 23033940 # unchanged
puts( str2.object_id )  #=> 23033928 # unchanged
puts( str3.object_id )  #=> 23033928 # now the same as str2!
```

在这个例子中，`str1` 永远没有被修改，所以它始终具有相同的 `object_id`；`str2` 通过连接操作被修改。然而，`<<` 操作符不会创建一个新的对象，所以 `str2` 也保留了其原始的 `object_id`。

但 `str3` 在结束时与开始时是不同的对象，因为它被分配了这个表达式的值：`str2 << str1`。这个值恰好是 `str2` 对象本身，所以 `str3` 的 `object_id` 现在与 `str2` 的相同（也就是说，`str2` 和 `str3` *现在引用了同一个对象*）。

总结来说，那么，以 `!` 结尾的方法，如 `reverse!`，以及一些其他方法，如 `<<` 连接方法，会改变接收器对象的值。大多数其他方法不会改变接收器对象的值。要使用调用这些方法之一产生的新值，你必须将该值赋给一个变量（或将产生的值作为参数传递给一个方法）。

### 注意

事实是，只有少数方法会修改接收器对象，而大多数方法则不会，这看起来可能无害，但请注意：这种行为为你提供了通过“引用”检索参数值的能力，而不是检索显式返回的值。这样做会破坏封装，允许你的代码依赖于方法的内部实现细节。这可能导致不可预测的副作用，在我看来，应该避免。

# 依赖参数值而非显式返回值的潜在副作用

对于一个简单（但在现实世界的编程中可能严重的）例子，说明依赖参数的修改值而不是显式返回值如何引入对实现细节的不希望依赖，请参阅 *side_effects.rb*。这里有一个名为 `stringProcess` 的方法，它接受两个字符串参数，对它们进行操作，并返回结果：

*side_effects.rb*

```
def stringProcess( aStr, anotherStr )
    aStr.capitalize!
    anotherStr.reverse!.capitalize!
    aStr = aStr +  " " + anotherStr.reverse!
    return aStr
end
```

假设练习的目标是取两个小写字符串，并返回一个将这两个字符串结合起来的单个字符串，用空格分隔，并且首尾字母大写。所以，两个原始字符串可能是“hello”和“world”，返回的字符串是“Hello worlD”。这没问题：

```
str1 = "hello"
str2 = "world"
str3 = stringProcess( str1, str2 )
puts( "#{str3}" )    #=> Hello worlD
```

但现在有一个急躁的程序员不愿意处理返回值。他注意到方法内部所做的修改改变了传入参数的值。所以，他心想！（他决定），他干脆使用这些参数本身！这是他的版本：

```
puts( "#{str1} #{str2}" ) #=> Hello worlD
```

通过使用输入参数的值`str1`和`str2`，他已经得到了与使用返回值`str3`相同的结果。然后他离开并编写了一个极其复杂的文本处理系统，有成千上万的代码依赖于这两个参数的改变值。

但现在，最初编写`stringProcess`方法的程序员决定原始实现效率低下或不够优雅，因此重新编写了代码，并确信返回值没有改变（如果发送“hello”和“world”作为参数，将返回“Hello worlD”，就像上一个版本一样）：

```
def stringProcess( aStr, anotherStr )
    myStr = aStr.capitalize!
    anotherStr.reverse!.capitalize!
    myStr = myStr + " " + anotherStr.reverse
    return myStr
end

str1 = "hello"
str2 = "world"
str3 = stringProcess( str1, str2 )

puts( "#{str3}" )             #=> Hello worlD
puts( "#{str1} #{str2}" )     #=> Hello Dlrow
```

哎！但新的实现导致方法体内输入参数的值被改变。所以，那个急躁的程序员依赖这些*参数*而不是返回值的文本处理系统，现在充满了“hello Dlrow”这样的文本片段，而不是他预期的“Hello worlD”（实际上，他的程序正在处理莎士比亚的作品，所以一代演员最终会大声朗诵，“To eb or ton to eb, that si the noitseuq...”）。这是可以很容易避免的意外副作用，只要遵循单向输入、单向输出的原则。

# 并行赋值

我之前提到，一个方法可以返回多个值，这些值通过逗号分隔。通常，你希望将这些返回的值分配给一组匹配的变量。

在 Ruby 中，你可以通过并行赋值在单个操作中完成这个操作。这意味着你可以有多个左侧变量或赋值运算符，以及多个右侧值。右侧的值将按顺序分配给左侧的变量，如下所示：

*parallel_assign.rb*

```
s1, s2, s3 = "Hickory", "Dickory", "Dock"
```

这种能力不仅为你提供了快速进行多次赋值的方法；它还允许你交换变量的值（你只需改变赋值运算符两侧的顺序）：

```
i1 = 1
i2 = 2

i1, i2 = i2, i1        #=> i1 is now 2, i2 is 1
```

你还可以从方法返回的值中进行多次赋值：

```
def returnArray( a, b, c )
    a = "Hello, " + a
    b = "Hi, " + b
    c = "Good day, " + c
    return a, b, c
end
x, y, z = returnArray( "Fred", "Bert", "Mary" )
```

如果你指定赋值号右侧的值比左侧的变量多，任何“剩余”的变量将被赋值为`nil`：

```
x, y, z, extravar = returnArray( "Fred", "Bert", "Mary" ) # extravar = nil
```

方法返回的多个值被放入一个数组中。当你将数组放在多个变量赋值的右侧时，其各个元素将被分配给每个变量，并且如果提供的变量太多，额外的变量将被分配`nil`：

```
s1, s2, s3 = ["Ding", "Dong", "Bell"]
```

深入挖掘

在本节中，我们探讨了一些参数传递和对象相等性的内部工作原理。我还讨论了括号对于代码清晰性的价值。

通过引用还是通过值？

之前我说过，Ruby 在“通过值”传递的参数和“通过引用”传递的参数之间没有区别。即便如此，如果你在互联网上搜索，很快就会发现 Ruby 程序员经常就参数的确切传递方式发生争论。在许多过程式编程语言，如 Pascal 或 C 中，传递值或引用的参数之间存在明显的区别。

“通过值”的参数是原始变量的“副本”；你可以将它传递给一个函数并对其进行操作，而原始变量的值保持不变。

相反，“通过引用”的参数是原始变量的“指针”。当这个指针传递给一个过程时，你传递的不是一个新的副本，而是指向存储原始数据的内存块的引用。因此，在过程中所做的任何更改都会影响原始数据，并必然影响原始变量的值。

那么，Ruby 是以哪种方式传递参数的呢？实际上，解决这个问题非常简单。如果 Ruby 通过值传递，那么它会复制原始变量，因此这个副本将具有不同的`object_id`。实际上，情况并非如此。尝试运行`*arg_passing.rb*`程序来证明这一点。

`*arg_passing.rb*`

```
def aMethod( anArg )
    puts( "#{anArg.object_id}\n\n" )
end

class MyClass
end

i = 10
f = 10.5
s = "hello world"
ob = MyClass.new

puts( "#{i}.object_id = #{i.object_id}" )
aMethod( i )
puts( "#{f}.object_id = #{f.object_id}" )
aMethod( f )
puts( "#{s}.object_id = #{s.object_id}" )
aMethod( s )
puts( "#{ob}.object_id = #{ob.object_id}" )
aMethod( ob )
```

该程序在原始声明时以及将它们作为`aMethod()`方法的参数传递时，打印出整数、浮点数、字符串和自定义对象的`object_id`。在每种情况下，参数的 ID 与原始变量的 ID 相同，因此参数必须是通过引用传递的。

现在，在某种情况下，参数的传递可能“幕后”被“实现”为“通过值”。然而，这样的实现细节应该对 Ruby 解释器和编译器的编写者而不是 Ruby 程序员感兴趣。简单的事实是，如果你以“纯”面向对象的方式编程——通过将参数传递给方法，但只随后使用这些方法返回的值——那么实现细节（通过值或通过引用）对你来说将无关紧要。

尽管如此，由于 Ruby 有时可以修改参数（例如，使用 `!` 方法或 `<<`，如 修改接收者和返回新对象 中所述），一些程序员养成了使用参数自身修改后的值的习惯（相当于在 C 中使用按引用参数），而不是使用返回的值。在我看来，这是一种不良做法。它使你的程序依赖于方法的实现细节，因此应该避免。

作业是副本还是引用？

我之前说过，当某个表达式 *yield* 一个值时，会创建一个新的对象。所以，例如，如果你将新值赋给一个名为 `x` 的变量，赋值后的对象将与赋值前的对象不同（即，它将具有不同的 `object_id`）：

```
x = 10      # this x has one object_id
x +=1       # and this x has a different one
```

但创建新对象的不是赋值操作，而是产生的值。在上面的例子中，`+=1` 是一个产生值的表达式（`x+=1` 等价于表达式 `x=x+1`）。

简单地将一个变量的值赋给另一个变量不会创建一个新的对象。所以，假设你有一个名为 `num` 的变量和另一个名为 `num2` 的变量。如果你将 `num2` 赋值给 `num`，这两个变量将引用同一个对象。你可以使用 Object 类的 `equal?` 方法来测试这一点：

*assign_ref.rb*

```
num = 11.5
num2 = 11.5

    # num and num 2 are not equal
puts( "#{num.equal?(num2)}" )    #=> false

num = num2
    # but now they are equal
puts( "#{num.equal?(num2)}" )    #=> true
```

相等性测试：== 还是 equal？

默认情况下（如 Ruby 的 `Kernel` 模块中定义的），使用 `==` 进行测试，当被测试的两个对象是同一个对象时返回 `true`。因此，如果值相同但对象不同，则返回 `false`：

*equal_tests.rb*

```
ob1 = Object.new
ob2 = Object.new
puts( ob1==ob2 ) #=> false
```

事实上，`==` 经常被 String 等类覆盖，然后当值相同但对象不同时返回 `true`：

```
s1 = "hello"
s2 = "hello"
puts( s1==s2 ) #=> true
```

因此，当你想要确定两个变量是否引用同一个对象时，`equal?` 方法更可取：

```
puts( ob1.equal?(ob2) )    #=> false
puts( s1.equal?(s2) )      #=> false
```

何时两个对象是相同的？

作为一般规则，如果你用 10 个值初始化 10 个变量，每个变量将引用不同的对象。例如，如果你这样创建两个字符串：

*identical.rb*

```
s1 = "hello"
s2 = "hello"
```

然后 `s1` 和 `s2` 将引用独立的对象。对于两个浮点数也是一样：

```
f1 = 10.00
f2 = 10.00
```

但是，如前所述，整数是不同的。创建具有相同值的两个整数，它们最终会引用同一个对象：

```
i1 = 10
i2 = 10
```

这甚至适用于字面整数值。如果有疑问，请使用 `equals?` 方法来测试两个变量或值是否引用了完全相同的对象：

```
10.0.equal?(10.0) # compare floats - returns false
10.equal?(10)     # compare integers (Fixnums) - returns true
```

括号避免歧义

方法可能与局部变量有相同的名称。例如，你可能有一个名为 `name` 的变量和一个名为 `name` 的方法。如果你习惯于不带括号调用方法，那么可能不清楚你是在指方法还是变量。再次强调，括号可以避免歧义：

*parentheses.rb*

```
greet = "Hello"
name = "Fred"

def greet
    return "Good morning"
end

def name
    return "Mary"
end

def sayHi( aName )
    return "Hi, #{aName}"
end

puts( greet )                 #=> Hello
puts greet                    #=> Hello
puts greet()                  #=> good morning
puts( sayHi( name ) )         #=> Hi, Fred
puts( sayHi( name() ) )       #=> Hi, Mary
```
