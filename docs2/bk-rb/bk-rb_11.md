# 第十一章。符号

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

许多新接触 Ruby 的人对符号感到困惑。符号是一个以冒号（`:`）开头的标识符，所以`:this`是一个符号，`:that`也是。实际上，符号并不复杂——在某些情况下，它们可能非常有用，您很快就会看到。

让我们首先明确符号不是什么：它不是一个字符串，它不是一个常量，它也不是一个变量。符号简单地说，是一个没有内在意义的标识符，除了它的名字。而您可能像这样给变量赋值 . . .

```
name = "Fred"
```

您**不会**给符号赋值：

```
:name = "Fred"    # Error!
```

符号的值就是它本身。因此，名为`:name`的符号的值是`:name`。

### 注意

对于符号的更技术性描述，请参阅深入挖掘中的深入挖掘。

当然，您之前已经使用过符号。例如，在第二章中，您通过将符号传递给`attr_reader`和`attr_writer`方法来创建属性读取器和写入器，如下所示：

```
attr_reader( :description )
attr_writer( :description )
```

您可能还记得，之前的代码导致 Ruby 创建一个`@description`实例变量以及一对名为`description`的获取器（读取器）和设置器（写入器）方法。Ruby 将符号的值视为字面量。`attr_reader`和`attr_writer`方法从该名称创建具有匹配名称的变量和方法。

# 符号和字符串

人们普遍认为符号是一种字符串类型。毕竟，符号`:hello`和字符串`"hello"`不是非常相似吗？实际上，符号与字符串截然不同。首先，每个字符串都是不同的——因此，从 Ruby 的角度来看，`"hello"`、`"hello"`和`"hello"`是三个具有三个不同`object_id`的独立对象。

*symbol_ids.rb*

```
# These 3 strings have 3 different object_ids
puts( "hello".object_id ) #=> 16589436
puts( "hello".object_id ) #=> 16589388
puts( "hello".object_id ) #=> 16589340
```

但符号是唯一的，所以`:hello`、`:hello`和`:hello`都指向具有相同`object_id`的相同对象。

```
# These 3 symbols have the same object_id
puts( :hello.object_id ) #=> 208712
puts( :hello.object_id ) #=> 208712
puts( :hello.object_id ) #=> 208712
```

在这方面，符号与整数的相似之处多于与字符串的相似之处。您可能记得，给定整数值的每个出现都指向同一个对象，所以`10`、`10`和`10`可以被认为是同一个对象，并且它们具有相同的`object_id`。记住，分配给对象的实际 ID 每次运行程序时都会改变。数字本身并不重要。重要的是要注意，每个独立的对象始终有一个唯一的 ID，因此当 ID 重复时，它表示对同一对象的重复引用。

*ints_and_symbols.rb*

```
# These three symbols have the same object_id
puts( :ten.object_id )  #=> 20712
puts( :ten.object_id )  #=> 20712
puts( :ten.object_id )  #=> 20712

# These three integers have the same object_id
puts( 10.object_id )    #=> 21
puts( 10.object_id )    #=> 21
puts( 10.object_id )    #=> 21
```

您还可以使用`equal?`方法进行相等性测试：

*symbols_strings.rb*

```
puts( :helloworld.equal?( :helloworld ) )     #=> true
puts( "helloworld".equal?( "helloworld" ) )   #=> false
puts( 1.equal?( 1 ) )                         #=> true
```

作为独特的符号，提供了一个明确的标识符。您可以将符号作为参数传递给方法，例如：

```
amethod( :deletefiles )
```

一个方法可能包含用于测试传入参数值的代码：

*symbols_1.rb*

```
def amethod( doThis )
    if (doThis == :deletefiles) then
       puts( 'Now deleting files...')
    elsif (doThis == :formatdisk) then
       puts( 'Now formatting disk...')
    else
        puts( "Sorry, command not understood." )
    end
end
```

