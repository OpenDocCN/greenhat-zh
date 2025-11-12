# 第十四章。YAML

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

在某个时候，大多数桌面应用程序都希望将结构化数据保存到磁盘并从中读取。你已经看到了如何使用简单的 IO 例程，如 `gets` 和 `puts` 来读取和写入数据。但如果你要保存和恢复来自混合对象类型列表的数据，你会怎么做？使用 Ruby，一种简单的方法是使用 YAML。

### 注意

YAML 是一个首字母缩略词，有人认为它代表“另一种标记语言”（Yet Another Markup Language），或者（递归地）代表“YAML 不是标记语言”（YAML Ain’t Markup Language）。

# 转换为 YAML

YAML 定义了一种序列化（数据保存）格式，以人类可读的文本形式存储信息。YAML 可以与各种编程语言一起使用；为了在 Ruby 中使用它，你的代码需要访问 Ruby 的 *yaml.rb* 文件中的例程。通常，这会在代码单元的顶部通过以下方式加载或“要求”文件：

```
require "yaml"
```

完成这一步后，你将能够访问各种将 Ruby 对象转换为 YAML 格式的方法，以便将它们的数据写入文件。随后，你将能够读取这些保存的数据，并使用它来重建 Ruby 对象。要将对象转换为 YAML 格式，你可以使用 `to_yaml` 方法。这将转换标准对象类型，如字符串、整数、数组、哈希等。例如，这是转换字符串的方法：

*to_yaml1.rb*

```
"hello world".to_yaml
```

这就是转换数组的方法：

```
["a1", "a2" ].to_yaml
```

这是数组转换的结果 YAML 格式，其中每个键值对都放置在新的一行上：

```
---
- a1
- a2
```

注意定义 YAML “文档”开始的三条横线和定义列表中每个新元素的单条横线。在 YAML 术语中，文档不是磁盘上的单独文件，而是单独的 YAML 定义；一个磁盘文件可能包含多个 YAML 文档。有关 YAML 格式的更多信息，请参阅 深入挖掘 中的 深入挖掘。

你还可以将非标准类型的对象转换为 YAML。例如，假设你创建了这个类和对象：

*to_yaml2.rb*

```
class MyClass
    def initialize( anInt, aString )
        @myint = anInt
        @mystring =aString
    end
end

ob1 = MyClass.new( 100, "hello world" ).to_yaml
```

该对象的 YAML 表示形式将先于文本 `!ruby/object:`，然后是类名，变量名后跟冒号（但去掉 `@`），以及它们的值，每行一个：

```
--- !ruby/object:MyClass
myint: 100
mystring: hello world
```

如果你想要打印出对象的 YAML 表示形式，你可以使用方法 `y()`，这是 YAML 的一个类似于熟悉 `p()` 方法的等效方法，用于检查和打印正常的 Ruby 对象：

*yaml_test1.rb*

```
y( ['Bert', 'Fred', 'Mary'] )
```

这将显示以下内容：

```
---
- Bert
- Fred
- Mary
```

你可以类似地显示一个哈希：

```
y({ 'fruit' => 'banana', :vegetable => 'cabbage', 'number' => 3 })
```

在这种情况下，每个键值对都放置在新的一行上：

```
---
fruit: banana
:vegetable: cabbage
number: 3
```

### 注意

哈希元素的顺序可能因你使用的 Ruby 版本而异（见第四章）。在处理哈希时，最好假设没有内在的顺序。

或者，你可以显示你自己的“自定义”对象：

```
t = Treasure.new( 'magic lamp', 500 )
y( t )
```

这显示了数据格式，就像在之前的示例中我使用`to_yaml`时一样，类名在顶部，变量名和值成对地出现在连续的行上。这是包含实例变量`@name`和`@value`的 Treasure 对象的 YAML 表示：

```
--- !ruby/object:Treasure
name: magic lamp
value: 500
```

你甚至可以使用`y()`来显示嵌套数组等相当复杂的对象：

*yaml_test2.rb*

```
arr1 =    [    ["The Groovesters", "Groovy Tunes", 12 ],
               [  "Dolly Parton", "Greatest Hits", 38 ]
          ]
y( arr1 )
```

这是`arr1`的 YAML 表示：

```
---
- - The Groovesters
  - Groovy Tunes
  - 12
- - Dolly Parton
  - Greatest Hits
  - 38
```

这里是另一个包含用户定义类型对象的数组的示例：

