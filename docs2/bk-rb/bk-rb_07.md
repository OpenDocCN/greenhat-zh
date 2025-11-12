# 第七章。方法

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

你在这本书中使用了无数的方法。总的来说，它们并不特别复杂，所以你可能想知道为什么关于方法的这一章这么长。正如你将发现的，方法远比表面看起来要复杂得多。

# 类方法

你到目前为止使用的方法都是 *实例方法*。实例方法属于类的特定实例——换句话说，属于单个对象。你也可以编写 *类方法*。（其他一些语言将这种方法称为静态方法。）类方法属于类本身。要定义类方法，必须在方法名前加上类名和句点。

*class_methods1.rb*

```
class MyClass
   def MyClass.classMethod
      puts( "This is a class method" )
   end

   def instanceMethod
      puts( "This is an instance method" )
   en
end
```

当调用类方法时，你应该使用类名：

```
MyClass.classMethod
```

特定的对象不能调用类方法。同样，类也不能调用实例方法：

```
MyClass.instanceMethod        #=> Error! This is an 'undefined method'
ob.classMethod                #=> Error! This is an 'undefined method'
```

# 类方法的作用是什么？

但是，你可能会合理地问，为什么你想要创建一个类方法而不是更常见的实例方法？主要有两个原因：首先，类方法可以用作“即用函数”而无需麻烦地创建一个对象来使用，其次，它可以在需要在使用对象之前就运行方法的情况下使用。

举几个将方法作为“即用函数”使用的例子，可以考虑 Ruby 的 File 类。它的大多数方法都是类方法。这是因为大多数时候，你会使用它们对现有文件进行操作或返回有关文件的信息。你不需要创建一个新的 File 对象来做这件事；相反，你将文件名作为参数传递给 File 类的方法。你将在第十三章（第十三章。文件和 IO）中更详细地了解 File 类。以下是它的一些类方法使用示例：

*file_methods.rb*

```
fn = 'file_methods.rb'
if File.exist?(fn) then
   puts(File.expand_path(fn))
   puts(File.basename(fn))
   puts(File.dirname(fn))
   puts(File.extname(fn))
   puts(File.mtime(fn))
   puts("#{File.size(fn)} bytes")
else
   puts( "Can't find file!")
end
```

这会输出类似以下的内容：

```
C:/bookofruby2/ch7/file_methods.rb
file_methods.rb
.
.rb
2010-10-05 16:14:53 +0100
300 bytes
```

在需要在使用对象之前就使用一个方法时，类方法至关重要。这个例子中最重要的是 `new` 方法。

你每次创建对象时都会调用 `new` 方法。在对象被创建之前，显然你不能调用其实例方法——因为只能从已经存在的对象中调用实例方法。当你使用 `new` 时，你是在调用类本身的方法，并告诉类创建一个新的实例。

# 类变量

类方法可能会让你想起之前使用的类变量（即，以 `@@` 开头的变量）。你可能记得，你曾在简单的冒险游戏中使用类变量（见 属性读取器和写入器 中的 *2adventure.rb*）来统计游戏中对象的总数；每次创建一个新的 Thing 对象时，`@@num_things` 类变量就增加 1：

```
class Thing
   @@num_things = 0

   def initialize( aName, aDescription )
      @@num_things +=1
   end

end
```

与实例变量（即属于从类创建的特定对象的变量）不同，类变量在首次声明时必须赋予一个值：

```
@@classvar = 1000     # class variables must be initialized
```

在类体内部初始化实例或类变量只会影响类本身存储的值。类变量对类本身以及从该类创建的对象都是可用的。然而，每个实例变量都是唯一的；每个对象都有其自己的任何实例变量的副本——*类本身也可能有自己的实例变量*。

类变量、实例变量和方法：总结

实例变量以 `@` 开头：

```
@myinstvar        # instance variable
```

类变量以 `@@` 开头：

```
@@myclassvar      # class variable
```

实例方法通过 `def` *`MethodName`* 定义：

```
def anInstanceMethod
    # some code
end
```

类方法通过 `def` *`ClassName`*.*`MethodName`* 定义：

```
def MyClass.aClassMethod
    # some code
end
```

要了解一个类可能有哪些实例变量，请参考 *class_methods2.rb* 程序。该程序定义了一个包含一个类方法和一个实例方法的类：

*class_methods2.rb*