符号也可以用于 `case` 语句中，它们提供了字符串的可读性和整数的唯一性：

```
case doThis
    when :deletefiles then puts( 'Now deleting files...')
    when :formatdisk then puts( 'Now formatting disk...')
    else  puts( "Sorry, command not understood." )
end
```

声明符号的作用域不会影响其唯一性。考虑以下情况：

*symbol_ref.rb*

```
module One
     class Fred
     end
     $f1 = :Fred
end

module Two
     Fred = 1
     $f2 = :Fred
end

def Fred()
end

$f3 = :Fred
```

在这里，变量 `$f1`、`$f2` 和 `$f3` 在三个不同的作用域中分别被赋值为符号 `:Fred`：模块 `One`、模块 `Two` 和“主”作用域。以 `$` 开头的变量是全局的，因此一旦创建，就可以在任何地方引用。我将在第十二章（Chapter 12）中详细介绍模块。现在，只需将它们视为定义不同作用域的“命名空间”即可。然而，每个变量都引用相同的符号 `:Fred`，并且具有相同的 `object_id`。

```
# All three display the same id!
puts( $f1.object_id )  #=> 208868
puts( $f2.object_id )  #=> 208868
puts( $f3.object_id )  #=> 208868
```

即使如此，符号的“含义”会根据其作用域而改变。在模块 `One` 中，`:Fred` 指的是类 `Fred`；在模块 `Two` 中，它指的是常量 `Fred = 1`；在主作用域中，它指的是方法 `Fred`。

之前程序的改写版本展示了这一点：

*symbol_ref2.rb*

```
module One
    class Fred
    end
    $f1 = :Fred
    def self.evalFred( aSymbol )
        puts( eval( aSymbol.id2name ) )
    end
end

module Two
    Fred = 1
    $f2 = :Fred
    def self.evalFred( aSymbol )
        puts( eval( aSymbol.id2name ) )
    end
end

def Fred()
    puts( "hello from the Fred method" )
end

$f3 = :Fred
```

首先，我使用两个冒号（`::`）在名为 `One` 的模块内部访问 `evalFred` 方法，这是 Ruby 的“作用域解析运算符”。然后，我将 `$f1` 传递给该方法：

```
One::evalFred( $f1 )
```

在这个上下文中，`Fred` 是在模块 `One` 内部定义的类的名称，因此当 `:Fred` 符号被评估时，会显示模块和类名：

```
One::Fred
```

接着，我将 `$f2` 传递给模块 `Two` 的 `evalFred` 方法：

```
Two::evalFred( $f2 )
```

在这个上下文中，`Fred` 是一个被赋值为整数 1 的常量，所以显示的是 `1`。最后，我调用一个名为 `method` 的特殊方法。这是一个 Object 的方法。它试图找到与作为参数传递给它的符号具有相同名称的方法，如果找到，则返回该方法作为可以调用的对象：

```
method($f3).call
```

`Fred` 方法存在于主作用域中，当被调用时，其输出是以下字符串：

```
"hello from the Fred method"
```

自然地，由于变量 `$f1`、`$f2` 和 `$f3` 引用了相同的符号，所以在任何特定时刻使用哪个变量都无关紧要。任何被符号赋值的变量，或者，实际上，符号本身，都会产生相同的结果。以下内容是等价的：

```
One::evalFred( $f1 )    #=> One::Fred
Two::evalFred( $f2 )    #=> 1
method($f3).call        #=> hello from the Fred method

One::evalFred( $f3 )    #=> One::Fred
Two::evalFred( $f1 )    #=> 1
method($f2).call        #=> hello from the Fred method

One::evalFred( :Fred )  #=> One::Fred
Two::evalFred( :Fred )  #=> 1
method(:Fred).call      #=> hello from the Fred method
```

# 符号和变量

