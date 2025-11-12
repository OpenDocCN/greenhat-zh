# 第二十章 动态规划

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

在过去的 19 章中，我涵盖了 Ruby 语言的大量特性。有一件事我还没有详细讲解，那就是 Ruby 的动态规划能力。

如果你只使用过非动态语言（比如 C 或 Pascal 家族中的语言），那么编程中的动态性可能需要一点时间来适应。在继续之前，我将明确说明我所说的*动态语言*的含义。实际上，这个定义有点模糊，并不是所有声称自己是动态语言的语言都共享所有相同的特性。然而，从一般意义上讲，一个提供某种方式在运行时修改程序的语言可以被认为是动态的。动态语言的另一个特点是它能够改变给定变量的类型——你在这本书中的例子中已经无数次这样做过了。

也可以在 Ruby 这样的*动态类型*语言和*静态类型*语言（变量类型在声明时预先声明并固定）之间做出进一步的区分，例如 C、Java 或 Pascal。在本章中，我将专注于 Ruby 的自我修改能力。

### 注意

在形式化的计算机科学中，术语*动态规划*有时被用来描述解决复杂问题的一种分析方法。但这并不是本章中术语所用的含义。

# 自修改程序

在大多数编译型语言和许多解释型语言中，编写程序和运行程序是两种完全不同的操作：你编写的代码是固定的，在程序运行时已经不可能进行任何进一步的修改。

但 Ruby 并非如此。一个程序——我指的是*Ruby 代码本身*——在程序运行时可以被修改。甚至可以在运行时输入新的 Ruby 代码并执行新代码，而无需重新启动程序。

将数据视为可执行代码的能力被称为*元编程*。你在这本书中一直在进行元编程，尽管是相当简单的一种。每次你在双引号字符串中嵌入一个表达式时，你就是在进行元编程。毕竟，嵌入的表达式并不是真正的程序代码——它是一个字符串——然而 Ruby 显然必须“将其转换为”程序代码才能对其进行评估。

大多数时候，你可能会在双引号字符串中的`#{`和`}`分隔符之间嵌入相当简单的代码片段。你可能会嵌入变量名，比如，或者数学表达式：

*str_eval.rb*

```
aStr = 'hello world'
puts( "#{aStr}" )
puts( "#{2*10}" )
```

但你并不局限于这样的简单表达式。实际上，你可以将几乎所有内容嵌入到双引号字符串中。你甚至可以在字符串中编写整个程序。你甚至不需要使用`print`或`puts`来显示最终结果。只需将双引号字符串放入你的程序中，Ruby 就会对其进行评估：

```
"#{def x(s)
        puts(s.reverse)
    end;
(1..3).each{x(aStr)}}"
```

即使前面的代码片段是一个字符串，Ruby 解释器也会评估其嵌入的代码并显示结果，如下所示：

```
dlrow olleh
dlrow olleh
dlrow olleh
```

虽然这很有趣，但在字符串内编写整个程序可能是一个相当无意义的工作。然而，在其他情况下，这种功能和类似的功能可以更加高效地使用。例如，你可能使用元编程来探索人工智能和“机器学习”。实际上，任何需要根据用户交互修改程序行为的程序都是元编程的理想候选。

### 注意

动态（元编程）特性在 Ruby 中无处不在。例如，考虑属性访问器：将符号（如`:aValue`）传递给`attr_accessor`方法会导致创建两个方法（`aValue`和`aValue=`）。

# eval

`eval`方法提供了一种简单的方法来评估字符串中的 Ruby 表达式。乍一看，`eval`可能看起来与双引号字符串中的`#{ }`定界符做的是同样的工作。这两行代码产生相同的结果：

*eval.rb*

```
puts( eval("1 + 2" ) )    #=> 3
puts( "#{1 + 2}" )        #=> 3
```

然而，有时结果可能并非你所期望的。看看下面的例子：

*eval_string.rb*

```
exp = gets().chomp()    #<= User enters 2*4
puts( eval( exp ))      #=> 8
puts( "#{exp}" )        #=> 2*4
```