```
class MyClass
    @@classvar = 1000
    @instvar = 1000

    def MyClass.classMethod
        if @instvar == nil then
            @instvar = 10
        else
            @instvar += 10
        end

        if @@classvar == nil then
            @@classvar = 10
        else
            @@classvar += 10
        end
    end

    def instanceMethod
        if @instvar == nil then
            @instvar = 1
        else
            @instvar += 1
        end

        if @@classvar == nil then
            @@classvar = 1
        else
            @@classvar += 1
        end

    end

    def showVars
        return "(instance method) @instvar = #{@instvar}, @@classvar = #{@@classvar}"
    end

    def MyClass.showVars
        return "(class method) @instvar = #{@instvar}, @@classvar = #{@@classvar}"
    end

end
```

注意，它分别声明并初始化了一个类变量和一个实例变量，`@@classvar` 和 `@instvar`。它的类方法 `classMethod` 将这两个变量都增加 10，而它的实例方法 `instanceMethod` 将这两个变量各增加 1。注意，我已经为类变量和实例变量都赋值了：

```
@@classvar = 1000
@instvar = 1000
```

我之前说过，通常不会以这种方式为实例变量分配初始值。这个规则的例外是当你为类的实例变量而不是从该类派生的对象分配一个值时。这个区别很快就会变得清楚。

我编写了一些代码，创建了 MyClass 的三个实例（`ob` 变量在循环的每次迭代中初始化为一个新的实例），然后调用了类方法和实例方法：

```
for i in 0..2 do
   ob = MyClass.new
   MyClass.classMethod
   ob.instanceMethod
   puts( MyClass.showVars )
   puts( ob.showVars )
end
```

类方法 `MyClass.showVars` 和实例方法 `showVars` 在每次循环迭代中显示 `@instvar` 和 `@@classvar` 的值。当你运行代码时，显示的值如下：

```
(class method) @instvar = 1010, @@classvar = 1011
(instance method) @instvar = 1, @@classvar = 1011
(class method) @instvar = 1020, @@classvar = 1022
(instance method) @instvar = 1, @@classvar = 1022
(class method) @instvar = 1030, @@classvar = 1033
(instance method) @instvar = 1, @@classvar = 1033
```

你可能需要仔细查看这些结果，以便了解这里发生了什么。总结来说，这是正在发生的事情：在类方法 `MyClass.classMethod` 和实例方法 `instanceMethod` 中的代码都会增加类变量和实例变量 `@@classvar` 和 `@instvar` 的值。

你可以清楚地看到，这两个方法（类方法在创建新对象时将 10 加到`@@classvar`上，而实例方法将其加 1）都会增加类变量。然而，每当创建一个新对象时，它的实例变量都会由`instanceMethod`初始化为 1。这是预期的行为，因为每个对象都有自己的实例变量副本，但所有对象共享一个唯一的类变量。可能不那么明显的是，类本身也有自己的实例变量，`@instvar`。这是因为，在 Ruby 中，类是一个对象，因此可以包含实例变量，就像任何其他对象一样。`MyClass`变量`@instvar`是由类方法`MyClass.classMethod`增加的：

```
@instvar += 10
```

当实例方法`showVars`打印`@instvar`的值时，它打印的是存储在特定对象`ob`中的值；`ob`的`@instvar`的初始值是`nil`（不是`MyClass`变量`@instvar`初始化时的 1,000），这个值在`instanceMethod`中被增加 1。

当类方法`MyClass.showVars`打印`@instvar`的值时，它打印的是存储在类本身中的值（换句话说，MyClass 的`@instvar`与`ob`的`@instvar`是不同的变量）。但当任一方法打印类变量`@@classvar`的值时，值是相同的。

只需记住，类变量只有一个副本，但实例变量可能有多个副本。如果这仍然让你感到困惑，请查看`*inst_vars.rb*`程序：

*inst_vars.rb*

```
class MyClass
   @@classvar = 1000
   @instvar = 1000

   def MyClass.classMethod
      if @instvar == nil then
         @instvar = 10
      else
         @instvar += 10
      end
   end

   def instanceMethod
      if @instvar == nil then
         @instvar = 1
      else
         @instvar += 1
      end
   end
end

ob = MyClass.new
puts MyClass.instance_variable_get(:@instvar)
puts( '--------------' )
for i in 0..2 do
   # MyClass.classMethod
   ob.instanceMethod
   puts( "MyClass @instvar=#{MyClass.instance_variable_get(:@instvar)}")
   puts( "ob @instvar= #{ob.instance_variable_get(:@instvar)}" )
end
```

这次，你不再在循环的每次迭代中创建新的对象实例，而是在一开始就创建一个单独的实例(`ob`)。当调用`ob.instanceMethod`时，`@instvar`增加 1。

