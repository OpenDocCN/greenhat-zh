# 第十二章 模块和混入

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

在 Ruby 中，每个类只有一个直接的“父类”，尽管每个父类可能有多个“子类”。通过将类层次结构限制为单行继承，Ruby 避免了那些允许多行继承的编程语言（如 C++）可能出现的某些问题。当类有多个父类和子类，并且它们的父类和子类也有其他父类和子类时，你可能会得到一个难以穿透的网络（一个结网？），而不是你可能期望的整洁、有序的层次结构。

尽管如此，有时对于不是紧密相关的类实现一些共享功能是有用的。例如，剑可能是一种武器，但也是一种宝藏；个人电脑可能是一种计算机，但也是一种投资；等等。但是，由于定义武器和宝藏或计算机和投资的类来自不同的祖先类，它们的类层次结构没有明显的共享数据和方法的途径。Ruby 解决这个问题的方案是通过模块提供的。

# 模块就像一个类……

模块的定义看起来非常类似于类的定义。实际上，模块和类是紧密相关的；模块类是类类的直接祖先。就像类一样，模块可以包含常量、方法和类。以下是一个简单的模块：

*simple_module.rb*

```
module MyModule
    REWARD = 100

    def prize
        return "You've won #{REWARD} credits"
    end

end
```

如您所见，这包含一个常量`REWARD`和一个*实例方法*，`prize`。

# 模块方法

除了实例方法之外，模块还可能有模块方法。就像类方法以类的名称为前缀一样，模块方法以模块的名称为前缀：

```
def MyModule.lose
    return "Sorry, you didn't win"
end
```

你可以像调用类的类方法一样调用模块的模块方法，使用点符号，如下所示：

```
MyModule.lose   #=> "Sorry, you didn't win"
```

但如何调用实例方法？以下两种尝试都没有成功：

```
puts( prize )           # Error: undefined local variable or method
puts( MyModule.prize )  # Error: undefined method 'prize'
```

尽管它们有相似之处，但类有两个主要特征是模块不具备的：*实例*和*继承*。类可以有实例（从类创建的对象），超类（父类），和子类（子类）；模块则没有这些。从模块的实例（一个“模块对象”）中调用实例方法是不可能的，简单的原因是无法创建模块的实例。这就是当你尝试调用前面代码中的`prize`方法时出现错误的原因。

### 注意

模块类确实有一个超类，即 Object。然而，你创建的任何命名模块都没有超类。有关模块和类之间关系的更详细说明，请参阅深入挖掘中的深入挖掘。

这引出了下一个问题：如果你不能从模块创建对象，模块有什么用？这可以用两个词来回答：*命名空间*和*混入*。Ruby 的混入提供了一种处理多重继承问题的方法。你很快就会了解混入是如何工作的。首先，让我们看看命名空间。

# 模块作为命名空间

你可以将模块视为围绕一组方法、常量和类的一种命名“包装”。模块内部的代码片段共享相同的“命名空间”，因此它们彼此可见，但对模块外部的代码不可见。

Ruby 类库定义了多个模块，例如`Math`和`Kernel`。`Math`模块包含数学方法，如`sqrt`来返回平方根，以及常量如`PI`。`Kernel`模块包含了许多你从一开始就一直在使用的方 法，如`print`、`puts`和`gets`。

假设你已经编写了这个模块：

*modules1.rb*

```
module MyModule
    GOODMOOD = "happy"
    BADMOOD = "grumpy"

    def greet
        return "I'm #{GOODMOOD}. How are you?"
    end

    def MyModule.greet
        return "I'm #{BADMOOD}. How are you?"
    end
end
```

你已经看到了如何使用模块方法，例如`MyModule.greet`，你可以像访问类常量一样访问模块常量，使用作用域解析运算符`::`，如下所示：

```
puts(MyModule::GOODMOOD)    #=> happy
```

但你如何访问实例方法`greet`？这就是混入介入的地方。

# 包含的模块，或称为“混入”

一个对象可以通过使用`include`方法包含一个模块来访问该模块的实例方法。如果你要在你的程序中包含`MyModule`，那么该模块中的所有内容都会突然在当前作用域中存在。因此，`MyModule`的`greet`方法现在可以访问：

*modules2.rb*

```
include MyModule
```