假设你输入`2 * 4`，并将其分配给`exp`。当你用`eval`评估`exp`时，结果是 8，但当你用双引号字符串评估`exp`时，结果是`"2*4"`。这是因为`gets()`读取的任何内容都是一个字符串，`"#{exp}"`将其*作为字符串*评估，而不是作为表达式，而`eval( exp )`则将字符串*作为表达式*评估。为了在字符串内强制评估，你可以在字符串中放置`eval`（尽管这确实可能违背了练习的目的）：

```
puts( "#{eval(exp)}" )
```

这里还有一个例子。尝试它，并按照提示操作：

*eval2.rb*

```
print("Enter a string method name (e.g. reverse or upcase):")
                                   # user enters: upcase
methodname = gets().chomp()
exp2 = "'Hello world'."<< methodname
puts( eval( exp2 ) )               #=> HELLO WORLD
puts( "#{exp2}" )                  #=> 'Hello world'.upcase
puts( "#{eval(exp2)}" )            #=> HELLO WORLD
```

`eval`方法可以评估跨越多行的字符串，使得在字符串中执行整个程序成为可能：

*eval3.rb*

```
eval( 'def aMethod( x )
    return( x * 2 )
end

num = 100
puts( "This is the result of the calculation:" )
puts( aMethod( num ))' )
```

仔细看看前面的代码。它只包含一个可执行的表达式，即对`eval()`方法的调用。其他所有看起来像代码的东西，实际上是一个单引号字符串，作为参数传递给`eval()`。`eval()`方法“解包”字符串的内容，将其转换为真正的 Ruby 代码，然后执行。这显示为：

```
This is the result of the calculation:
200
```

在所有这些`eval`的巧妙之处，现在让我们看看编写一个可以自己编写程序的程序有多容易。这里就是：

*eval4.rb*

```
input = ""
until input == "q"
    input = gets().chomp()
    if input != "q" then eval( input ) end
end
```

这可能看起来不多，但这个小程序让你可以从提示符创建和执行 Ruby 代码。试试看。运行程序，一次输入这里显示的两个方法中的一行（但*不要按* *q* *退出*——你很快就会写更多的代码）：

```
def x(aStr); puts(aStr.upcase);end
def y(aStr); puts(aStr.reverse);end
```

注意，你必须将每个整个方法输入到单行中，因为程序会逐行评估输入。我稍后会解释如何绕过这个限制。多亏了 `eval`，每个方法都被转换成了真正的、可工作的 Ruby 代码。你可以通过输入以下内容来证明这一点：

```
x("hello world")
y("hello world")
```

现在，当你按下回车键在上一段代码的每一行之后，表达式将被评估，并调用你刚才编写的两个方法 `x()` 和 `y()`，产生以下输出：

```
HELLO WORLD
dlrow olleh
```

对于仅仅五行代码来说，这已经很不错了！

# 特殊类型的 eval

在 `eval` 主题上存在一些变体，形式为名为 `instance_eval`、`module_eval` 和 `class_eval` 的方法。`instance_eval` 方法可以从特定的对象中调用，并提供对该对象实例变量的访问。它可以带一个代码块或一个字符串来调用：

*instance_eval.rb*

```
class MyClass
 def initialize
   @aVar = "Hello world"
 end
end

ob = MyClass.new
p( ob.instance_eval { @aVar } )         #=> "Hello world"
p( ob.instance_eval( "@aVar" ) )        #=> "Hello world"
```

另一方面，`eval` 方法不能以这种方式从对象中调用，因为它是对象的私有方法（而 `instance_eval` 是公共的）：

```
p( ob.eval( "@aVar" )  )    # This won't work!
```

实际上，你可以通过将 `eval` 的名称（符号 `:eval`）发送到 `public` 方法来显式更改 `eval` 的可见性。在这里，我将 `eval` 添加为 Object 类的公共方法：

```
class Object
    public :eval
end
```

事实上，考虑到当你编写“独立”代码时，你实际上是在 Object 的作用域内工作，简单地输入以下代码（没有 Object 类的“包装”）会产生相同的效果：

```
public :eval
```

现在，你可以将 `eval` 作为 `ob` 变量的方法使用：

```
p( ob.eval( "@aVar" ) )        #=> "Hello world"
```

### 注意

严格来说，`eval` 是 `Kernel` 模块的一个方法，它被混合到 Object 类中。实际上，是 `Kernel` 模块提供了大多数作为 Object 方法可用的函数。