在这里，我使用了一个小技巧来查看类和方法内部，并使用 Ruby 的`instance_variable_get`方法检索`@instvar`的值（当我介绍动态规划时，我会回到这个话题第二十章)：

```
puts( "MyClass @instvar= #{MyClass.instance_variable_get(:@instvar)}" )
puts( "ob @instvar= #{ob.instance_variable_get(:@instvar)}" )
```

因为只有当你增加属于对象`ob`的`@instvar`时，它的`@instvar`值才会从 1 增加到 3，这是`for`循环执行的结果。但属于`MyClass`类的`@instvar`永远不会增加；它保持在初始值 1,000：

```
1000
--------------
MyClass @instvar= 1000
ob @instvar= 1
MyClass @instvar= 1000
ob @instvar= 2
MyClass @instvar= 1000
ob @instvar= 3
```

但现在让我们取消注释这一行：

```
MyClass.classMethod
```

这调用了一个类方法，该方法将`@instvar`增加 10。这次当你运行程序时，你会看到，就像之前一样，`ob`的`@instvar`变量在每次循环迭代时增加 1，而`MyClass`的`@instvar`变量增加 10：

```
1000
--------------
MyClass @instvar= 1010
ob @instvar= 1
MyClass @instvar= 1020
ob @instvar= 2
MyClass @instvar= 1030
ob @instvar= 3
```

类是一个对象

要理解类的实例变量，只需记住一个类是一个对象（实际上，它是 `Class` 类的一个实例！）MyClass “类对象”有自己的实例变量（`@instvar`），就像 `ob` 对象有自己的实例变量一样（这里也恰好也叫做 `@instvar`）。尽管名字相同，但这两个变量是不同的：一个属于类本身；另一个属于从该类创建的每个对象内部。实例变量始终是对象实例独有的，因此没有任何两个对象（甚至是一个像 MyClass 这样的对象，它也恰好是一个类！）可以共享单个实例变量。

# Ruby 构造器：new 还是 initialize？

在 第一章 中，我对 `new` 和 `initialize` 方法进行了简要的解释。在那个阶段，你还没有检查 Ruby 的类方法和实例方法以及变量的区别，因此无法全面讨论 `new` 和 `initialize` 如何协同工作。因为这些方法非常重要，所以现在我们将更详细地研究它们。

负责创建对象的那个方法被称为 *构造器*。在 Ruby 中，构造器方法叫做 `new`。`new` 方法是一个类方法，一旦它创建了一个对象，如果存在这样的方法，它将运行一个名为 `initialize` 的实例方法。

简而言之，`new` 方法是构造器，而 `initialize` 方法用于在对象创建后立即初始化任何变量的值。但为什么你不能只是编写自己的 `new` 方法并在其中初始化变量呢？好吧，让我们试试：

*new.rb*

```
class MyClass
   def initialize( aStr )
      @avar = aStr
   end

   def MyClass.new( aStr )
      super
      @anewvar = aStr.swapcase
   end
end

ob = MyClass.new( "hello world" )
puts( ob )
puts( ob.class )
```

在这里，我编写了一个以 `super` 关键字开始的 `MyClass.new` 方法，以调用其超类的 `new` 方法。然后我创建了一个字符串实例变量，`@anewvar`。那么我最终得到了什么？不是，正如你可能想象的那样，一个包含字符串变量的新 MyClass 对象。记住，在 Ruby 中，方法最后评估的表达式是该方法的返回值。这里 `new` 方法最后评估的表达式是一个字符串。我评估这个：

```
ob = MyClass.new( "hello world" )
```

我展示了新创建的 `ob` 对象及其类：

```
puts( ob )
puts( ob.class )
```

这就是输出结果：

```
HELLO WORLD
String
```

这证明了 `MyClass.new` 返回一个字符串，并且这个字符串（*而不是* MyClass 对象）被分配给了变量 `ob`。如果你觉得这很困惑，不要慌张。这个故事的意义在于，覆盖 `new` *确实* 是令人困惑的，通常不是一个好主意。除非你有非常充分的理由这样做，否则你应该避免尝试覆盖 `new` 方法。

# 单例方法

单例方法是一个属于单个对象而不是整个类的方法。Ruby 类库中的许多方法都是单例方法。这是因为，如前所述，每个类都是 Class 类型的一个对象。或者，简单地说，每个类的类是 Class。这对所有类都适用——无论是你自己定义的还是 Ruby 类库提供的：

*class_classes.rb*

```
class MyClass
end

puts( MyClass.class )    #=> Class
puts( String.class )     #=> Class
puts( Object.class )     #=> Class
puts( Class.class )      #=> Class
puts( IO.class )         #=> Class
```

