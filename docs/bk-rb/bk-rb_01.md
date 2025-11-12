# 第一章。字符串、数字、类和对象

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

关于 Ruby 的第一件事是它很容易使用。为了证明这一点，让我们看看传统“Hello world”程序的代码：

*1helloworld.rb*

```
puts 'hello world'
```

这就是它的全部内容。程序包含一个方法，`puts`，和一个字符串，“hello world。”它没有任何头文件或类定义，也没有任何导入部分或“main”函数。这真的是简单到极致。加载代码，`*1helloworld.rb*`，然后尝试运行。

# 获取和输出输入

将字符串“输出”到输出（这里是一个命令窗口）后，明显的下一步是“获取”一个字符串。正如你可能猜到的，Ruby 用于此的方法是 `gets`。`*2helloname.rb*` 程序提示用户输入他的或她的名字——假设它是 Fred——然后显示问候：“Hello Fred。”以下是代码：

*2helloname.rb*

```
print( 'Enter your name: ' )
name = gets()
puts( "Hello #{name}" )
```

虽然这仍然非常简单，但需要解释一些重要的细节。首先，请注意，我使用了 `print` 而不是 `puts` 来显示提示。这是因为 `puts` 在打印的字符串末尾添加一个换行符，而 `print` 则不会；在这种情况下，我希望光标保持在提示的同一行。

在下一行，我使用 `gets()` 读取用户按下回车键时输入的字符串。这个字符串被分配给变量 `name`。我没有预先声明这个变量，也没有指定其类型。在 Ruby 中，你可以在需要时创建变量，解释器“推断”它们的类型。在这个例子中，我将一个字符串分配给 `name`，这样 Ruby 就知道 `name` 变量的类型必须是字符串。

### 注意

Ruby 区分大小写。一个名为 `myvar` 的变量与一个名为 `myVar` 的变量不同。在示例项目中，变量如 `name` 必须以小写字母开头。如果它以大写字母开头，Ruby 会将其视为常量。我将在第六章中更多地讨论常量。

顺便说一下，`gets()` 后面的括号是可选的，`print` 和 `puts` 后面括号包围的字符串也是可选的；如果你去掉它们，代码仍然会正常运行。然而，括号可以帮助解决歧义，并且在某些情况下，如果省略了它们，解释器会警告你。

# 字符串和嵌入式评估

示例代码的最后一行相当有趣：

```
puts( "Hello #{name}" )
```

在这里，`name` 变量被嵌入到字符串中。你这样做是通过将变量放在由星号（或“数字”或“井号”字符）前缀的两个大括号之间，例如 `#{}`。这种*嵌入式*评估仅适用于双引号分隔的字符串。如果你尝试用单引号分隔的字符串这样做，变量将不会被评估，字符串 `'Hello #{name}'` 将会按原样显示。

你还可以嵌入非打印字符，如换行符（`"\n"`）和制表符（`"\t"`），甚至可以嵌入程序代码和数学表达式。例如，假设你有一个名为 `showname` 的方法，它返回字符串 “Fred.”。以下字符串在评估过程中会调用 `showname` 方法并显示 “Hello Fred”：

```
puts "Hello #{showname}"
```

看看你是否能猜出以下代码会显示什么：

*3string_eval.rb*

```
puts( "\n\t#{(1 + 2) * 3}\nGoodbye" )
```

现在运行 *3string_eval.rb* 程序，看看你是否正确。

# 数字

数字的使用与字符串一样简单。例如，假设你想根据某项商品不含税价值或小计来计算销售价格或总金额。为此，你需要将小计乘以适用的税率，并将结果加到小计的价值上。假设小计为 $100，税率为 17.5%，以下 Ruby 程序执行计算并显示结果：

*4calctax.rb*

```
subtotal = 100.00
taxrate = 0.175
tax = subtotal * taxrate
puts "Tax on $#{subtotal} is $#{tax}, so grand total is $#{subtotal+tax}"
```

显然，如果这个程序能够对各种小计进行计算，而不是反复计算相同的值，将会更有用！以下是一个简单的计算器，它会提示用户输入小计：

```
taxrate = 0.175
print "Enter price (ex tax): "
s = gets
subtotal = s.to_f
tax = subtotal * taxrate
puts "Tax on $#{subtotal} is $#{tax}, so grand total is $#{subtotal+tax}"
```