注意，只有实例方法被包含。在先前的例子中，`greet`（实例）方法已被包含，但`MyModule.greet`（模块）方法没有被包含。由于它被包含，`greet`实例方法可以像当前作用域中的普通实例方法一样使用，而模块方法，也命名为`greet`，则是使用点符号来访问的：

```
puts( greet )            #=> I'm happy. How are you?
puts( MyModule.greet )   #=> I'm grumpy. How are you?
```

包含模块的过程也称为*混入*，这也解释了为什么包含的模块通常被称为混入。当你将模块混入类定义中时，从该类创建的任何对象都将能够使用混入模块的实例方法，就像它们在类本身中定义一样。这里`MyClass`类混入了`MyModule`模块：

*modules3.rb*

```
class MyClass
    include MyModule

    def sayHi
        puts( greet )
    end

end
```

不仅这个类的方 法可以访问`MyModule`中的`greet`方法，从该类创建的任何对象也可以：

```
ob = MyClass.new
ob.sayHi          #=> I'm happy. How are you?
puts(ob.greet)    #=> I'm happy. How are you?
```

你可以将模块视为离散的代码单元，这可能有助于简化可重用代码库的创建。另一方面，你可能更感兴趣的是将模块用作多重继承的替代方案。

回到我在本章开头提到的例子，假设你有一个 Sword 类，它不仅是一件武器，也是一件宝物。也许 Sword 是 Weapon 类的后代（因此继承了 Weapon 的`deadliness`属性），但它还需要具有宝物的属性（如`value`和`owner`）。此外，由于这是一把精灵剑，它还需要具有魔法物品的属性。如果你在`Treasure`和`MagicThing`模块内部而不是类内部定义这些属性，Sword 类就能够包含这些模块，以便“混入”它们的方法或属性：

*modules4.rb*

```
module MagicThing
    attr_accessor :power
end

module Treasure
    attr_accessor :value
    attr_accessor :owner
end

class Weapon
    attr_accessor :deadliness
end

class Sword < Weapon        # descend from Weapon
    include Treasure        # mix in Treasure
    include MagicThing      # mix in MagicThing
    attr_accessor :name
end
```

现在 Sword 对象可以访问 Sword 类的方法和属性，以及其祖先类 Weapon 的方法和属性，还有混入的模块`Treasure`和`MagicThing`的方法和属性：

```
s = Sword.new
s.name = "Excalibur"
s.deadliness = "fatal"
s.value = 1000
s.owner = "Gribbit The Dragon"
s.power = "Glows when Orcs appear"
puts(s.name)            #=> Excalibur
puts(s.deadliness)      #=> fatal
puts(s.value)           #=> 1000
puts(s.owner)           #=> Gribbit The Dragon
puts(s.power)           #=> Glows when Orcs appear
```

顺便提一下，任何属于模块的局部变量都无法从模块外部访问。即使模块内部的方法试图访问局部变量，并且该方法被模块外部的代码调用，例如通过混入模块，也是如此：

*mod_vars.rb*

```
x = 1             # local to this program

module Foo
    x = 50        # local to module Foo

                  # this can be mixed in but the variable x won't be visible
    def no_bar
        return x
    end

    def bar
         @x = 1000
         return  @x
    end
    puts( "In Foo: x = #{x}" )   # this can access the module-local x
end

include Foo                      # mix in the Foo module
```

当你运行这个程序时，`puts`方法在模块初始化时执行，并显示模块局部变量`x`的值：

```
In Foo: x = 50
```

如果你想在程序的主作用域中显示变量`x`，则使用的是程序主作用域中局部变量`x`的值，*而不是*模块内部局部变量`x`的值：

```
puts(x)           #=> 1
```

但尝试执行`no_bar`方法将会失败：

```
puts( no_bar )    # Error: undefined local variable or method 'x'
```

在这里，`no_bar`方法无法访问名为`x`的任何局部变量，尽管`x`在模块的作用域（`x = 50`）和当前或“主”作用域（`x = 1`）中都被声明了。但对于实例变量则没有这样的问题。`bar`方法能够返回实例变量`@x`的值：

```
puts(bar)         #=> 1000
```

模块可以有自己的实例变量，这些变量仅属于模块“对象”。这些实例变量将适用于模块方法：

*inst_class_vars.rb*

```
module X
    @instvar = "X's @instvar"

    def self.aaa
        puts(@instvar)
    end
end

X.aaa #=> X's @instvar
```

但在实例对象中引用的实例变量“属于”将模块混入的作用域：