现在，一些类也有类方法——即属于 Class 对象本身的方法。从某种意义上说，这些是 Class 对象的单例方法。确实，如果你评估以下内容，你会看到一个与 IO 类方法名称匹配的方法名称数组：

```
p( IO.singleton_methods )
```

这将显示以下内容：

```
[:new, :open, :sysopen, :for_fd, :popen, :foreach, :readlines,
 :read, :binread, :select, :pipe, :try_convert, :copy_stream]
```

如前所述，当你编写自己的类方法时，你通过在方法名前加上类的名称来这样做：

```
def MyClass.classMethod
```

结果表明，在创建特定对象的单例类时，你可以使用类似的语法。这次，你需要在方法名前加上对象名：

```
def myObject.objectMethod
```

查找对象的祖先类

最终，所有类都从 Object 类派生。在 Ruby 1.9 中，Object 类本身是从 BasicObject 类派生的（参见第二章）。即使是`Class`类也是如此！为了证明这一点，尝试运行*class_hierarchy.rb*程序：

```
def showFamily( aClass )
    if (aClass != nil) then
        puts( "#{aClass} :: about to recurse with aClass.superclass =
 #{aClass.superclass.inspect}" )
        showFamily( aClass.superclass )
    end
end
```

将类名传递给此方法以追踪其祖先类的家族树。例如，尝试这样做：

```
showFamily(File)
```

在 Ruby 1.9 中，这将显示以下内容：

```
File :: about to recurse with aClass.superclass = IO
IO :: about to recurse with aClass.superclass = Object
Object :: about to recurse with aClass.superclass = BasicObject
BasicObject :: about to recurse with aClass.superclass = nil
```

*class_hierarchy.rb*

让我们来看一个具体的例子。假设你有一个包含许多不同物种的 Creature 对象的程序（也许你是一位兽医，动物园的负责人，或者像本书的作者一样，是一位热情的冒险游戏玩家）；每个生物都有一个名为`talk`的方法，用于显示每个生物通常发出的声音。

这是我的 Creature 类和一些生物对象：

*singleton_meth1.rb*

```
class Creature
   def initialize( aSpeech )
      @speech = aSpeech
   end

   def talk
      puts( @speech )
   end
end

cat = Creature.new( "miaow" )
dog = Creature.new( "woof" )
budgie = Creature.new( "Who's a pretty boy, then!" )
werewolf = Creature.new( "growl" )
```

突然间，你意识到这些生物中只有一个具有额外的特殊行为。在满月之夜，狼人不仅会说话（“咆哮”），还会嗥叫（“How-oo-oo-oo-oo！”）。它真的需要一个`howl`方法。

你可以将这样的方法添加到 Creature 类中，但最终你会得到嗥叫的狗、猫和鹦鹉——这并不是你想要的。你可以创建一个新的从 Creature 派生的 Werewolf 类，但你将永远只有一个狼人（遗憾的是，它们是一种濒危物种），所以为什么你需要一个整个类来代表它呢？难道不是更有意义有一个与所有其他生物对象相同的狼人*对象*，除了它还有一个`howl`方法吗？好吧，让我们通过给狼人一个它自己的单例方法来实现这一点。下面是操作步骤：

```
def werewolf.howl
   puts( "How-oo-oo-oo-oo!" )
end
```

哎，你可以做得更好！它只在满月时嗥叫，所以让我们确保，如果月亮是新的，它只是咆哮。这是我的完成方法：

```
def werewolf.howl
    if FULLMOON then
      puts( "How-oo-oo-oo-oo!" )
   else
      talk
   end
end
```

注意，尽管这个方法是在 Creature 类外部声明的，但它仍然能够调用实例方法`talk`。这是因为`howl`方法现在“位于”狼人对象内部，因此在那个对象内部与`talk`方法具有相同的范围。然而，它并不位于狼人同伴的任何地方；`howl`方法只属于他一个人。尝试调用`budgie.howl`，Ruby 会告诉你`howl`是一个未定义的方法。

现在，如果你正在为自己的代码进行调试，由于未定义的方法而导致程序崩溃可能是可以接受的；然而，如果你的程序在“最终用户”的广阔、恶劣的世界中这样做，这绝对是不可以接受的。