这里 `s.to_f` 是 String 类的一个方法。它尝试将字符串转换为浮点数。例如，字符串 `"145.45"` 将被转换为浮点数 145.45。如果字符串无法转换，则返回 0.0。例如，`"Hello world".to_f` 将返回 0.0。

# 注释

许多与本书一起提供的源代码示例都带有注释，这些注释会被 Ruby 解释器忽略。你可以在散列符号（`#`）之后放置注释。该符号之后的整行文本都被视为注释：

```
# this is a comment
puts( "hello" ) # this is also a comment
```

如果你想注释掉多行文本，可以在开始处放置 `=begin`，在结尾处放置 `=end`（`=begin` 和 `=end` 必须与左边缘对齐）：

```
=begin
 This is a
 multiline
 comment
=end
```

# 测试条件：if..then

之前展示的简单税费计算器代码的问题在于它接受负小计并对其计算负税费——这种情况政府可能不会持乐观态度！因此，我需要检查负数，并在找到时将它们设置为零。这是我的新代码版本：

*5taxcalculator.rb*

```
taxrate = 0.175
print "Enter price (ex tax): "
s = gets
subtotal = s.to_f

if (subtotal < 0.0)  then
  subtotal = 0.0
end

tax = subtotal * taxrate
puts "Tax on $#{subtotal} is $#{tax}, so grand total is $#{subtotal+tax}"
```

Ruby 的 `if` 测试与其他编程语言中的 `if` 测试类似。注意，然而，括号是可选的，关键字 `then` 也是可选的。然而，如果你要写以下内容，且测试条件后没有换行，则 `then` 是必需的：

```
if (subtotal < 0.0) then subtotal = 0.0 end
```

将所有内容放在一行中，像这样并不会增加代码的清晰度，这就是为什么我倾向于避免这样做。我对 Pascal 的长期熟悉使我本能地想要在`if`条件之后添加一个`then`，但因为这个实际上并不是必需的，你可以把这看作是我的一种任性的古怪行为。终止`if`块的`end`关键字是*不是*可选的。如果你忘记添加它，你的代码将无法运行。

# 局部和全局变量

在上一个示例中，我给变量如`subtotal`、`tax`和`taxrate`赋值。以小写字母开头的变量，如这些，被称为*局部变量*。这意味着它们只存在于程序的一个特定部分——换句话说，它们被限制在一个明确的范围内。以下是一个例子：

*variables.rb*

```
localvar = "hello"
$globalvar = "goodbye"

def amethod
    localvar = 10
    puts( localvar )
    puts( $globalvar )
end

def anotherMethod
    localvar = 500
    $globalvar = "bonjour"
    puts( localvar )
    puts( $globalvar )
end
```

在之前的代码中，有两个函数（或*方法*），`amethod`和`anotherMethod`，每个都使用关键字`def`声明，并包含到`end`关键字之前的代码。有三个名为`localvar`的局部变量。一个在程序的“主要作用域”内被赋予了`"hello"`的值；另外两个在两个不同的方法的作用域内被赋予了整数。由于每个局部变量有不同的作用域，这些赋值对在不同作用域中具有相同名称的其他局部变量没有影响。你可以通过依次调用这些方法来验证这一点。以下示例显示了注释后的输出，后跟`=>`字符。在这本书中，输出或返回值通常会以这种方式表示：

```
amethod           #=> localvar = 10
anotherMethod     #=> localvar = 500
amethod           #=> localvar = 10
puts( localvar )  #=> localvar = "hello"
```

另一方面，一个*全局变量*——以美元符号字符（`$`）开头——具有全局作用域。当在方法内部对一个全局变量进行赋值时，这也会影响程序其他地方该变量的值：

```
amethod            #=>  $globalvar = "goodbye"
anotherMethod      #=>  $globalvar = "bonjour"
amethod            #=>  $globalvar = "bonjour"
puts( $globalvar ) #=>  $globalvar = "bonjour"
```

# 类和对象

我们不一一介绍 Ruby 的所有语法——它的类型、循环、模块等等——而是快速地看看如何创建类和对象。（但别担心，我们很快就会回到那些其他主题。）

基本术语：类、对象和方法

类是对象的蓝图。它定义了对象包含的数据以及它的行为方式。可以从单个类创建许多不同的对象。因此，你可能有一个 Cat *类*，但三个 cat *对象*：tiddles、cuddles 和 flossy。*方法*就像是在类内部定义的函数或子程序。