```
class CD
    def initialize( anArtist, aName, theNumTracks )
        @artist = anArtist
        @name   = aName
        @numtracks    = theNumTracks
    end
end

arr2 = [CD.new("The Beasts", "Beastly Tunes", 22),
       CD.new("The Strolling Bones","Songs For Senior Citizens",38)
]

y( arr2 )
```

这将输出以下 YAML：

```
---
- !ruby/object:CD
  artist: The Beasts
  name: Beastly Tunes
  numtracks: 22
- !ruby/object:CD
  artist: The Strolling Bones
  name: Songs For Senior Citizens
  numtracks: 38
```

# 嵌套序列

当相关的数据序列（如数组）嵌套在其他数据序列内部时，这种关系通过缩进来表示。例如，假设你在 Ruby 中声明了这个数组：

*nested_arrays.rb*

```
arr = [1,[2,3,[4,5,6,[7,8,9,10],"end3"],"end2"],"end1"]
```

当以 YAML 格式呈现（例如，通过`y(arr)`）时，它将如下所示：

```
---
- 1
- - 2
  - 3
  - - 4
    - 5
    - 6
    - - 7
      - 8
      - 9
      - 10
    - end3
  - end2
- end1
```

# 保存 YAML 数据

将你的 Ruby 对象转换为 YAML 格式的另一种方便的方法是由`dump`方法提供的。在最简单的情况下，它将你的 Ruby 数据转换为 YAML 格式并将其“写入”字符串：

*yaml_dump1.rb*

```
arr = ["fred", "bert", "mary"]
yaml_arr = YAML.dump( arr )
    # yaml_arr is now: "--- \n- fred\n- bert\n- mary\n"
```

更有用的是，`dump`方法可以接受第二个参数，这是一种 IO 对象，通常是文件。你可以打开一个文件并将数据写入它：

*yaml_dump2.rb*

```
f = File.open( 'friends.yml', 'w' )
YAML.dump( ["fred", "bert", "mary"], f )
f.close
```

或者你可以打开文件（或某种其他类型的 IO 对象）并将其传递给一个关联的块：

```
File.open( 'morefriends.yml', 'w' ){ |friendsfile|
    YAML.dump( ["sally", "agnes", "john" ], friendsfile )
}
```

在每种情况下，数组的 YAML 表示都将作为纯文本保存到指定的文件中。例如，当之前的代码执行时，它将以下文本写入*morefriends.yml*文件：

*morefriends.yml*

```
---
- sally
- agnes
- john
```

如果你使用一个块，文件将在退出块时自动关闭；否则，你应该显式地使用`close`方法关闭文件。你也可以以类似的方式使用块来打开文件并读取 YAML 数据：

```
File.open( 'morefriends.yml' ){ |f|
    $arr= YAML.load(f)
}
```

假设*morefriends.yml*包含之前保存的数据，一旦它被加载并分配给之前显示的块中的全局变量`$arr`，`$arr`将包含这个字符串数组：

```
["sally", "agnes", "john"]
```

# 保存时省略变量

如果出于某种原因，你希望在序列化对象时省略一些实例变量，你可以通过定义一个名为`to_yaml_properties`的方法来实现。在这个方法的主体中，放置一个字符串数组。每个字符串都应该匹配要保存的实例变量的名称。任何未指定的变量将不会被保存。看看这个例子：

*limit_y.rb*

```
class Yclass
    def initialize(aNum, aStr, anArray)
        @num = aNum
        @str = aStr
        @arr = anArray
    end

    def to_yaml_properties
        ["@num", "@arr"]     #<= @str will not be saved!
    end
end
```

在这里，`to_yaml_properties`限制了在调用`YAML.dump`时将保存的变量，即`@num`和`@arr`。字符串变量`@str`将不会被保存。如果你想根据保存的 YAML 数据重建对象，你有责任确保任何“缺失”的变量要么不是必需的（在这种情况下可以忽略），要么如果它们是必需的，它们应该被初始化为一些有意义的值。

```
ob = Yclass.new( 100, "fred", [1,2,3] )
    # ...creates object with @num=100, @str="fred", @arr=[1,2,3]

yaml_ob = YAML.dump( ob )
    #...dumps to YAML only the @num and @arr data (omits @str)

ob2 = YAML.load( yaml_ob )
    #...creates ob2 from dumped data with @num=100, @arr=[1,2,3]
    # but without @str
```

# 多文档，一个文件

之前我提到，三个短横线用于标记一个名为*document*的新 YAML 部分的开始。例如，假设你想要将两个数组`arr1`和`arr2`保存到一个文件中，*multidoc.yml*。在这里，`arr1`是一个包含两个嵌套数组的数组，而`arr2`是一个包含两个 CD 对象的数组：