如果你认为未定义的方法可能会成为问题，你可以在尝试使用它之前通过测试单例方法是否存在来采取避免措施。Object 类有一个`singleton_methods`方法，它返回一个包含单例方法名称的数组。你可以使用 Array 类的`include?`方法测试一个方法名称是否包含。例如，在*singleton_meth2.rb*中，我编写了一个“打开盒子”游戏，其中包含多个 Box 对象，只有一个盒子打开时包含奖品。我给这个特殊盒子对象命名为`starprize`，并给它添加了一个名为`congratulate`的单例方法：

*singleton_meth2.rb*

```
starprize = Box.new( "Star Prize" )
def starprize.congratulate
    puts( "You've won a fabulous holiday in Grimsby!" )
end
```

当`starprize`盒子被打开时，应该调用`congratulate`方法。这段代码（其中`item`是一个 Box 对象）确保当打开其他盒子时，不会调用这个方法（它不存在于任何其他对象中）：

```
if item.singleton_methods.include?("congratulate") then
    item.congratulate
end
```

检查方法有效性的另一种方式是将该方法名称作为符号（一个以冒号开头的标识符）传递给 Object 类的`respond_to?`方法：

```
if item.respond_to?( :congratulate ) then
   item.congratulate
end
```

### 注意

你将在第二十章中看到处理不存在方法的另一种方法。

# 单例类

单例方法是一个属于单个对象的方法。另一方面，单例类是一个定义单个对象的类。困惑？我也是。让我们更仔细地看看。

假设你创建了数十个对象，每个对象都是 Object 类的一个实例。自然地，它们都可以访问继承的方法，例如`inspect`和`class`。但现在你决定你只想有一个特殊对象（为了多样性，让我们称他为`ob`），它有一个特殊的方法（让我们称它为`blather`）。

你不想为这个单一对象定义一个全新的类，因为你永远不会再次创建任何具有`blather`方法的更多对象。所以，你创建了一个专门为小`ob`设计的类。

你不需要给这个类命名。你只需通过在`class`关键字和对象名称之间放置一个`<<`来告诉它将自己附加到`ob`上。然后你以通常的方式向这个类中添加代码：

*singleton_class.rb*

```
ob = Object.new
    # singleton class
class << ob
    def blather( aStr )
        puts("blather, blather #{aStr}")
    end
end
```

现在 `ob`，只有 `ob`，不仅具有 Object 类的所有常规方法；它还具有它自己的特殊匿名类的方法（这里只是 `blather` 方法，原则上可以有更多）：

```
ob.blather( "weeble" )    #=> "blather, blather weeble"
```

如果你一直很注意，你可能已经注意到单例类似乎在做与单例方法非常相似的事情。使用单例类，我可以创建一个对象，然后添加封装在匿名类中的额外方法。使用单例方法，我可以创建一个对象，然后逐个添加方法：

*singleton_class2.rb*

```
ob2 = Object.new

def ob2.blather( aStr )        # <= this is a singleton method
   puts( "grippity, grippity #{aStr}" )
end

ob2.blather( "ping!" )         #=> grippity, grippity ping!
```

同样，我可以重写“star prize”程序。在前一个版本中，我为名为 `starprize` 的对象添加了一个单例方法 `congratulate`。我同样可以创建一个包含 `congratulate` 方法的单例类：

```
starprize = MyClass.new( "Star Prize" )

class << starprize
   def congratulate
      puts( "You've won a fabulous holiday in Grimsby!" )
   end
end
```

实际上，这种相似性远不止表面。前述代码的最终结果是 `congratulate` 成为了 `starprize` 的单例方法。我可以通过检查 `item` 对象可用的单例方法数组是否包含名称 `congratulate` 来验证这一点：

```
if item.singleton_methods.include?(:congratulate)    # Ruby 1.9
```

在 Ruby 1.9 中，`singleton_methods` 方法返回一个表示方法名的符号数组。这就是为什么我在前面的代码中使用符号 `:congratulate`。然而，在 Ruby 1.8 中，`singleton_methods` 返回一个字符串数组。所以，如果你使用 Ruby 1.8，你应该确保使用以下使用字符串参数 `"congratulate"` 的测试：

```
if item.singleton_methods.include?("congratulate")    # Ruby 1.8
```

### 注意

单例方法和单例类之间的区别是什么？简短的回答是，没有太多区别。这两个语法提供了向特定对象添加方法的不同方式，而不是将这些方法构建到其定义类中。

# 重写方法

有时候你可能想重新定义某个类中已经存在的方法。你之前已经这样做过了，例如，当你创建了具有自己的 `to_s` 方法的类以返回字符串表示时。从 Object 类向下，每个 Ruby 类都有一个 `to_s` 方法。Object 类的 `to_s` 方法返回类名和对象的唯一标识符的十六进制表示。然而，许多 Ruby 类都有自己的特殊版本的 `to_s`。例如，`Array.to_s` 连接并返回数组中的值。