在运行时修改类定义有时被称为 *猴子补丁*。这在某些高度专业化的编程类型中可能起到一定的作用，但作为一个一般原则，随意篡改标准 Ruby 类绝对是不推荐的。更改方法的可见性和向基类添加新行为是创建难以理解的代码依赖性的绝佳方式（例如，你的程序之所以能工作，是因为你偶然知道如何更改基类，但你的同事的程序不能工作，因为他们不知道类是如何被更改的）。

`module_eval` 和 `class_eval` 方法作用于模块和类，而不是对象。例如，下面的代码向 `X` 模块（在这里 `xyz` 在一个代码块中定义，并通过 `define_method`（它是 Module 类的一个方法）添加为接收者的实例方法）添加了 `xyz` 方法，并将 `abc` 方法添加到 Y 类：

*module_eval.rb*

```
module X
end

class Y
    @@x = 10
    include X
end

X::module_eval{ define_method(:xyz){ puts("hello" ) } }
Y::class_eval{ define_method(:abc){ puts("hello, hello" ) } }
```

### 注意

当访问类和模块方法时，你可以使用作用域解析运算符 `::` 或单个点。当访问常量时，作用域解析运算符是强制的，当访问方法时是可选的。

因此，现在一个 Y 类的实例对象将能够访问 Y 类的`abc`方法和被混合到 Y 类中的`X`模块的`xyz`方法：

```
ob = Y.new
ob.xyz        #=> hello
ob.abc        #=> hello, hello
```

尽管它们的名称不同，`module_eval`和`class_eval`在功能上是相同的，并且每个都可以与模块或类一起使用：

```
X::class_eval{ define_method(:xyz2){ puts("hello again" ) } }
Y::module_eval{ define_method(:abc2){ puts("hello, hello again") }}
```

你也可以以相同的方式向 Ruby 的标准类中添加方法：

```
String::class_eval{ define_method(:bye){ puts("goodbye" ) } }
"Hello".bye        #=> goodbye
```

# 添加变量和方法

你还可以使用`module_eval`和`class_eval`方法来检索类变量的值（但请注意，你这样做得越多，你的代码就越依赖于类的实现细节，从而损害封装）：

```
Y.class_eval( "@@x" )
```

实际上，`class_eval`可以评估任意复杂性的表达式。例如，你可以用它来通过评估一个字符串向一个类添加新方法：

```
ob = X.new
X.class_eval( 'def hi;puts("hello");end' )
ob.hi        #=> hello
```

返回到之前关于从*外部*一个类中添加和检索类变量的例子（使用`class_eval`），结果发现也有方法可以从*内部*一个类中做这件事。这些方法被称为`class_variable_get`（这个方法接受一个表示变量名的符号参数，并返回变量的值）和`class_variable_set`（这个方法接受一个表示变量名的符号参数和一个第二个参数，即要分配给变量的值）。

这里是这些方法使用的一个例子：

*classvar_getset.rb*

```
class X
    def self.addvar( aSymbol, aValue )
        class_variable_set( aSymbol, aValue )
    end

    def self.getvar( aSymbol )
        return class_variable_get( aSymbol )
    end
end

X.addvar( :@@newvar, 2000 )
puts( X.getvar( :@@newvar ) )    #=> 2000
```

要获取一个表示类变量名的字符串数组，请使用`class_variables`方法：

```
p( X.class_variables )    #=> ["@@abc", "@@newvar"]
```

你还可以在创建类和对象之后使用`instance_variable_set`向类和对象添加实例变量：

*dynamic.rb*

```
ob = X.new
ob.instance_variable_set("@aname", "Bert")
```

通过结合添加方法的能力，大胆（或可能鲁莽？）的程序员可以完全从“外部”改变一个类的内部结构。在这里，我在类 X 中实现了一个名为`addMethod`的方法，它使用`send`方法通过`define_method`和由`&block`定义的方法体来创建新的方法`m`：

```
def addMethod( m, &block )
    self.class.send( :define_method, m , &block )
end
```

### 注意

`send`方法调用由第一个参数（一个符号）标识的方法，并将任何指定的参数传递给它。