```
module X
    @instvar = "X's @instvar"
    @anotherinstvar = "X's 2nd @instvar"

        def amethod
             @instvar = 10       # creates @instvar in current scope
             puts(@instvar)
        end
end

include X
p( @instvar )                    #=> nil
amethod                          #=> 10
puts( @instvar )                 #=> 10
@instvar = "hello world"
puts( @instvar )                 #=> "hello world"
```

类变量也被混入，并且像实例变量一样，它们的值可以在当前作用域内重新赋值：

```
module X
    @@classvar = "X's @@classvar"
end

include X

puts( @@classvar )         #=> X's @classvar
@@classvar = "bye bye"
puts( @@classvar )         #=> "bye bye"
```

你可以使用`instance_variables`方法获取实例变量名称的数组：

```
p( X.instance_variables )      #=> [:@instvar, @anotherinstvar]
p( self.instance_variables )   #=> [:@instvar]
```

在这里，`X.instance_variables`返回属于 X 类的实例变量列表，而`self.instance_variables`返回当前、主对象的实例变量。`@instvar`变量在每个情况下都是不同的。

### 注意

在 Ruby 1.9 中，`instance_variables`方法返回一个符号数组。在 Ruby 1.8 中，它返回一个字符串数组。

# 命名冲突

模块方法（那些特别以模块名开头的方法）可以帮助保护你的代码免受意外的名称冲突。然而，模块中的实例方法并没有提供此类保护。假设你有两个模块——一个叫 `Happy`，另一个叫 `Sad`。它们各自包含一个名为 `mood` 的模块方法和一个名为 `expression` 的实例方法。

*happy_sad.rb*

```
module Happy
    def Happy.mood        # module method
        return "happy"
    end

    def expression        # instance method
        return "smiling"
    end
end
module Sad
    def Sad.mood          # module method
        return "sad"
    end

    def expression        # instance method
        return "frowning"
    end
end
```

现在类 `Person` 包含了这两个模块：

```
class Person
    include Happy
    include Sad
    attr_accessor :mood

    def initialize
        @mood = Happy.mood
    end
end
```

`Person` 类的 `initialize` 方法需要使用包含模块中的一个 `mood` 方法来设置其 `@mood` 变量的值。他们都有 `mood` 方法的事实并没有问题；作为一个模块方法，`mood` 必须在模块名之前，所以 `Happy.mood` 不会与 `Sad.mood` 混淆。

但是 `Happy` 和 `Sad` 模块也各自包含一个名为 `expression` 的方法。这是一个 *实例* 方法，当这两个模块都被包含在 `Person` 类中时，可以直接调用 `expression` 方法，无需任何限定：

```
p1 = Person.new
puts(p1.expression)
```

这里对象 `p1` 使用的是哪种 `expression` 方法？结果发现它使用的是最后定义的方法。在示例案例中，这恰好是 `Sad` 模块中定义的方法，因为 `Sad` 是在 `Happy` 之后被包含的。所以，`p1.expression` 返回的是“皱眉。”如果你改变包含顺序，使得 `Happy` 在 `Sad` 之后被包含，`p1` 对象将使用 `Happy` 模块中定义的 `expression` 方法，并显示“微笑。”

在沉迷于创建大型、复杂的模块并将它们定期混入你的类之前，请记住这个潜在的问题：*包含的具有相同名称的实例方法将“覆盖”彼此*。这个问题在我的小程序中可能很容易发现。但在一个大型应用程序中可能就不那么明显了！

# 别名方法

当你使用来自多个模块的具有相似名称的方法时，避免歧义的一种方法是对这些方法进行 *别名*。别名是现有方法的副本，具有新名称。你使用 `alias` 关键字后跟新名称，然后是旧名称：

```
alias  happyexpression expression
```

你也可以使用 `alias` 来复制被覆盖的方法，以便你可以具体引用其覆盖定义之前的版本：

*alias_methods.rb*

```
module Happy
    def Happy.mood
        return "happy"
    end

    def expression
        return "smiling"
    end
    alias happyexpression expression
end

module Sad
    def Sad.mood
        return "sad"
    end

    def expression
        return "frowning"
    end
    alias sadexpression expression
end

class Person
    include Happy
    include Sad
    attr_accessor :mood
    def initialize
        @mood = Happy.mood
    end
end

p2 = Person.new
puts(p2.mood)                 #=> happy
puts(p2.expression)           #=> frowning
puts(p2.happyexpression)      #=> smiling
puts(p2.sadexpression)        #=> frowning
```