当一个类中的方法替换了祖先类中同名的那个方法时，我们称其为*重写*该方法。你可以重写标准类库中定义的方法，如`to_s`，以及你自己的类中定义的方法。如果你需要向现有方法添加新行为，记得在重写方法的开头使用`super`关键字调用超类的方法。

这里有一个例子：

*override.rb*

```
class MyClass
   def sayHello
      return "Hello from MyClass"
   end

   def sayGoodbye
      return "Goodbye from MyClass"
   end
end

class MyOtherClass < MyClass
    def sayHello            #overrides (and replaces) MyClass.sayHello
        return "Hello from MyOtherClass"
    end

        # overrides MyClass.sayGoodbye   but first calls that method
        # with super. So this version "adds to" MyClass.sayGoodbye
    def sayGoodbye
        return super << " and also from MyOtherClass"
    end

        # overrides default to_s method
    def to_s
        return "I am an instance of the #{self.class} class"
    end
end
```

# 公共、受保护和私有方法

在某些情况下，你可能想限制你方法的“可见性”，以确保它们不能被出现在方法中的类外部的代码调用。

这可能在你定义的类需要执行某些不打算公开消费的功能时很有用。通过限制这些方法的访问权限，你可以防止程序员为了自己的恶意目的使用它们。这意味着你可以在以后阶段更改这些方法的实现，而不用担心会破坏其他人的代码。

Ruby 提供了三种方法可访问级别：

+   public

+   protected

+   private

如其名所示，公共方法是可访问性最高的，而私有方法是可访问性最低的。除非你明确指定，否则所有方法都是公共的。当一个方法是公共的，它就可以被定义其类的对象之外的世界使用。

当一个方法是私有的，它只能由定义其类的对象内部的其他方法使用。

受保护的方法通常与私有方法以相同的方式工作，但有一个微小但重要的区别：除了对当前对象的方法可见外，当第二个对象在第一个对象的作用域内时，受保护的方法对同一类型的对象也是可见的。

当你看到工作示例时，私有和受保护方法之间的区别可能更容易理解。考虑这个类：

*pub_prot_priv.rb*

```
class MyClass

    private
        def priv
             puts( "private" )
        end

    protected
        def prot
             puts( "protected" )
        end

   public
        def pub
             puts( "public" )
        end

        def useOb( anOb )
             anOb.pub
             anOb.prot
             anOb.priv
        end
end
```

我声明了三个方法，每个可访问级别一个。这些级别是通过在 `private`、`protected` 或 `public` 前放置一个或多个方法来设置的。指定的可访问级别对所有后续方法都有效，直到指定了另一个访问级别。

### 注意

`public`、`private` 和 `protected` 可能看起来像关键字。但实际上，它们是 Module 类的方法。

最后，我的类有一个公共方法 `useOb`，它接受一个 `MyOb` 对象作为参数，并调用该对象的三个方法 `pub`、`prot` 和 `priv`。现在，让我们看看如何使用 `MyClass` 对象。首先，我将创建该类的两个实例：

```
myob = MyClass.new
myob2 = MyClass.new
```

现在，我尝试依次调用这三个方法：

```
myob.pub         # This works! Prints out "public"
myob.prot        # This doesn't work! I get an error
myob.priv        # This doesn't work either - another error
```

从前面的例子中，看起来公共方法（正如预期的那样）可以从对象外部看到。但私有和受保护的方法都是不可见的。既然如此，受保护的方法有什么用？另一个例子应该有助于澄清这一点：

```
myob.useOb( myob2 )
```

这次，我正在调用 `myob` 对象的公共方法 `useOb`，并将第二个对象 `myob2` 作为参数传递给它。需要注意的是，`myob` 和 `myob2` 是同一类的实例。现在，回想一下我之前说过的话：*除了对当前对象的方法可见外，当第二个对象在第一个对象的作用域内时，受保护的方法对同一类型的对象也是可见的*。

这可能听起来像是胡言乱语。让我们看看我是否能从中找出一些道理。在程序中，当 `myob2` 作为参数传递给 `myob` 的一个方法时，第一个 MyClass 对象（这里 `myob`）在其作用域内有一个第二个 MyClass 对象。当这种情况发生时，你可以将 `myob2` 视为存在于 `myob` “内部”。现在 `myob2` 与“包含”对象 `myob` 共享作用域。在这种情况下——当两个相同类的对象位于该类定义的作用域内时——该类中任何对象的受保护方法都变得可见。