现在，一个 X 对象可以调用`addMethod`来向 X 类中插入一个新的方法：

```
ob.addMethod( :xyz ) { puts("My name is #{@aname}") }
```

虽然这个方法是从类的特定实例（这里`ob`）调用的，但它影响的是类本身，因此新定义的方法也将对从 X 类创建的任何后续实例（这里`ob2`）可用：

```
ob2 = X.new
ob2.instance_variable_set("@aname", "Mary")
ob2.xyz
```

如果你不在乎你的对象中数据的封装（我对*封装*的定义假设隐藏内部数据，尽管有些人有更宽松的定义），你也可以使用`instance_variable_get`方法来检索实例变量的值：

```
ob2.instance_variable_get( :@aname )
```

你也可以同样地*设置*和*获取*常量：

```
X::const_set( :NUM, 500 )
puts( X::const_get( :NUM ) )
```

因为 `const_get` 返回常量的值，所以你可以使用此方法获取类名（它本身也是一个常量）的值，然后附加 `new` 方法从这个类创建一个新的对象。这甚至可以为你提供一个在运行时通过提示用户输入类名和方法名来创建对象的方法。通过运行此程序来尝试：

*dynamic2.rb*

```
class X
    def y
        puts( "ymethod" )
   end
end

print( "Enter a class name: ")                  #<= Enter: X
cname = gets().chomp
ob = Object.const_get(cname).new
p( ob )                                         #=> #<X:0x2bafdc0>
print( "Enter a method to be called: " )        #<= Enter: y
mname = gets().chomp
ob.method(mname).call                           #=> ymethod
```

# 在运行时创建类

到目前为止，你已经修改了类并从现有类创建了新对象。但你是如何创建一个完全新的类呢？嗯，就像你可以使用 `const_get` 访问现有类一样，你也可以使用 `const_set` 创建新类。以下是一个示例，说明如何在创建类之前提示用户输入新类的名称，向其中添加一个方法（`myname`），创建该类的实例（`x`），并调用其 `myname` 方法：

*create_class.rb*

```
puts("What shall we call this class? ")
className = gets.strip().capitalize()
Object.const_set(className,Class.new)
puts("I'll give it a method called 'myname'" )
className = Object.const_get(className)
className::module_eval{ define_method(:myname){
        puts("The name of my class is '#{self.class}'" ) }
    }

x = className.new
x.myname
```

如果你运行此程序并在提示输入新类名时输入 `Xxx`，代码将使用 `const_set` 创建一个名为 `Xxx` 的新常量作为类；然后对类调用 `module_eval`，并使用 `define_method` 创建一个名称与符号 `:myname` 匹配的方法，其内容由花括号分隔的代码块给出；这里恰好是一个显示类名的 `puts` 语句。

运行此代码，当提示输入时请输入 `Xxx`。从 `Xxx` 类创建了一个对象 `x`；调用了其 `myname()` 方法；果然，它显示了类名：

```
The name of my class is 'Xxx'
```

# 绑定

`eval` 方法可能接受一个可选的“绑定”参数，如果提供，则会在特定的作用域或“上下文”内执行评估。在 Ruby 中，一个绑定是 Binding 类的实例。你可以使用 `binding` 方法返回一个绑定。Ruby 类库中 `eval` 的文档提供了以下示例：

*binding.rb*

```
def getBinding(str)
    return binding()
end
str = "hello"
puts( eval( "str + ' Fred'" ) )                    #=> "hello Fred"
puts( eval( "str + ' Fred'", getBinding("bye") ) ) #=> "bye Fred"
```

看起来可能很简单，但为了理解这里发生了什么，可能需要一些思考。本质上，第一次调用 `puts` 在当前作用域中评估 `str`，其中它具有“hello”值。第二次调用 `puts` 在 `getBinding()` 方法的范围内评估 `str`，其中它具有“bye”值。在这个例子中，`str` 偶然被作为参数传递，但这不是必需的。在这个重写的版本中，我已经将 `str` 作为 `getBinding()` 内部的局部变量。效果是相同的：

*binding2.rb*