要了解符号和如变量名这样的标识符之间的关系，请查看 *symbols_2.rb* 程序。它首先将值 1 赋给局部变量 `x`。然后，它将符号 `:x` 赋给局部变量 `xsymbol`：

*symbols_2.rb*

```
x = 1
xsymbol = :x
```

在这一点上，变量 `x` 和符号 `:x` 之间没有明显的联系。我声明了一个方法，它简单地接受一些传入的参数，并使用 `p` 方法检查和显示它。我可以使用变量和符号调用此方法：

```
def amethod( somearg )
    p( somearg )
end

# Test 1
amethod( x )
amethod( :x )
```

这是方法打印出的数据：

```
1
:x
```

换句话说，`x` 变量的值是 1，因为这是分配给它的值，`:x` 的值是 `:x`。但出现的一个有趣问题是：如果 `:x` 的值是 `:x`，这也是变量 `x` 的符号名称，那么是否可以使用符号 `:x` 来找到变量 `x` 的值？困惑了吗？我希望下一行代码会使这一点更清晰：

```
# Test 2
amethod( eval(:x.id2name))
```

这里，`id2name` 是 Symbol 类的一个方法。它返回与符号对应的名称或字符串（`to_s` 方法会执行相同的功能）；最终结果是，当给定符号 `:x` 作为参数时，`id2name` 返回字符串“x。”Ruby 的 `eval` 方法（在 Kernel 类中定义）能够评估字符串中的表达式。在本例中，这意味着它找到字符串`x`并尝试将其作为可执行代码进行评估。它发现 `x` 是一个变量的名称，`x` 的值是 1。因此，值 1 被传递给 `amethod`。你可以通过运行 *symbols2.rb* 来验证这一点。

### 注意

将数据作为代码进行评估的详细解释请参阅第二十章。

事情可能会变得更加复杂。记住，变量 `xsymbol` 已经被分配了符号 `:x`。

```
x = 1
xsymbol = :x
```

这意味着如果你 eval `:xsymbol`，你可以获得分配给它的名称——即符号 `:x`。获得 `:x` 后，你可以继续评估这个符号，得到 `x` 的值，即 1：

```
# Test 3
amethod( xsymbol )                                      #=> :x
amethod( :xsymbol )                                     #=> :xsymbol
amethod( eval(:xsymbol.id2name))                        #=> :x
amethod( eval( ( eval(:xsymbol.id2name)).id2name ) )    #=> 1
```

正如你所看到的，当用于创建属性访问器时，符号可以引用方法名。你可以通过将方法名作为符号传递给 `method` 方法，然后使用 `call` 方法调用指定的方法来利用这一点：

```
#Test 4
method(:amethod).call("")
```

`call` 方法允许你传递参数，所以，只是为了好玩，你可以通过评估一个符号来传递一个参数：

```
method(:amethod).call(eval(:x.id2name))
```

如果这看起来很复杂，请查看 *symbols_3.rb* 中的更简单示例。它从以下赋值开始：

*symbols_3.rb*

```
def mymethod( somearg )
   print( "I say: " << somearg )
end

this_is_a_method_name = method(:mymethod)
```

这里 `method(:mymethod)` 寻找由作为参数传递的符号指定的方法名（`:mymethod`），如果找到了，它将返回具有相应名称的方法对象。在我的代码中，我有一个名为 `mymethod` 的方法，现在它被分配给了变量 `this_is_a_method_name`。

当你运行这个程序时，你会看到输出第一行打印了变量的值：

```
puts( this_is_a_method_name )  #=> #<Method: Object#mymethod>
```

这表明变量 `this_is_a_method_name` 已经被分配了方法 `mymethod`，该方法绑定到 Object 类（所有作为“独立”函数输入的方法都是绑定到 Object 类的）。为了验证变量确实是 Method 类的实例，下一行代码打印出它的类：

```
puts( "#{this_is_a_method_name.class}" )  #=> Method
```