在当前情况下，对象 `myob2` 的受保护方法 `prot`（或者至少是“接收”`myob2` 的参数，这里称为 `anob`）变得可见并可执行。然而，它的私有参数是不可见的：

```
def useOb( anOb )
   anOb.pub
   anOb.prot    # protected method can be called
   anOb.priv    # calling a private method results in an error
end
```

深入挖掘

在这里，你将学习更多关于方法内部代码的可见性以及定义单例方法的另一种方式。

后代类中的受保护和私有方法

本章中描述的相同访问规则也适用于调用祖先和后代对象的方法。也就是说，当你将一个对象传递给一个方法（作为参数）时，该方法的接收对象（换句话说，方法所属的对象）与参数对象具有相同的类，参数对象可以调用该类的公共和受保护方法，但不能调用其私有方法。

例如，查看 *protected.rb* 程序。在这里，我创建了一个名为 `myob` 的 MyClass 对象和一个名为 `myotherob` 的 MyOtherClass 对象，其中 MyOtherClass 继承自 MyClass：

*protected.rb*

```
class MyClass

    private
        def priv( aStr )
            return aStr.upcase
        end

    protected
        def prot( aStr )
            return aStr << '!!!!!!'
        end

    public

        def exclaim( anOb )  # calls a protected method
            puts( anOb.prot( "This is a #{anOb.class} - hurrah" ) )
        end

        def shout( anOb )    # calls a private method
            puts( anOb.priv( "This is a #{anOb.class} - hurrah" ) )
        end

end

class MyOtherClass < MyClass

end

class MyUnrelatedClass

end
```

现在，我创建了这三个类中的每一个对象，并尝试将 `myotherob` 作为参数传递给 `myob` 的公共方法 `shout`：

```
myob = MyClass.new
myotherob = MyOtherClass.new
myunrelatedob = MyUnrelatedClass.new
```

如果你从代码存档中加载此程序，你会看到其中包含许多行代码，这些代码中的三个对象试图执行 `shout` 和 `exclaim` 方法。其中许多尝试注定会失败，因此已被注释掉。然而，在测试代码时，你可能希望逐个取消注释每个方法调用以查看结果。这是我的第一次尝试：

```
myob.shout( myotherob )     # fails
```

在这里，`shout` 方法在参数对象上调用私有方法 `priv`：

```
def shout( anOb )    # calls a private method
   puts( anOb.priv( "This is a #{anOb.class} - hurrah" ) )
end
```

这将不起作用！Ruby 抱怨 `priv` 方法是私有的。

类似地，如果我将顺序反过来——也就是说，通过传递祖先对象 `myob` 作为参数并在后代对象上调用 `shout` 方法——我也会遇到相同的错误：

```
myotherob.shout( myob )     # fails
```

MyClass 类还有一个公共方法，`exclaim`。这个方法调用受保护的方法，`prot`：

```
def exclaim( anOb )  # calls a protected method
   puts( anOb.prot( "This is a #{anOb.class} - hurrah" ) )
end
```

现在，我可以将 MyClass 对象 `myob` 或 MyOtherClass 对象 `myotherob` 作为参数传递给 `exclaim` 方法，当调用受保护的方法时不会发生错误：

```
myob.exclaim( myotherob )        # This is OK
myotherob.exclaim( myob )        # And so is this...
myob.exclaim( myunrelatedob )    # But this won't work
```

不言而喻，这只有在两个对象（点左侧的接收对象和传递给方法的参数）共享相同的继承线时才有效。如果你传递一个无关的对象作为参数，无论它们的保护级别如何，你都无法调用接收类的任何方法。

侵犯私有方法的隐私

私有方法整个点在于它不能从属于它的对象的作用域之外被调用。所以，这不会起作用：

*send.rb*

```
class X
    private
        def priv( aStr )
             puts("I'm private, " << aStr)
        end
end

ob = X.new
ob.priv( "hello" )      # This fails
```

然而，结果证明，Ruby 提供了一种“退出条款”（或者我或许应该说“进入条款”）的形式，即名为 `send` 的方法。

`send` 方法调用与符号（以冒号开始的标识符，如 `:priv`）名称匹配的方法，该符号作为第一个参数传递给 `send`，如下所示：

```
ob.send( :priv, "hello" )    # This succeeds
```

在符号（如字符串“hello”）之后提供的任何参数都按正常方式传递给指定的方法。

使用 `send` 来获取对私有方法的公共访问通常不是一个好主意。毕竟，如果你需要访问某个方法，为什么一开始要将其设置为私有呢？谨慎使用此技术或根本不使用。