```
def getBinding()
    str = "bye"
    return binding()
end
str = "hello"
puts( eval( "str + ' Fred'" )   )                  #=> "hello Fred"
puts( eval( "str + ' Fred'", getBinding() ) )      #=> "bye Fred"
puts( eval( "str + ' Fred'" )   )                  #=> "hello Fred"
```

注意，`binding` 是 Kernel 的一个私有方法。`getBinding` 方法能够在当前上下文中调用 `binding` 并返回 `str` 的当前值。在第一次调用 `eval` 时，上下文是 *主* 对象，局部变量 `str` 的值被使用；在第二次调用中，上下文移动到 `getBinding` 方法内部，局部值 `str` 现在是方法中的 `str` 参数或变量的值。上下文也可以由一个类定义。在 *binding3.rb* 中，你可以看到实例变量 `@mystr` 的值根据类而变化。那么，当你使用不同的绑定 `eval` 这些变量时会发生什么？

*binding3.rb*

```
class MyClass
   @@x = " x"
   def initialize(s)
      @mystr = s
   end
   def getBinding
      return binding()
   end
end

class MyOtherClass
   @@x = " y"
   def initialize(s)
      @mystr = s
   end
   def getBinding
      return binding()
   end
end

@mystr = self.inspect
@@x = " some other value"

ob1 = MyClass.new("ob1 string")
ob2 = MyClass.new("ob2 string")
ob3 = MyOtherClass.new("ob3 string")

puts(eval("@mystr << @@x", ob1.getBinding))
puts(eval("@mystr << @@x", ob2.getBinding))
puts(eval("@mystr << @@x", ob3.getBinding))
puts(eval("@mystr << @@x", binding))
```

在 Ruby 1.8 中，你会看到以下输出，显示实例变量 `@mystr` 和类变量 `@@x` 的绑定都被应用：

```
ob1 string x
ob2 string x
ob3 string y
main some other value
```

但在 Ruby 1.9 中，只有实例变量的绑定被应用；当前（*主*）上下文中的类变量始终被使用：

```
ob1 string some other value
ob2 string some other value
ob3 string some other value
main some other value
```

这是否意味着给定绑定中的类变量被忽略？让我们做一个实验。只需在主上下文中注释掉对 `@@x` 的赋值：

```
# @@x = " some other value"
```

现在再次运行程序。这次，Ruby 1.9 显示如下：

```
ob1 string x
ob2 string x
ob3 string y
...uninitialized class variable @@x in Object (NameError)
```

显然，Ruby 1.9 *确实*在绑定中评估类变量。然而，如果存在类变量，它会优先考虑 *当前* 绑定中的类变量。如果你正在将 Ruby 1.8 程序迁移到 Ruby 1.9 或更新的版本，你需要注意这个差异。

# send

你可以使用 `send` 方法调用与指定符号具有相同名称的方法：

*send1.rb*

```
name = "Fred"
puts( name.send( :reverse ) )    #=> derF
puts( name.send( :upcase ) )     #=> FRED
```

虽然文档中说明 `send` 方法需要符号参数，但你也可以使用字符串参数。或者，为了保持一致性，你可以使用 `to_sym` 将字符串转换为符号，然后使用与该符号相同的名称调用方法：

```
name = MyString.new( gets() )
methodname = gets().chomp.to_sym #<= to_sym is not strictly necessary
name.send(methodname)
```

这里是使用 `send` 在运行时执行命名方法的示例：

*send2.rb*

```
class MyString < String
    def initialize( aStr )
        super aStr
    end

    def show
        puts self
    end

    def rev
        puts self.reverse
    end
end

print("Enter your name: ")          #<= Enter: Fred
name = MyString.new( gets() )
print("Enter a method name: " )     #<= Enter: rev
methodname = gets().chomp.to_sym
puts( name.send(methodname) )       #=> derF
```

# 移除方法

回想一下，你之前（*dynamic.rb*）使用 `send` 调用 `define_method` 并传递给它要创建的方法的名称，`m`，以及包含新方法代码的块，`&block`：

*dynamic.rb*

```
def addMethod( m, &block )
    self.class.send( :define_method, m , &block )
end
```

除了创建新方法外，有时你可能想移除现有方法。你可以在给定类的范围内使用 `remove_method` 来做这件事。这将从一个特定的类中移除由符号指定的方法：

*rem_methods1.rb*