# 小心混入！

尽管每个类只能继承一个超类，但它可以混入多个模块。实际上，将一组模块混入另一组模块，然后将这些其他模块混入类，然后将这些类放入更多的模块中，然后继续这样做是完全允许的。

以下是一些代码示例，它子类化了某些类，混入了某些模块，甚至在混入的模块中子类化了类。我故意简化了以下代码，以便你能看到正在发生的事情。要查看完整的工作示例的恐怖之处，请参阅代码存档中提供的示例程序，*multimods.rb*：

*multimods.rb*

```
# This is an example of how NOT to use modules!
module MagicThing                           # module
    class MagicClass                        # class inside module
    end
end

module Treasure                             # module
end

module MetalThing
    include MagicThing                      # mixin
    class Attributes < MagicClass           # subclasses class from mixin
    end
end

include MetalThing                          # mixin
class Weapon < MagicClass                   # subclass class from mixin
    class WeaponAttributes < Attributes     # subclass
    end
end

class Sword < Weapon                        # subclass
    include Treasure                        # mixin
    include MagicThing                      # mixin
end
```

让我强调，之前展示并包含在存档中的代码并不是作为要仿效的模型。远非如此！它仅仅是为了展示一个过度使用模块的程序可能会变得多么难以理解，几乎无法调试。

简而言之，尽管在使用得当的情况下，模块可以帮助避免与 C++ 多重继承相关的一些复杂性，但它们仍然容易受到误用的威胁。如果程序员真的想创建具有难以理解的依赖关系的复杂类层次结构，那么他们当然可以做到。*multimods.rb* 中的代码展示了如何仅用几行代码就写出难以理解的程序。想象一下，在成千上万行代码和数十个代码文件中你能做什么！在混合模块之前要仔细思考。

# 从文件中包含模块

到目前为止，我混合了在单个源文件中定义的模块。通常，在单独的文件中定义模块并在需要时混合它们更有用。为了使用其他文件中的代码，你必须首先使用 `require` 方法加载该文件，如下所示：

*require_module.rb*

```
require( "./testmod.rb" )
```

可选地，你可以省略文件扩展名：

```
require( "./testmod" )  # this works too
```

如果没有指定路径，所需的文件必须在当前目录、搜索路径或预定义数组变量 `$:` 中列出的文件夹中。你可以使用常规的数组追加方法 `<<` 将目录添加到这个数组变量中，如下所示：

```
$: << "C:/mydir"
```

### 注意

全局变量 `$:`（一个美元符号和一个冒号）包含一个字符串数组，表示 Ruby 在查找已加载或所需的文件时搜索的目录。

在 Ruby 1.8 和 Ruby 1.9 中 `require` 的行为存在一个已记录的差异。在 Ruby 1.8 中，文件名不会被转换成绝对路径，因此 `require "a"; require "./a"` 会加载 `a.rb` 两次。而在 Ruby 1.9 中，文件名会被转换成绝对路径，所以 `require "a"; require "./a"` 不会加载 `a.rb` 两次。

此外，我发现至少在某些版本的 Ruby 1.9 中，如果你使用未指定扩展名的文件名，如 `require("testmod")`，`require` 可能无法从当前目录加载文件。在这种情况下，会抛出一个 LoadError 异常。这通常发生在全局变量 `$:` 中存储的可搜索目录数组不包含当前目录时。你可以通过运行以下代码来验证这一点：

*search_dirs.rb*

```
puts( $: )
```

搜索路径将按行显示。应该有一行显示单个点（`.`），代表当前目录。如果这个点缺失，那么当前目录中的文件不在搜索路径上，无法使用未指定扩展名的文件名来加载。

为了确保文件被加载，我在文件名前加了一个点来指定当前目录，现在这成功了：`require( "./testmod" )`。或者，你也可以使用`require_relative`方法，尽管这是 Ruby 1.9 的新特性，不能在早期版本中使用：

```
require_relative( "testmod.rb" )    # Ruby 1.9 only
```

或者，如果`$:`不包含当前目录，你可以将其添加进去。一旦这样做，`require`将能够与当前目录中文件的未限定名称一起工作：

```
$: << "."              # add current directory to array of search paths
require( "testmod.rb" )
```

`require`方法如果成功加载指定的文件，则返回`true`值；否则，返回`false`。如果文件不存在，它返回一个 LoadError。如果有疑问，你可以简单地显示结果。

