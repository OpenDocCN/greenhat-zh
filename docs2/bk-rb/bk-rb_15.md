# 第十五章：Marshal

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

Ruby 的 Marshal 库提供了一种保存和加载数据的替代方法。它有一组类似 YAML 的方法，使你能够将数据保存到磁盘并从磁盘加载数据。

# 保存和加载数据

将以下程序与上一章的 *yaml_dump2.rb* 进行比较：

*marshal1.rb*

```
f = File.open( 'friends.sav', 'w' )
Marshal.dump( ["fred", "bert", "mary"], f )
f.close

File.open( 'morefriends.sav', 'w' ){ |friendsfile|
    Marshal.dump( ["sally", "agnes", "john" ], friendsfile )
}

File.open( 'morefriends.sav' ){ |f|
    $arr= Marshal.load(f)
}
myfriends = Marshal.load(File.open( 'friends.sav' ))
morefriends = Marshal.load(File.open( 'morefriends.sav' ))

p( myfriends )    #=> ["fred", "bert", "mary"]
p( morefriends )  #=> ["sally", "agnes", "john"]
p( $arr )         #=> ["sally", "agnes", "john"]
```

这两个程序几乎完全相同，只是将每个 `YAML`（如 `YAML.dump` 和 `YAML.load`）的出现都替换成了 `Marshal`。此外，Marshal 是 Ruby 的标准库之一，所以你不需要 `require` 任何额外的文件来使用它。

如果你查看生成的数据文件（例如 *friends.sav*），你会立即看到有一个主要区别。然而，YAML 文件是纯文本格式，而 Marshal 文件是二进制格式。所以尽管你可能能够读取 *一些* 字符，例如字符串中的字符，但你无法简单地加载保存的数据并在文本编辑器中修改它。

与 YAML 类似，大多数数据结构可以通过简单地序列化顶层对象并在需要重建其下所有对象时加载它来自动序列化使用 Marshal。例如，看看我的小型冒险游戏程序。在前一章中，我解释了如何通过序列化和加载 Map 对象 `mymap`（见 Adventures in YAML 中的 *gamesave_y.rb*）来保存和恢复包含宝物的房间。你可以用 Marshal 而不是 YAML 来做同样的事情：

*gamesave_m.rb*

```
File.open( 'game.sav', 'w' ){ |f|
    Marshal.dump( mymap, f )    # save data to file
}

File.open( 'game.sav' ){ |f|
    mymap = Marshal.load(f)     # reload saved data from file
}
```

在一些特殊情况下，对象不能如此轻易地序列化。这些异常在 Ruby 的 `Marshal` 模块（*marshal.c*）的代码中有记录，它指出，“如果待序列化的对象包括绑定、过程或方法对象、IO 类的实例或单例对象，将引发 TypeError。” 我将在讨论如何使用序列化保存单例对象时展示一个例子。

# 保存时省略变量

与 YAML 序列化一样，使用 Marshal 序列化时可以限制要保存的变量。在 YAML 中，你通过编写一个名为 `to_yaml_properties` 的方法来完成此操作。对于 Marshal，你需要编写一个名为 `marshal_dump` 的方法。在这个方法的代码中，你应该创建一个包含要保存的 *实际变量* 的数组（在 YAML 中，你创建了一个包含变量 *名称* 的字符串数组的数组）。

这是一个示例：

```
def marshal_dump
    [@variable_a, @variable_b]
end
```

另一个区别是，使用 YAML，你可以简单地加载数据以重新创建对象。而使用 Marshal，你需要添加一个名为 `marshal_load` 的特殊方法，将任何加载的数据作为参数传递给它。当你调用 `Marshal.load` 时，它将自动被调用，并将数据以数组的形式传递。之前保存的对象可以从这个数组中解析出来。你还可以为在保存数据时省略的任何变量（例如这里的 `@some_other_variable`）分配值：

```
def marshal_load(data)
    @variable_a = data[0]
    @variable_b = data[1]
    @some_other_variable = "a default value"
end
```

这是一个完整的程序，它保存和恢复了 `@num` 和 `@arr` 变量，但省略了 `@str`：

*limit_m.rb*