*multi_docs.rb*

```
arr1 =  [   ["The Groovesters", "Groovy Tunes", 12 ],
            [  "Dolly Parton", "Greatest Hits", 38 ]
        ]

arr2 =  [ CD.new("Gribbit Mcluskey", "Fab Songs", 22),
          CD.new("Wayne Snodgrass", "Singalong-a-Snodgrass", 24)
        ]
```

这是我的将数组转换为 YAML 并写入文件的常规操作（如第十三章 Files and IO 中所述），`'w'`参数导致文件以写入模式打开）：

```
File.open( 'multidoc.yml', 'w' ){ |f|
    YAML.dump( arr1, f )
    YAML.dump( arr2, f )
}
```

如果你现在查看*multidoc.yml*文件，你会看到数据已经被保存为两个独立的“文档”，每个文档都以三个短横线开始：

```
---
- - The Groovesters
  - Groovy Tunes
  - 12
- - Dolly Parton
  - Greatest Hits
  - 38
---
- !ruby/object:CD
  artist: Gribbit Mcluskey
  name: Fab Songs
  numtracks: 22
- !ruby/object:CD
  artist: Wayne Snodgrass
  name: Singalong-a-Snodgrass
  numtracks: 24
```

现在，我需要找到一种方法通过读取两个文档来重建这些数组。这就是`load_documents`方法发挥作用的地方。`load_documents`方法调用一个块，并将每个连续的文档传递给它。以下是如何使用此方法从两个 YAML 文档中重建两个数组（放置在另一个数组`$new_arr`中）的示例：

```
File.open( 'multidoc.yml' ) {|f|
    YAML.load_documents( f ) { |doc|
      $new_arr << doc
    }
 }
```

你可以通过执行以下操作来验证`$new_arr`是否已初始化为两个数组：

```
p( $new_arr )
```

这显示了包含加载数据的两个嵌套数组的数组：

```
[[["The Groovesters", "Groovy Tunes", 12], ["Dolly Parton",
 "Greatest Hits", 38]], [#<CD:0x2c30e98 @artist="Gribbit Mcluskey", @name="Fab
 Songs", @numtracks=22>, #<CD:0x2c30ad8 @artist="Wayne Snodgrass",
 @name="Singalong-a-Snodgrass", @numtracks=24>]]
```

因为这有点难以管理，你可能更喜欢使用外数组的索引来单独显示每个嵌套数组：

```
p( $new_arr[0] )
p( $new_arr[1] )
```

之前的假设是你事先知道可用的嵌套数组的数量。作为替代，这里有一个更通用的方法来做同样的事情，使用`each`方法将所有可用的项目传递到一个块中；这适用于任何数量的数组：

```
$new_arr.each{ |arr| p( arr ) }
```

# YAML 数据库

为了查看一个稍微复杂一点的示例，该示例以 YAML 格式保存和加载数据，请查看*cd_db.rb*示例程序。它实现了一个简单的 CD 数据库。它定义了三种类型的 CD 对象：一个包含名称、艺术家和曲目数量的基本`CD`；以及两个更专业的派生对象，`PopCD`，它添加了关于流派（例如，摇滚或乡村）的数据，以及`ClassicalCD`，它添加了关于指挥家和作曲家的数据：

*cd_db.rb*

```
class CD
    def initialize( arr )
        @name       = arr[0]
        @artist     = arr[1]
        @numtracks  = arr[2]
    end

    def getdetails
        return[@name, @artist, @numtracks]
    end
end

class PopCD < CD

    def initialize( arr )
        super( arr  )
        @genre = arr[3]
    end

    def getdetails
        return( super << @genre )
    end
end

class ClassicalCD < CD
    def initialize( arr )
        super( arr )
        @conductor  = arr[3]
        @composer   = arr[4]
    end

    def getdetails
        return( super << @conductor << @composer )
    end
end
```

当程序运行时，用户可以输入数据以创建任何这三种类型的新的 CD 对象。还有一个选项可以将数据保存到磁盘上。当应用程序随后再次运行时，现有的数据将被重新加载。

数据本身在代码中组织得非常简单（甚至可以说是简单的），每个对象的数据在创建对象之前被读入一个数组。整个 CD 对象数据库被保存到全局变量`$cd_arr`中，并使用 YAML 方法写入磁盘并重新加载到内存中：

```
def saveDB
    File.open( $fn, 'w' ) {
        |f|
        f.write($cd_arr.to_yaml)
    }
end

def loadDB
    input_data = File.read( $fn )
    $cd_arr = YAML::load( input_data )
end
```