```
puts( "hello".reverse )  #=> olleh
class String
    remove_method( :reverse )
end
puts( "hello".reverse )  #=> undefined method error!
```

如果为该类的祖先定义了具有相同名称的方法，则不会移除祖先类的方法：

*rem_methods2.rb*

```
class Y
    def somemethod
        puts("Y's somemethod")
    end
end

class Z < Y
    def somemethod
        puts("Z's somemethod")
    end
end

zob = Z.new
zob.somemethod                     #=> Z's somemethod
class Z
     remove_method( :somemethod )  # Remove somemethod from Z class
end

zob.somemethod                     #=> Y's somemethod
```

在这个例子中，`somemethod` 从 Z 类中被移除，因此当在 Z 对象上随后调用 `zob.somemethod` 时，Ruby 会执行 Z 的 *祖先* 类中第一个具有该名称的方法。在这里，Y 是 Z 的祖先，所以它的 `somemethod` 方法被使用。

与之相反，`undef_method` 阻止指定的类响应方法调用，即使在其祖先中定义了具有相同名称的方法。以下示例使用了之前示例中使用的相同的 Y 和 Z 类。唯一的区别是这次使用 `undef_method` 而不是使用 `remove_method` 仅从当前类中移除 `somemethod`：

*undef_methods.rb*

```
zob = Z.new
zob.somemethod                       #=> Z's somemethod

class Z
   undef_method( :somemethod )       #=> undefine somemethod
end

zob.somemethod                       #=> undefined method error
```

# 处理缺失的方法

当 Ruby 尝试执行一个未定义的方法（或者，用面向对象的话说，当一个对象收到它无法处理的消息时），错误会导致程序退出。你可能希望你的程序能够从这样的错误中恢复。你可以通过编写一个名为 `method_missing` 的方法来实现，该方法将未找到的方法名称分配给一个参数。这将在一个不存在的方法被调用时执行：

*nomethod1.rb*

```
def method_missing( methodname )
   puts( "Sorry, #{methodname} does not exist" )
end
xxx        #=> Sorry, xxx does not exist
```

`method_missing` 方法还可以在缺失的方法名称之后接受一个传入参数的列表 (`*args`)：

*nomethod2.rb*

```
def method_missing( methodname, *args )
      puts( "Class #{self.class} does not understand:
                   #{methodname}( #{args.inspect} )" )
end
```

假设之前的 `method_missing` 方法被写入一个名为 X 的类中，你现在可以尝试在 X 对象上调用任何方法，无论该方法是否存在以及是否传递了任何参数。例如，如果你尝试调用一个不存在的名为 `aaa` 的方法，首先不带任何参数，然后带三个整数参数，`method_missing` 方法将响应无效的方法调用并显示适当的错误信息：

```
ob = X.new
ob.aaa            #=> Class X does not understand: aaa( [] )
ob.aaa( 1,2,3 )   #=> Class X does not understand: aaa( [1, 2, 3] )
```

`method_missing` 方法甚至可以动态地创建一个未定义的方法，以便对不存在的方法的调用自动使其存在：

```
def method_missing( methodname, *args )
       self.class.send( :define_method, methodname,
            lambda{ |*args| puts( args.inspect) } )
end
```

记住，`lambda` 方法将一个块（这里是大括号之间的代码）转换成一个 Proc 对象。这已在 第十章 中解释。然后代码能够将此对象作为参数传递给 `send`，定义一个与传递给 `method_missing` 的 `methodname` 参数具有相同名称的新方法。结果是，当在 Z 对象上调用未知方法时，会创建一个具有该名称的方法。运行包含此代码的 *nomethod2.rb* 程序：

```
ob3 = Z.new
ob3.ddd( 1,2,3)
ob3.ddd( 4,5,6 )
```

这会产生以下输出：

```
Class Z does not understand: ddd( [1, 2, 3] )
Now creating method ddd( )
[4, 5, 6]
```

# 在运行时编写程序

最后，让我们回到你之前查看的程序：*eval4.rb*。你可能还记得，这个程序提示用户输入字符串以在运行时定义代码，评估这些字符串，并从它们中创建新的可运行方法。