```
class Mclass
    def initialize(aNum, aStr, anArray)
        @num = aNum
        @str = aStr
        @arr = anArray
    end

    def marshal_dump
        [@num, @arr]
    end

    def marshal_load(data)
        @num = data[0]
        @arr = data[1]
        @str = "default"
   end
end

ob = Mclass.new( 100, "fred", [1,2,3] )
p( ob )
#=> #<Mclass:0x2be7278 @num=100, @str="fred", @arr=[1, 2, 3]>

marshal_data = Marshal.dump( ob )
ob2 = Marshal.load( marshal_data )
p( ob2 )
#=> #<Mclass:0x2be70e0 @num=100, @str="default", @arr=[1, 2, 3]>
```

注意，尽管这里的序列化是在内存中完成的，但相同的技巧也可以用于使用 Marshal 将对象保存到磁盘和从磁盘加载。

# 保存单例

让我们看看之前提到的一个具体问题，即无法使用 marshaling 来保存和加载单例。在 *singleton_m.rb* 中，我创建了一个 Object 实例，名为 `ob`，然后以单例类的形式扩展它，并给它添加了一个额外的 `xxx` 方法：

*singleton_m.rb*

```
ob = Object.new

class << ob
    def xxx( aStr )
        @x = aStr
    end
end
```

当我尝试使用 `Marshal.dump` 将这些数据保存到磁盘时，Ruby 显示了一个错误信息：“singleton can’t be dumped (TypeError)。”

## YAML 和 单例

在考虑如何处理这个问题之前，让我们简要地看看 YAML 在这种情况下会如何应对。程序 *singleton_y.rb* 尝试使用 `YAML.dump` 保存我刚才创建的单例，与 `Marshal.dump` 不同，它成功了——好吧，算是吧：

*singleton_y.rb*

```
# YAML version of singleton-save
ob.xxx( "hello world" )

File.open( 'test.yml', 'w' ){ |f|
    YAML.dump( ob, f )
}

ob.xxx( "new string" )

File.open( 'test.yml' ){ |f|
    ob = YAML.load(f)
}
```

如果你查看保存的 YAML 文件，名为 *test.yml*，你会发现它定义了一个普通的 Object 实例，并附加了一个名为 `x` 的变量，该变量的字符串值为 `hello world`：

```
--- !ruby/object
x: hello world
```

这一切看起来都很好。然而，当你通过加载保存的数据重新构建对象时，新的 `ob` 将是一个标准的 Object 实例，它恰好包含一个额外的实例变量，`@x`。由于它不再是原始的单例，这个 `ob` 将无法访问在该单例中定义的任何方法（这里是指 `xxx` 方法）。因此，尽管 YAML 序列化在保存和加载数据项方面对在单例中创建的数据项更为宽容，但它不会在重新加载保存的数据时自动重新创建单例。

## Marshal 和 单例

现在，让我们回到这个程序的 Marshal 版本。我需要做的第一件事是找到至少保存和加载数据项的方法。一旦我做到了这一点，我就会尝试找出如何在重新加载时重建单例。

要保存特定的数据项，我可以定义前面解释过的`marshal_dump`和`marshal_load`方法（参见*limit_m.rb*）。这些方法通常应该定义在单例派生的类中，而不是单例本身。这是因为，正如前面解释的，当数据被保存时，它将以单例派生类的表示形式存储。这意味着尽管你确实可以向从类 X 派生的单例添加`marshal_dump`，但在重构对象时，你将加载一个通用类型 X 的对象的数据，而不是特定单例实例的数据。

此代码创建了一个类 X 的单例`ob`，保存其数据，然后重新创建了一个类 X 的通用对象：

*singleton_m2.rb*

```
class X
    def marshal_dump
        [@x]
    end

    def marshal_load(data)
        @x = data[0]
    end
end

ob = X.new

class << ob
    def xxx( aStr )
        @x = aStr
    end
end

ob.xxx( "hello" )
p( ob )

File.open( 'test2.sav', 'w' ){ |f|
    Marshal.dump( ob, f )
}

ob.xxx( "new string" )
p( ob )
File.open( 'test2.sav' ){ |f|
    ob = Marshal.load(f)
}

p( ob )
```

此处使用的代码使用`Marshal.dump`保存类 X 的对象`ob`，然后调用单例方法`xxx`在重新加载保存的数据之前将不同的字符串分配给`@x`变量，然后使用`Marshal.load`重新加载保存的数据，并使用这些数据重新创建对象。使用`p()`显示`ob`的内容，在保存之前，然后在新字符串分配给它之后，最后在重新加载时再次显示。这让你可以验证在重新加载的对象重构时`@x`被分配了保存时的值：