单例类方法

之前，我通过将方法名附加到类名上来创建类方法，如下所示：

```
def MyClass.classMethod
```

有一种“快捷”语法来做这件事。以下是一个示例：

*class_methods3.rb*

```
class MyClass

   def MyClass.methodA
      puts("a")
   end

   class << self
      def methodB
         puts("b")
      end

      def methodC
         puts("c")
      end
   end

end
```

这里，`methodA`、`methodB` 和 `methodC` 都是 MyClass 的类方法；`methodA` 是使用之前的方法声明的语法：

```
def *`ClassName`*.*`methodname`*
```

但 `methodB` 和 `methodC` 是使用实例方法的语法声明的：

```
def *`methodname`*
```

所以，为什么它们最终成为类方法呢？这完全是因为方法声明被放置在这个代码中：

```
class << self
    # some method declarations
end
```

这可能会让你想起声明单例类所使用的语法。例如，在 *singleton_class.rb* 程序中，你可能还记得我首先创建了一个名为 `ob` 的对象，然后给它添加了一个名为 `blather` 的方法：

```
class << ob
   def blather( aStr )
      puts("blather, blather #{aStr}")
   end
end
```

这里 `blather` 方法是 `ob` 对象的单例方法。同样，在 *class_methods3.rb* 程序中，`methodB` 和 `methodC` 方法是 `self` 的单例方法——而 `self` 正好是 MyClass 类。你可以通过使用 `<<` 后跟类名，从类定义外部添加单例方法，如下所示：

```
class << MyClass
   def methodD
      puts( "d" )
   end
end
```

最后，代码通过首先打印所有可用的单例方法名称，然后调用它们来检查所有四个方法确实都是单例方法：

```
puts( MyClass.singleton_methods.sort )
MyClass.methodA
MyClass.methodB
MyClass.methodC
MyClass.methodD
```

这会显示以下内容：

```
methodA
methodB
methodC
methodD
a
b
c
d
```

嵌套方法

你可以嵌套方法；也就是说，你可以编写包含其他方法的方法。这为你提供了一种将长方法分成可重用块的方法。例如，如果方法 `x` 需要在几个不同的点执行计算 `y`，你可以在 `x` 方法（以下示例中的方法称为 `outer_x`、`nested_y` 和 `nested_z` 以便清晰）中放置 `y` 方法：

*nested_methods.rb*

```
class X

   def outer_x
      print( "x:" )

      def nested_y
         print("ha! ")
      end

      def nested_z
         print( "z:" )
         nested_y
      end

      nested_y
      nested_z
   end

end
```

嵌套方法最初在它们定义的作用域之外是不可见的。所以，在上面的例子中，尽管`nested_y`和`nested_z`可以从`outer_x`内部调用，但它们可能无法被任何其他代码调用：

```
ob = X.new
ob.outer_x         #=> x:ha! z:ha!
```

如果在之前的代码中，你调用的是`ob.outer_x`而不是`ob.nested_y`或`ob.nested_z`，你会看到一个错误信息，因为在这个阶段`nested_y`和`nested_z`方法将不可见。然而，当你运行一个包含嵌套方法的方法时，那些嵌套方法*将会*被带到该方法的外部作用域中！

*nested_methods2.rb*

```
class X
   def x
      print( "x:" )
      def y
         print("y:")
      end

      def z
         print( "z:" )
         y
      end
   end
end

ob = X.new
ob.x        #=> x:
puts
ob.y        #=> y:
puts
ob.z        #=> z:y:
```

为了看到这个的另一个例子，尝试再次运行*nested_methods.rb*代码，但这次取消注释所有三个方法调用。这次，当`outer_x`方法执行时，它会将`nested_y`和`nested_z`带到作用域中，因此对这两个嵌套方法的调用现在成功了：

```
ob.outer_x     #=> x:ha! z:ha!
ob.nested_y    #=> ha!
ob.nested_z    #=> z:ha!
```

方法名

最后一点，值得提一下的是，Ruby 中的方法名几乎总是以小写字母开头，就像这样：

```
def fred
```

然而，这只是一个*约定*，并不是*义务*。也可以以大写字母开头命名方法名，就像这样：

```
def Fred
```

由于`Fred`方法看起来像是一个常量（它以大写字母开头），在调用它时你需要通过添加括号来告诉 Ruby 它是一个方法：

*method_names.rb*

```
Fred         # <= Ruby complains 'uninitialized constant'
Fred()       # <= Ruby calls the Fred method
```

总体来说，坚持使用以小写字母开头的方法名约定会更好。