那个程序的一个缺点是它坚持要求每个方法都在单行上输入。实际上，编写一个允许用户输入跨越多行的方法的程序相当简单。例如，以下是一个评估直到输入空白行为止的所有输入代码的程序：

*writeprog.rb*

```
program = ""
input = ""
line = ""
until line.strip() == "q"
    print( "?- " )
    line = gets()
    case( line.strip() )
    when ''
        puts( "Evaluating..." )
        eval( input )
        program += input
        input = ""
    when '1'
        puts( "Program Listing..." )
        puts( program )
   else
        input += line
    end
end
```

你可以通过输入完整的方法后跟空白行来尝试这个方法（当然，只输入代码，不要输入注释）：

```
def a(s)             # <= press Enter after each line
return s.reverse     # <= press enter (and so on...)
end
                     # <- Enter a blank line here to eval these two methods
def b(s)
return a(s).upcase
end
                     # <- Enter a blank line here to eval these two methods
puts( a("hello" ) )

                     # <- Enter a blank line to eval
                     #=> olleh
puts( b("goodbye" ) )
                     # <- Enter a blank line to eval
                     #=> EYBDOOG
```

输入每一行后，都会出现一个提示（`?-`），除非程序正在评估代码的过程中，此时会显示“Evaluating”，或者当它显示评估结果时，例如 `olleh`。

如果您按照之前指示的文本输入，您应该看到以下内容：

```
Write a program interactively.
Enter a blank line to evaluate.
Enter 'q' to quit.
?- def a(s)
?- return s.reverse
?- end
?-
Evaluating...
?- def b(s)
?- return a(s).upcase
?- end
?-
Evaluating...
?- puts(a("hello"))
?-
Evaluating...
olleh
?- b("goodbye")
?-
Evaluating...
EYBDOOG
```

这个程序仍然非常简单。它甚至没有基本的错误恢复功能，更不用说文件保存和加载等花哨的功能。即便如此，这个小例子展示了在 Ruby 中编写自修改程序是多么容易。

# 进一步探索

使用本章概述的技术，您可以创建从可以教授语法规则的天然语言解析器到可以学习新谜题的冒险游戏等任何东西。

在这本书中，我涵盖了大量的内容——从“hello world”到动态规划。您已经探索了 Ruby 语言的大部分重要和强大的功能。其余的取决于您。

这就是冒险真正开始的地方。

深入挖掘

有时候，您可能想确保您的 Ruby 对象不能以本章中描述的方式被修改。在这里，您将学习如何做到这一点。

冻结对象

在您拥有所有这些修改对象的方法之后，您可能会担心对象可能会被无意中修改。实际上，您可以使用 `freeze` 方法“冻结”对象来具体固定对象的状态，这是您在第十二章中首次遇到的。一旦被冻结，对象包含的数据就不能被修改，如果尝试修改，将会引发 TypeError 异常。然而，在冻结对象时要小心，因为一旦被冻结，它就不能被“解冻”。

*freeze.rb*

```
s = "Hello"
s << " world"
s.freeze
s << " !!!"   # Error: "can't modify frozen string"
```

您可以使用 `frozen?` 方法具体检查一个对象是否被冻结：

```
a = [1,2,3]
a.freeze
if !(a.frozen?) then
    a << [4,5,6]
end
```

请注意，尽管冻结对象的数据不能被修改，但定义它的类可以被修改。假设您有一个包含 `addMethod` 方法的类 X，该方法可以使用由符号 `m` 给定的名称创建新方法：

*cant_freeze.rb*

```
def addMethod( m, &block )
    self.class.send( :define_method, m , &block )
end
```

现在，如果您有一个由 M 类创建的对象，`ob`，那么调用 `addMethod` 向类 M 添加新方法是完全合法的：

```
ob.freeze
ob.addMethod( :abc ) { puts("This is the abc method") }
```

如果您想防止冻结对象修改其类，当然可以使用 `frozen?` 方法测试其状态：

```
if not( ob.frozen? ) then
  ob.addMethod(:def){puts("'def' is not a good name for a method")}
end
```

您还可以冻结类本身（记住，类也是一个对象）：

*freeze_class.rb*

```
X.freeze
if not( X.frozen? ) then
  ob.addMethod(:def){puts("'def' is not a good name for a method")}
end
```