说 Ruby 是面向对象的可能看起来没什么大不了的。现在不是所有语言都是这样吗？好吧，在一定程度上是这样的。大多数现代的“面向对象”语言（Java、C++、C#、Object Pascal 等等）都有不同程度的面向对象编程（OOP）特性。然而，Ruby 则是极度面向对象的。实际上，除非你曾经使用过 Smalltalk 或 Eiffel（这些语言对对象的执着甚至超过 Ruby），否则 Ruby 可能是你用过的最面向对象的编程语言。每一块数据——从简单的数字或字符串到更复杂的东西，如文件或模块——都被视为对象。而且，你几乎用对象做的所有事情都是通过方法完成的。甚至像加号（`+`）和减号（`−`）这样的运算符也是方法。考虑以下例子：

```
x = 1 + 2
```

在这里，`+` 是 Fixnum（整数）对象 1 的一个方法。值 2 被发送到这个方法；结果是 3，然后返回，并分配给对象 x。顺便提一下，赋值运算符（`=`）是“你用对象做的所有事情都是通过方法完成”的规则中罕见的例外之一。赋值运算符是一个特殊的内置“东西”（我急忙补充，这不是正式术语），它不是任何东西的方法。

现在，你将看到如何创建自己的对象。在大多数其他面向对象编程语言中，Ruby 对象是由类定义的。类就像一个蓝图，从该蓝图构建单个对象。例如，这个类定义了一只狗：

*6dogs.rb*

```
class Dog
    def set_name( aName )
       @myname = aName
    end
end
```

注意，类定义以关键字 `class`（全部小写）和类名本身开始，类名必须以大写字母开头。该类包含一个名为 `set_name` 的方法。该方法接受一个传入参数，`aName`。方法体将 `aName` 的值赋给一个名为 `@myname` 的变量。

## 实例变量

以井号（`@`）开头的变量是 *实例变量*，这意味着它们属于类的单个对象（或 *实例*）。不需要预先声明实例变量。我可以通过调用 `new` 方法来创建 Dog 类的实例（即“狗对象”）。在这里，我创建了两个狗对象（注意，尽管类名以大写字母开头，但对象名以小写字母开头）：

```
mydog = Dog.new
yourdog = Dog.new
```

目前，这两只狗还没有名字。所以，接下来我要做的是调用 `set_name` 方法来给它们命名：

```
mydog.set_name( 'Fido' )
yourdog.set_name( 'Bonzo' )
```

## 从对象中检索数据

给每只狗取了名字后，我需要有一种方法来在以后找出它们的名字。我该如何做呢？我不能在对象内部乱翻来获取 `@name` 变量，因为每个对象的内部细节只有对象本身才知道。这是“纯”面向对象的一个基本原则：每个对象内部的数据是私有的。每个对象都有精确的进入和退出方式（例如，`set_name` 方法）和精确的退出方式。只有对象本身可以随意处理其内部状态；外部世界不能。这被称为 *数据隐藏*，它是 *封装* 原则的一部分。

封装

*封装* 描述了对象包含其自己的数据和操作这些数据所需的方法的事实。一些面向对象的语言鼓励或强制执行 *数据隐藏*，以便对象内部封装的数据不能被该对象之外的其他代码访问。在 Ruby 中，数据隐藏的执行并不像最初看起来那么严格。你可以使用一些非常脏的技巧来在对象内部捣乱，但为了简单起见，我现在会默默地忽略这个语言的这些特性。

由于你需要每只狗都知道自己的名字，让我们为 Dog 类提供一个 `get_name` 方法：

```
def get_name
    return @myname
end
```

这里的 `return` 关键字是可选的。当它被省略时，Ruby 方法将返回最后一个评估的表达式。然而，为了清晰起见——以及避免比这个更复杂的方法带来的意外结果——我将养成显式返回任何我打算使用的值的习惯。

最后，让我们通过让狗说话来给它一些行为。下面是完成后的类定义：

```
class Dog
    def set_name( aName )
       @myname = aName
    end

    def get_name
       return @myname
    end

    def talk
       return 'woof!'
    end
end
```

现在，你可以创建一只狗，给它取名字，显示它的名字，并让它说话：