请记住，这个程序是为了简单而不是美观而编写的。在现实世界的应用中，你肯定希望创建一些更优雅的数据结构来管理你的 Dolly Parton 收藏！

# YAML 冒险之旅

作为使用 YAML 的最后一个示例，我提供了一个冒险游戏的基本框架（*gamesave_y.rb*）。这创建了一些宝藏对象和一些房间对象。宝藏对象被“放入”房间对象中（即，它们被放置在房间包含的数组中），然后房间对象被放入地图对象中。这相当于构建了一个中等复杂的数据结构，其中一个类型的对象（地图）包含任意数量的另一个类型的对象（房间），每个房间可能包含零个或多个其他类型的对象（宝藏）。

初看之下，找到一种方法将整个混合对象类型的网络存储到磁盘上并在以后阶段重建这个网络可能看起来像是一个编程噩梦。实际上，多亏了 Ruby YAML 库提供的序列化功能，保存和恢复这些数据几乎可以轻松完成。这是因为序列化可以免除你逐个保存每个对象的麻烦。相反，你只需要“转储”顶层对象；在这里，即地图对象，`mymap`。

完成此操作后，顶层对象“包含”的任何对象（如房间）或包含的对象本身所包含的任何对象（如宝藏）都将自动为你保存。然后，只需通过一次性加载所有保存的数据并将其分配给“顶层”对象（在这里是地图）即可重新构建它们：

*gamesave_y.rb*

```
# Save mymap
File.open( 'game.yml', 'w' ){ |f|
    YAML.dump( mymap, f )
}

# Reload mymap
File.open( 'game.yml' ){ |f|
    mymap = YAML.load(f)
}
```

这个程序的完整代码太长，无法在此展示，所以我建议你尝试源代码存档中提供的程序，以欣赏使用 YAML 保存和加载相当复杂的数据结构是多么简单。

深入挖掘

本节总结了 YAML 数据文件的结构，并解释了如何在 YAML 格式中保存嵌套哈希。

YAML 指南简述

如我之前提到的，YAML 以称为 *documents* 的文本块的形式存储信息，这些文档包含数据的 *sequences*。每个文档以三个连字符（`---`）开始，列表中的每个单独元素都以单个连字符（`-`）字符开始。例如，以下是一个包含一个文档和两个列表项的 YAML 数据文件：

```
---
- artist: The Groovesters
  name: Groovy Tunes
  numtracks: 12
- artist: Dolly Parton
  name: Greatest Hits
  numtracks: 38
```

在前面的示例中，你可以看到每个列表项由两部分组成：一个名称，如 `artist:`（在列表项中相同），以及其右侧的数据，如 `Dolly Parton`，这可能因列表项而异。这些项类似于 Ruby 哈希中的键值对。YAML 将键值列表称为 *maps*。

以下是一个包含两个项目的 YAML 文档，每个项目包含三个项目；换句话说，它是包含两个三项目“嵌套”数组的数组的 YAML 表示：

```
---
- - The Groovesters
  - Groovy Tunes
  - 12
- - Dolly Parton
  - Greatest Hits
  - 38
```

现在我们来看看 YAML 如何处理嵌套哈希。考虑这个哈希：

*hash_to_yaml.rb*

```
hsh = { :friend1 => 'mary',
        :friend2 => 'sally',
        :friend3 => 'gary',
        :morefriends => { :chap_i_met_in_a_bar => 'simon',
                          :girl_next_door => 'wanda'
                         }
}
```

正如您已经看到的，在 YAML 中，哈希自然地表示为键值对列表。然而，在前面展示的示例中，键 `:morefriends` 与其值关联的是一个嵌套的哈希。YAML 是如何表示这个嵌套哈希的呢？结果是，与数组（参见 嵌套序列）一样，它只是简单地缩进嵌套的哈希：

```
:friend1: mary
:friend2: sally
:friend3: gary
:morefriends:
     :chap_i_met_in_a_bar: simon
     :girl_next_door: wanda
```

### 注意

如需深入了解 YAML，请参阅 [`www.yaml.org/`](http://www.yaml.org/)。

Ruby 伴随的 YAML 库相当庞大且复杂，其中包含的方法比本章描述的要多得多。然而，您现在应该对 YAML 有足够的了解，可以在自己的程序中有效地使用它。您可以在空闲时间探索 YAML 库的外围。不过，结果却是 YAML 并非在 Ruby 中序列化数据的唯一方式。您将在下一章中看到另一种方法。