```
puts(require( "testmod.rb" )) #=> true, false or LoadError
```

当文件运行时通常会被执行的任何代码，在文件被`require`时也会被执行。所以，如果文件*testmod.rb*包含以下代码：

*testmod.rb*

```
def sing
    puts( "Tra-la-la-la-la....")
end

puts( "module loaded")
sing
```

当运行*require_module.rb*程序并要求*testmod.rb*时，这将显示：

```
module loaded
Tra-la-la-la-la....
```

当在所需的文件中声明模块时，它可以被混合：

```
require_module2.rb
require( "testmod.rb")
include MyModule       #mix in MyModule declared in testmod.rb
```

Ruby 还允许你使用`load`方法加载一个文件。在大多数方面，`require`和`load`可以被视为可互换的。但是，它们之间有一些细微的差别。特别是，`load`可以接受一个可选的第二个参数，如果这个参数是`true`，则将代码作为未命名的或匿名模块加载和执行：

```
load( "testmod.rb", true)
```

当第二个参数是`true`时，加载的文件不会将新的命名空间引入主程序，你将无法访问加载文件中的模块。在这种情况下，模块的方法、常量和实例方法将**不会**对你的代码可用：

*load_module.rb*

```
load( "testmod.rb", true)

puts( MyModule.greet )   #=>Error:uninitialized constant Object::MyModule
puts(MyModule::GOODMOOD) #=>Error:uninitialized constant Object::MyModule
include MyModule         #=>Error:uninitialized constant Object::MyModule
puts( greet )            #=>Error:undefined local variable or method 'greet'
```

然而，当`load`的第二个参数是`false`或者没有第二个参数时，你将**能够**访问加载文件中的模块：

*load_module_false.rb*

```
load( "testmod.rb", false)

puts( MyModule.greet )   #=> I'm grumpy. How are you?
puts(MyModule::GOODMOOD) #=> happy
include MyModule         #=> [success]
puts( greet )            #=> I'm happy. How are you?
```

注意，你必须使用`load`输入完整的文件名（*testmod*去掉*.rb*扩展名是不够的）。`load`和`require`之间的另一个区别是，`require`只加载文件一次（即使你的代码多次要求该文件），而`load`每次调用`load`都会重新加载指定的文件。假设你有一个包含以下代码的文件*test.rb*：

*test.rb*

```
MyConst = 1
if @a == nil then
    @a = 1
else
    @a += MyConst
end

puts @a
```

你现在**require**这个文件三次：

*require_again.rb*

```
require "./test"
require "./test"
require "./test"
```

这将是输出：

```
1
```

但如果你**加载**该文件三次 . . .

*load_again.rb*

```
load "test.rb"
load "test.rb"
load "test.rb"
```

然后，这将输出：

```
1
./test.rb:1: warning: already initialized constant MyConst
2
./test.rb:1: warning: already initialized constant MyConst
3
```

深入挖掘

模块与类究竟是如何相关的？在这里，我们回答这个问题，检查一些重要的 Ruby 模块，并了解如何使用模块来扩展对象。

模块和类

在本章中，我讨论了模块的**行为**。现在，让我们弄清楚模块究竟**是什么**。结果证明，就像 Ruby 中的大多数其他事物一样，模块是一个对象。每个命名的模块实际上都是 Module 类的实例：

*module_inst.rb*

```
module MyMod
end

puts( MyMod.class )            #=> Module
```

你不能创建**命名模块**的子类，所以这是不允许的：

```
module MyMod
end

module MyOtherMod < MyMod      # You can't do this!
end
```

然而，与其他类一样，可以创建 `Module` 类的子类：

```
class X < Module               # But you can do this
end
```

事实上，`Class` 类本身是 `Module` 类的子类。它继承了 `Module` 的行为并添加了一些重要的新行为，特别是创建对象的能力。您可以通过运行 *modules_classes.rb* 程序来验证 `Module` 是 `Class` 的超类，该程序显示了此层次结构：

*modules_classes.rb*

```
Class
Module            #=> is the superclass of Class
Object            #=> is the superclass of Module
BasicObject       #=> (in Ruby 1.9) is the superclass of Module
```

预定义模块

以下模块是 Ruby 解释器内建的：`Comparable`、`Enumerable`、`FileTest`、`GC`、`Kernel`、`Math`、`ObjectSpace`、`Precision`、`Process` 和 `Signal`。