```
mydog = Dog.new
mydog.set_name( 'Fido' )
puts(mydog.get_name)
puts(mydog.talk)
```

我在 *6dogs.rb* 程序中编写了这个代码的扩展版本。这个版本还包含一个与 Dog 类相似的 Cat 类，除了它的 `talk` 方法，自然地返回一个 *喵喵* 而不是 *汪汪*。

当一个变量未分配时会发生什么？

哎呀！看来这个程序包含了一个错误。名为 `someotherdog` 的对象从未为其 `@name` 变量分配值，因为它的 `set_name()` 方法从未被调用。这意味着以下尝试打印其名称的代码无法成功：

```
puts(someotherdog.get_name)
```

幸运的是，当你尝试显示这只狗的名字时，Ruby 并不会崩溃。相反，它只是打印出“nil。”你很快就会看到一种简单的方法来确保这种错误不再发生。

## 消息、方法和多态性

顺便说一句，这个猫和狗的例子是基于一个经典的 Smalltalk 演示程序，它说明了相同的“消息”（例如 `talk`）可以发送到不同的对象（例如猫和狗），并且每个不同的对象都会用其自己的特殊方法（这里是指 `talk` 方法）对相同的消息做出不同的响应。具有包含具有相同名称的方法的不同类的能力，可以用花哨的面向对象术语 *多态性* 来描述。

当你运行一个如 *6dogs.rb* 的程序时，代码是按顺序执行的。类本身的代码不会执行，直到程序底部的代码创建了这些类的实例（即对象）。你会发现我经常将类定义与在程序运行时执行的“自由”代码块混合。这可能不是你编写主要应用程序的方式，但仅用于尝试事情，它非常方便。

自由代码块？

如果你认为 Ruby 真的是一个面向对象的语言，你可能觉得可以输入“自由浮动”的方法很奇怪。实际上，当你运行一个程序时，Ruby 会创建一个主对象，而你主代码单元（即你加载并运行的主 Ruby 代码文件）中的任何代码实际上都是在那个对象内部运行的。你可以通过创建一个新的源文件并添加以下代码来轻松验证这一点：

```
puts self
puts self.class
```

当你运行这个程序时，你会看到以下输出：

```
main
Object
```

这个程序的一个明显缺陷是，两个类，猫和狗，非常重复。有一个类，动物（Animal），它有 `get_name` 和 `set_name` 方法，而两个子类，猫和狗，只包含特定物种动物的行为（汪汪叫或喵喵叫）会更合理。我们将在下一章中了解到如何做到这一点。

## 构造函数：new 和 initialize

让我们来看一个用户自定义类的另一个例子。加载 *7treasure.rb*。这是一个正在制作中的冒险游戏。它包含两个类，物品（Thing）和宝藏（Treasure）。物品类与之前程序中的猫（Cat）和狗（Dog）类类似——只不过它不会汪汪叫或喵喵叫，就是这样。

*7treasure.rb*

```
class Thing
    def set_name( aName )
        @name = aName
    end

    def get_name
        return @name
    end
end

class Treasure
      def initialize( aName, aDescription )
        @name         = aName
        @description  = aDescription
      end

      def to_s # override default to_s method
       "The #{@name} Treasure is #{@description}\n"
      end
end

thing1 = Thing.new
thing1.set_name( "A lovely Thing" )
puts thing1.get_name

t1 = Treasure.new("Sword", "an Elvish weapon forged of gold")
t2 = Treasure.new("Ring", "a magic ring of great power")
puts t1.to_s
puts t2.to_s
# The inspect method lets you look inside an object
puts "Inspecting 1st treasure: #{t1.inspect}"
```

宝藏类没有 `get_name` 和 `set_name` 方法。相反，它包含一个名为 `initialize` 的方法，它接受两个参数。这两个值随后被分配给 `@name` 和 `@description` 变量。当一个类包含一个名为 `initialize` 的方法时，当使用 `new` 方法创建对象时，它将自动被调用。这使得它是一个设置对象实例变量值的方便位置。

这有两个明显的优点，比使用 `set_name` 这样的方法设置每个实例变量要好。首先，一个复杂的类可能包含许多实例变量，你可以使用单个 `initialize` 方法设置所有这些变量的值，而不是使用许多单独的“设置”方法；其次，如果变量在对象创建时自动初始化，你就不会得到一个“空”变量（就像在之前程序中尝试显示某个其他狗的名字时返回的“nil”值）。