```
#<X:0x2b86cc0 @x="hello">       # value when saved
#<X:0x2b86cc0 @x="new string">  # new value then assigned
#<X:0x2b869f0 @x="hello">       # value after saved data loaded
```

在包含的数据方面，保存的对象和重新加载的对象是相同的。然而，重新加载的对象对单例类一无所知。单例类中包含的`xxx`方法并不构成重构对象的一部分。因此，以下操作将失败：

```
ob.xxx( "this fails" )
```

此代码的 Marshal 版本与前面给出的 YAML 版本等效。它正确地保存和恢复了数据，但它不会重构单例。那么，如何从保存的数据中重构单例呢？无疑，有许多巧妙和微妙的方法可以实现这一点。然而，我将选择一个非常简单的技术：

*singleton_m3.rb*

```
FILENAME = 'test2.sav'

class X
    def marshal_dump
        [@x]
    end

    def marshal_load(data)
        @x = data[0]
    end
end

ob = X.new

# a) if File exists, load data into ob - a generic X object
if File.exists?(FILENAME) then
    File.open(FILENAME){ |f|
        ob = Marshal.load(f)
    }
else
    puts( "Saved data can't be found" )
end
# b) Now transform ob in a singleton
class << ob
    def xxx=( aStr )
        @x = aStr
    end

    def xxx
        return @x
    end
end
```

此代码首先检查是否可以找到包含保存数据的文件。（这个示例被故意保持简单——在实际应用中，你当然需要编写一些异常处理代码来处理读取无效数据的可能性。）如果找到文件，数据将被加载到一个通用类型`X`的对象中。

只有完成此操作后，此对象才会以通常的方式“转换”为单例。换句话说，对象被加载，然后执行以`class << ob`开始的代码（简单地因为单例创建代码在加载代码之后，所以由 Ruby 解释器按顺序执行）。这为对象提供了额外的`xxx`单例方法。然后你可以将新数据保存回磁盘，并在稍后阶段按照前面解释的方法重新加载和重新创建修改后的单例：

```
if ob.xxx == "hello" then
   ob.xxx = "goodbye"
else
   ob.xxx = "hello"
end

File.open( FILENAME, 'w' ){ |f|
    Marshal.dump( ob, f )
}
```

如果你想在真实的应用程序中保存和加载单例，单例“重建”代码自然可以有自己的方法，这样你就不必像上一个例子那样依赖于它在代码中的位置。

*singleton_m4.rb*

```
def makeIntoSingleton( someOb )
   class << someOb
      def xxx=( aStr )
         @x = aStr
      end

      def xxx
         return @x
      end
   end
   return someOb
end
```

深入挖掘

如果你尝试加载使用不同版本的 Marshal 库保存的数据，可能会遇到问题。在这里，你将学习如何验证 Marshal 的版本。

Marshal 版本号

Marshal 库的嵌入式文档（一个名为 *marshal.c* 的 C 语言文件）声明如下：“序列化的数据存储了与对象信息一起的主版本号和次版本号。在正常使用中，序列化只能加载与相同主版本号和相同或更低次版本号的数据。”

这明显提出了一个潜在问题，即序列化创建的数据文件格式可能与当前的 Ruby 应用程序不兼容。顺便提一下，Marshal 版本号不依赖于 Ruby 版本号，因此仅基于 Ruby 版本做出兼容性的假设是不安全的。

这种不兼容的可能性意味着在尝试加载之前，你应该始终检查保存数据的版本号。但你怎么获取版本号呢？再次，嵌入式文档提供了线索。它声明，“你可以通过读取序列化数据的前两个字节来提取版本号。”

Ruby 1.8 提供了以下示例：

```
str = Marshal.dump("thing")
RUBY_VERSION       #=> "1.8.0"
str[0]             #=> 4
str[1]             #=> 8
```

好的，那么让我们尝试在一个完整的代码块中实现这个功能。下面是代码：

*version_m.rb*

```
x = Marshal.dump( "hello world" )
print( "Marshal version: #{x[0]}:#{x[1]}\n" )
```

在之前的代码中，`x` 是一个字符串，其前两个字节是主版本号和次版本号。在 Ruby 1.8 中，它会输出以下内容：