`Comparable` 是一个允许包含类实现比较运算符的混入模块。包含的类必须定义 `<=>` 运算符，该运算符将接收者与另一个对象进行比较，根据接收者是否小于、等于或大于另一个对象返回 -1、0 或 +1。

+   `Comparable` 使用 `<=>` 实现传统的比较运算符（`<`、`<=`、`==`、`>=` 和 `>`）和 `between?` 方法。

+   `Enumerable` 是一个用于枚举的混入模块。包含的类必须提供 `each` 方法。

+   `FileTest` 是一个包含文件测试函数的模块；其方法也可以从 `File` 类访问。

+   `GC` 模块提供了 Ruby 标记和清除垃圾收集机制的接口。一些底层方法也可以通过 `ObjectSpace` 模块访问。

+   `Kernel` 是由 `Object` 类包含的模块；它定义了 Ruby 的“内置”方法。

+   `Math` 是一个包含基本三角函数和超越函数模块函数的模块。它具有相同定义和名称的“实例方法”和模块方法。

+   `ObjectSpace` 是一个包含与垃圾收集设施交互的例程的模块，并允许您使用迭代器遍历所有活动对象。

+   `Precision` 是一个用于具有精度的具体数值类的混入模块。在这里，“精度”意味着实数的近似精度，因此此模块不应包含在不是实数子集的任何内容中（因此不应包含在复数或矩阵等类中）。

+   `Process` 是用于操作进程的模块。它所有的方法都是模块方法。

+   `Signal` 是处理发送给运行进程的信号的模块。可用的信号名称列表及其解释取决于系统。

以下是对三个最常用的 Ruby 模块的一个简要概述。

**Kernel**

预定义模块中最重要的一个是 `Kernel`，它提供了许多“标准”Ruby 方法，如 `gets`、`puts`、`print` 和 `require`。与 Ruby 类库的许多部分一样，`Kernel` 是用 C 语言编写的。尽管 `Kernel` 实际上“内置于”Ruby 解释器中，但从概念上讲，它可以被视为一个混合模块，就像一个正常的 Ruby 混合模块一样，它将方法直接提供给任何需要它的类。由于它被混合到所有其他 Ruby 类继承的 `Object` 类中，因此 `Kernel` 的方法对所有类都是通用的。

**Math**

`Math` 模块的方法既提供为“模块”方法又提供为“实例”方法，因此可以通过将 `Math` 混合到类中或通过使用模块名称、点和方法名称来“从外部”访问模块方法；你可以使用双冒号来访问常量：

*math.rb*

```
puts( Math.sqrt(144) )  #=> 12.0
puts( Math::PI )        #=> 3.141592653589793
```

**Comparable**

`Comparable` 模块提供了定义自己的比较“运算符”的便捷能力，如 `<`, `<=`, `==`, `>=`, 和 `>`（严格来说，这些是方法，但它们可以像其他语言中的比较运算符一样使用）。这是通过将模块混合到你的类中并定义 `<=>` 方法来实现的。然后你可以指定比较当前对象中的某个值与其他值的标准。例如，你可以比较两个整数、两个字符串的长度，或者一些更古怪的价值，比如字符串在数组中的位置。我在示例程序 *compare.rb* 中选择了这种古怪的比较类型。它使用神话生物数组中字符串的索引来比较一个生物的名字与另一个生物的名字。一个低索引，如索引 0 的 `hobbit`，被认为“小于”一个高索引，如索引 6 的 `dragon`：

*compare.rb*

```
class Being
       include Comparable

       BEINGS = ['hobbit','dwarf','elf','orc','giant','oliphant','dragon']

       attr_accessor :name

       def <=> (anOtherName)
               BEINGS.index(@name)<=>BEINGS.index(anOtherName.name)
       end

       def initialize( aName )
               @name = aName
       end

end

elf =  Being.new('elf')
orc = Being.new('orc')
giant = Being.new('giant')

puts( elf < orc )     #=> true
puts( elf > giant )   #=> false
```

范围解析

与类一样，你可以使用双冒号范围解析运算符来访问模块内部声明的常量（包括类和其他模块）。例如，假设你有一个嵌套的模块和类，如下所示：

```
module OuterMod
    moduleInnerMod
        class Class1
        end
    end
end
```

你可以使用 `::` 运算符来访问 `Class1`，如下所示：