最后，我创建了一个名为 `to_s` 的方法，它返回 Treasure 对象的字符串表示形式。方法名 `to_s` 不是任意的——在标准的 Ruby 对象层次结构中使用了相同的方法名。实际上，`to_s` 方法是为 Object 类本身定义的，它是 Ruby 中所有其他类的最终祖先（除了 BasicObject 类，你将在下一章中更详细地了解）。通过重新定义 `to_s` 方法，我添加了更适合 Treasure 类的新行为。换句话说，我 *重写* 了它的 `to_s` 方法。

由于 `new` 方法创建了一个对象，它可以被认为是对象的 *构造函数*。构造函数是一个为对象分配内存并执行（如果存在）`initialize` 方法以将指定的值分配给新对象的内部变量的方法。你通常不应该实现自己的 `new` 方法版本。相反，当你想要执行任何“设置”操作时，应在 `initialize` 方法中执行。

垃圾回收

在许多语言（如 C++ 和 Delphi for Win32）中，当对象不再需要时，程序员有责任销毁任何创建的对象。换句话说，对象既有 *析构函数* 也有 *构造函数*。在 Ruby 中这不是必要的，因为内置的 *垃圾回收器* 会自动销毁对象并回收它们不再被程序引用时使用的内存。

## 检查对象

注意，在 *7treasure.rb* 程序中，我使用了 `inspect` 方法“查看”了 Treasure 对象 t1 的内部结构：

```
puts "Inspecting 1st treasure: #{t1.inspect}"
```

`inspect` 方法为所有 Ruby 对象定义。它返回一个包含对象人类可读表示的字符串。在本例中，它显示如下：

```
#<Treasure:0x28962f8 @description="an Elvish weapon forged of gold", @name="Sword">
```

这以类名 Treasure 开头。接下来是一个数字，这个数字可能与之前显示的数字不同——这是 Ruby 为这个特定对象提供的内部识别码。接下来显示了对象的变量名称和值。

Ruby 还提供了 `p` 方法作为检查对象和打印其详细信息的快捷方式，如下所示：

```
p( anobject )
```

其中 `anobject` 可以是任何类型的 Ruby 对象。例如，假设你创建了以下三个对象：一个字符串、一个数字和一个 Treasure 对象：

*p.rb*

```
class Treasure
    def initialize( aName, aDescription )
      @name         = aName
      @description  = aDescription
    end

    def to_s # override default to_s method
         "The #{@name} Treasure is #{@description}\n"
    end
end

a = "hello"
b = 123
c = Treasure.new( "ring", "a glittery gold thing" )
```

现在，你可以使用 `p` 来显示这些对象：

```
p( a )
p( b )
p( c )
```

这是 Ruby 显示的内容：

```
"hello"
123
#<Treasure:0x3489c4 @name="ring", @description="a glittery gold thing">
```

要了解如何使用 `to_s` 与各种对象，并测试在没有重写 `to_s` 方法的情况下 Treasure 对象将如何转换为字符串，请尝试 *8to_s.rb* 程序。

*8to_s.rb*

```
puts(Class.to_s)        #=> Class
puts(Object.to_s)       #=> Object
puts(String.to_s)       #=> String
puts(100.to_s)          #=> 100
puts(Treasure.to_s)     #=> Treasure
```

正如你所见，当调用 `to_s` 方法时，Class、Object、String 和 Treasure 等类会简单地返回它们的名称。一个对象，例如 Treasure 对象 t，会返回它的标识符——这个标识符与 `inspect` 方法返回的标识符相同：

```
t = Treasure.new( "Sword", "A lovely Elvish weapon" )
puts(t.to_s)
    #=> #<Treasure:0x3308100>
puts(t.inspect)
    #=> #<Treasure:0x3308100 @name="Sword", @description="A lovely Elvish weapon">
```

尽管 *7treasure.rb* 程序可能为包含多种不同对象类型的游戏奠定了基础，但其代码仍然重复。毕竟，为什么会有一个包含名称的 Thing 类和一个也包含名称的 Treasure 类？将 Treasure 视为“一种” Thing 更有意义。在一个完整的游戏中，其他对象如 Rooms 和 Weapons 可能是 Thing 的其他类型。显然，是时候开始构建合适的类层次结构了，这正是你将在下一章中要做的。