```
Marshal version: 4:8
```

然而，在 Ruby 1.9 中，没有显示任何数字。这是因为 Ruby 1.8 中前两个字节作为整数返回，但在 Ruby 1.9 中作为字符串返回。这些字符串不一定可打印。你可以通过使用 `p()` 方法来显示数组 `x` 的索引 0 和索引 1 的元素来简单地看到这一点：

```
p( x[0] )   #=> 4 (Ruby 1.8)    "\x04" (Ruby 1.9)
p( x[1] )   #=> 8 (Ruby 1.8)    "\b"   (Ruby 1.9)
```

Ruby 1.9 返回的字符串可以显示为十六进制值或转义字符。在这里，你可以看到，对于 Marshal 版本 4.8，第一个值是 \x04，这是 4 的十六进制表示，而第二个值是 \b，它是退格符的转义字符，恰好具有 ASCII 值 8。可以使用 `ord` 方法将字符串转换为整数。这是 Ruby 1.9 的版本：

```
print( "Marshal version: #{x[0].ord}:#{x[1].ord}\n" )
```

现在正确地显示了版本号：`4:8`。当然，如果你使用的是 Marshal 库的不同版本，显示的数字将不同。Marshal 库还声明了两个常量，`MAJOR_VERSION` 和 `MINOR_VERSION`，它们存储当前正在使用的 Marshal 库的版本号。所以，乍一看，似乎应该很容易比较保存数据的版本号和当前版本号。

只有一个问题：当你将数据保存到磁盘上的文件时，`dump` 方法接受一个 IO 或 File 对象，并返回一个 IO（或 File）对象，而不是一个字符串：

*version_error.rb*

```
f = File.open( 'friends.sav', 'w' )
x = Marshal.dump( ["fred", "bert", "mary"], f )
f.close        #=> x is now: #<File:friends.sav (closed)>
```

如果你现在尝试获取 `x[0]` 和 `x[1]` 的值，你会收到一个错误信息：

```
p( x[0] )
#=> Error: undefined method '[]' for #<File:friends.sav (closed)> (NoMethodError)
```

从文件中重新加载数据并没有什么指导意义：

```
File.open( 'friends.sav' ){ |f|
    x = Marshal.load(f)
}

puts( x[0] )
puts( x[1] )
```

这里的两个 `puts` 语句并没有（正如我天真地希望的那样）打印出被序列化数据的版本号的主版本和次版本号；实际上，它们打印出的是名称“fred”和“bert”——也就是说，从数据文件 *friends.sav* 中加载到数组 `x` 的前两个元素。

那么，究竟如何从保存的数据中获取版本号呢？我必须承认，我不得不阅读 *marshal.c* 中的 C 代码（这不是我最喜欢的活动！）并检查保存文件中的十六进制数据来找出答案。结果证明，正如文档所说，“你可以通过读取序列化数据的前两个字节来提取版本号。”然而，这并不是自动完成的。你必须显式地读取这些数据，如下所示：

*version_m2.rb*

```
f = File.open('test2.sav')
if (RUBY_VERSION.to_f > 1.8) then
    vMajor = f.getc().ord
    vMinor = f.getc().ord
else
    vMajor = f.getc()
    vMinor = f.getc()
end
f.close
```

在这里，`getc` 方法从输入流中读取下一个 8 位字节。请注意，我再次编写了一个测试，使其与 Ruby 1.8 兼容，在 Ruby 1.8 中 `getc` 返回一个数值字符值，以及与 Ruby 1.9 兼容，在 Ruby 1.9 中 `getc` 返回一个必须使用 `ord` 转换为整数的单字符字符串。

我的示例项目 *version_m2.rb* 展示了一种简单的方法，用于比较保存数据的版本号与当前 Marshal 库的版本号，以便在尝试重新加载数据之前确定数据格式是否可能兼容。

```
if vMajor == Marshal::MAJOR_VERSION then
   puts( "Major version number is compatible" )
   if vMinor == Marshal::MINOR_VERSION then
      puts( "Minor version number is compatible" )
   elsif vMinor < Marshal::MINOR_VERSION then
      puts( "Minor version is lower - old file format" )
   else
      puts( "Minor version is higher - newer file format" )
   end
else
   puts( "Major version number is incompatible" )
end
```