```
OuterMod::InnerMod::Class1
```

### 注意

有关类中常量范围解析的介绍，请参阅第二章。

每个模块和类都有自己的范围，这意味着单个常量名可能在不同的范围内使用。因此，你可以使用 `::` 运算符来指定精确范围内的常量：

```
Scope1::Scope2::Scope3        #...etc
```

如果你在这个常量名的开头使用这个运算符，这将导致跳出当前范围并访问“顶级”范围：

```
::ACONST                      # refers to ACONST at top-level scope
```

以下程序提供了一些范围运算符的示例：

*scope_resolution.rb*

```
ACONST = "hello"                         # This is a top-level constant

module OuterMod
   module InnerMod
      ACONST=10                          # OuterMod::InnerMod::ACONST
      class Class1
         class Class2
            module XYZ
               class ABC
                  ACONST=100             # Deeply nested ACONST
                  def xyz
                     puts( ::ACONST )    # <= This refers to top-level ACONST
                  end
               end
            end
         end
      end
   end
end

puts(OuterMod::InnerMod::ACONST)                             #=> 10
puts(OuterMod::InnerMod::Class1::Class2::XYZ::ABC::ACONST)   #=> 100
ob = OuterMod::InnerMod::Class1::Class2::XYZ::ABC.new
ob.xyz                                                       #=> hello
```

模块函数

如果你希望一个函数既作为实例方法又作为模块方法可用，你可以使用与实例方法名称匹配的 `module_function` 方法，如下所示：

*module_func.rb*

```
module MyModule
    def sayHi
        return "hi!"
    end

    def sayGoodbye
        return "Goodbye"
  end

    module_function :sayHi
end
```

`sayHi` 方法现在可以被混合到类中，并用作实例方法：

```
class MyClass
    include MyModule
        def speak
            puts(sayHi)
            puts(sayGoodbye)
        end
end
```

它可以用作模块方法，使用点符号表示：

```
ob = MyClass.new
ob.speak                   #=> hi!\nGoodbye
puts(MyModule.sayHi)       #=> hi!
```

由于这里的 `sayGoodbye` 方法不是一个模块函数，因此不能以这种方式使用：

```
puts(MyModule.sayGoodbye)  #=> Error: undefined method
```

Ruby 在其一些标准模块中使用了 `module_function`，例如 `Math`（在 Ruby 库文件 *complex.rb* 中），来创建模块和实例方法的“匹配对”。

扩展对象

你可以使用 `extend` 方法将模块的方法添加到特定的对象（而不是整个类）中，如下所示：

*extend.rb*

```
module A
    def method_a
        puts( 'hello from a' )
    end
end

class MyClass
    def mymethod
        puts( 'hello from mymethod of class MyClass' )
    end
end

ob = MyClass.new
ob.mymethod       #=> hello from mymethod of class MyClass
ob.extend(A)
```

现在，对象 `ob` 被扩展了模块 `A`，它可以访问该模块的实例方法 `method_a`：

```
ob.method_a      #=> hello from a
```

实际上，你可以一次扩展一个对象为多个模块。在这里，模块 `B` 和 `C` 扩展了对象 `ob`：

```
module B
    def method_b
        puts( 'hello from b' )
    end
end

module C
    def mymethod
        puts( 'hello from mymethod of module C' )
    end
end

ob.extend(B, C)
ob.method_b       #=> hello from b
ob.mymethod       #=> hello from mymethod of module C
```

当一个对象被扩展为一个包含与对象类中方法同名的方法的模块时，模块中的方法将替换类中的方法。因此，当 `ob` 被扩展为 `C` 并调用 `ob.mymethod` 时，将显示字符串“hello from mymethod of module `C`”，而不是在 `ob` 扩展模块 `C` 之前显示的“hello from mymethod of class MyClass”。

冻结对象

你可以通过使用 `freeze` 方法“冻结”对象来显式防止对象被扩展：

```
ob.freeze
```

任何尝试进一步扩展此对象的行为都将导致运行时错误：

```
module D
    def method_d
        puts( 'hello from d' )
    end
end
ob.extend( D ) #=> Error: can't modify frozen object (RuntimeError)
```

为了避免这种错误，你可以使用 `frozen?` 方法来测试一个对象是否已被冻结：

```
if !(ob.frozen?)
    ob.extend( D )
    ob.method_d
else
    puts( "Can't extend a frozen object" )
end
```