好吧，如果这真的是一个方法，那么你应该能够调用它，不是吗？要做到这一点，你需要使用 `call` 方法。这正是代码最后一行所做的事情：

```
this_is_a_method_name.call( "hello world" )  #=>I say: hello world
```

# 为什么使用符号？

Ruby 类库中的一些方法指定符号作为参数。自然地，如果你需要调用这些方法，你必须向它们传递符号。然而，除了这些情况之外，在你的编程中并没有绝对必要使用符号。对于许多 Ruby 程序员来说，“传统”的数据类型，如字符串和整数，已经足够完美。然而，许多 Ruby 程序员确实喜欢使用符号作为散列的键。例如，当你查看第十九章（ch19.html "第十九章。Ruby on Rails"）中的 Rails 框架时，你会看到类似以下示例的内容：

```
{ :text => "Hello world" }
```

然而，符号在“动态”编程中确实有一个特殊的位置。例如，Ruby 程序能够在某个类的范围内通过调用 `define_method` 并传递一个表示要定义的方法的符号和一个表示方法代码的代码块来在运行时创建一个新的方法：

*add_method.rb*

```
class Array
    define_method( :aNewMethod, lambda{
        |*args| puts( args.inspect)
    } )
end
```

在执行之前的代码后，Array 类将获得一个名为 `aNewMethod` 的方法。你可以通过调用 `method_defined?` 并传递一个表示方法名称的符号来验证这一点：

```
Array.method_defined?( :aNewMethod )   #=> true
```

当然，你也可以调用该方法本身：

```
[].aNewMethod( 1,2,3     #=> [1,2,3]
```

你可以通过在类内部调用 `remove_method` 并传递一个提供要删除的方法名称的符号来以类似的方式在运行时删除现有方法：

```
class Array
    remove_method( :aNewMethod )
end
```

动态规划在需要修改正在执行中的 Ruby 程序行为的应用中非常有价值。例如，在 Rails 框架中广泛使用动态规划，并在本书的最后一章中进行了深入讨论。

深入挖掘

符号是 Ruby 的基础。在这里，你将了解为什么是这样，以及如何显示所有可用的符号。

什么是符号？

之前我说过，符号是一个其值就是自身的标识符。从 Ruby 程序员的视角来看，这大致描述了符号的行为方式。但这并没有告诉你从 Ruby 解释器的视角来看符号是什么。实际上，符号是符号表的指针。符号表是 Ruby 的已知标识符的内部列表——例如变量和方法名称。

如果你想要深入了解 Ruby，你可以像这样显示 Ruby 所知道的全部符号：

*allsymbols.rb*

```
p( Symbol.all_symbols )
```

这将显示包括 `:to_s` 和 `:reverse` 这样的方法名称、`:$/` 和 `:$DEBUG` 这样的全局变量、`:Array` 和 `:Symbol` 这样的类名称在内的成千上万的符号。你可以使用数组索引来限制显示的符号数量，如下所示：

```
p( Symbol.all_symbols[0,10] )
```

在 Ruby 1.8 中，你不能对符号进行排序，因为符号不被认为是固有的顺序。在 Ruby 1.9 中，排序是可能的，符号字符被像字符串一样排序：

```
# In Ruby 1.9
p [:a,:c,:b].sort       #=> [:a,:b,:c]

# In Ruby 1.8
p [:a,:c,:b].sort       #=> 'sort': undefined method '<=>' for :a:Symbol
```

在避免与 Ruby 版本相关的兼容性问题的情况下，显示符号排序列表的最简单方法是将符号转换为字符串，并对这些字符串进行排序。在下面的代码中，我将 Ruby 所知的所有符号传递到一个块中，将每个符号转换为字符串，并将这些字符串收集到一个新的数组中，该数组被分配给`str_array`变量。现在我可以对这个数组进行排序并显示结果：

```
str_arr = Symbol.all_symbols.collect{ |s| s.to_s }
puts( str_arr.sort )
```
