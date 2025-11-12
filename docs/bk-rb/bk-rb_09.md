# 第九章。异常处理

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

即使是最精心编写的程序有时也会遇到未预见到的错误。例如，如果你编写了一个需要从磁盘读取一些数据的程序，它基于这样的假设：指定的磁盘实际上是可用的，数据是有效的。如果你的程序基于用户输入进行计算，它基于这样的假设：输入适合用于计算。

尽管你可能在问题出现之前尝试预测一些潜在的问题——例如，通过编写代码来检查在从文件中读取数据之前文件是否存在，或者在执行计算之前检查用户输入是否为数值型——但你永远无法提前预测到每一个可能的问题。

例如，用户可能在已经开始从数据盘中读取数据后移除该数据盘；或者某些晦涩的计算可能在你的代码尝试除以这个值之前得到 0。当你知道代码在运行时可能会因为一些未预见的情况而“出错”，你可以通过使用*异常处理*来尝试避免灾难。

*异常*是一个封装成对象的错误。该对象是 Exception 类（或其子类）的一个实例。你可以通过捕获 Exception 对象来处理异常，可选地使用它包含的信息（例如打印适当的错误消息）并采取任何必要的措施来从错误中恢复——可能通过关闭任何仍然打开的文件或将变量赋值为一个有意义的值，该变量可能由于错误的计算而被赋予了一些无意义的值。

# rescue：当发生错误时执行代码

异常处理的语法可以总结如下：

```
begin
   # Some code which may cause an exception
rescue <Exception Class>
   # Code to recover from the exception
end
```

当异常未被处理时，你的程序可能会崩溃，Ruby 可能会显示一个相对不友好的错误消息：

*div_by_zero.rb*

```
x = 1/0
puts( x )
```

程序以这个错误终止：

```
C:/bookofruby/ch9/div_by_zero.rb:3:in `/': divided by 0 (ZeroDivisionError)
 from C:/bookofruby/ch9/div_by_zero.rb:3:in `<main>'
```

为了防止这种情况发生，你应该自己处理异常。以下是一个处理尝试除以零的异常处理器的示例：

*exception1.rb*

```
begin
   x = 1/0
rescue Exception
   x = 0
   puts( $!.class )
   puts( $! )
end
puts( x )
```

当运行时，跟随`rescue Exception`的代码执行并显示以下内容：

```
ZeroDivisionError
divided by 0
0
```

在`begin`和`end`之间的代码是我的异常处理块。我将麻烦的代码放在了`begin`之后。当发生异常时，它会在以`rescue`开始的段落中处理。我首先做的事情是将变量`x`设置为一个有意义的值。接下来是这两个难以理解的语句：

```
puts( $!.class )
puts( $! )
```

在 Ruby 中，`$!`是一个全局变量，它被分配了最后一个异常。打印`$!.class`会显示类名，这里为 ZeroDivisionError；单独打印变量`$!`的效果是显示 Exception 对象包含的错误消息，这里为“除以 0。”

我通常不太喜欢依赖于全局变量，尤其是当它们有像`$!`这样不描述性的名字时。幸运的是，有一个替代方案。你可以在异常的类名之后和变量名之前放置“关联运算符”（`=>`）来将变量名与异常关联起来：

`exception2.rb`

```
rescue Exception => exc
```

现在，你可以使用变量名（这里为`exc`）来引用异常对象：

```
puts( exc.class )
puts( exc )
```

虽然当除以零时，你可能会得到 ZeroDivisionError 异常似乎很明显，但在实际代码中，有时异常的类型并不那么可预测。假设，例如，你有一个基于用户提供的两个值进行除法的方法：

```
def calc( val1, val2 )
    return val1 / val2
end
```

这可能会产生各种不同的异常。显然，如果用户输入的第二个值是 0，你将得到一个 ZeroDivisionError。

异常有一个家族树

要理解`rescue`子句如何捕获异常，只需记住异常是对象，就像所有其他对象一样，它们由一个类定义。还有一个清晰的“继承链”，从基类开始：Object（在 Ruby 1.8 中）或 BasicObject（Ruby 1.9 中）。运行`exception_tree.rb`来显示异常的祖先。这是 Ruby 1.9 显示的内容：

```
ZeroDivisionError
StandardError
Exception
Object
BasicObject
```

`exception_tree.rb`

然而，如果第二个值是一个字符串，异常将是一个 TypeError，而如果第一个值是一个字符串，它将是一个 NoMethodError（因为 String 类没有定义“除法运算符”，即`/`）。在这里，`rescue`块处理所有可能的异常：

`multi_except.rb`

```
def calc( val1, val2 )
    begin
        result = val1 / val2
    rescue Exception => e
        puts( e.class )
        puts( e )
        result = nil
    end
    return result
end
```

你可以通过故意生成不同的错误条件来测试这一点：

```
calc( 20, 0 )
      #=> ZeroDivisionError
      #=> divided by 0
calc( 20, "100" )
      #=> TypeError
      #=> String can't be coerced into Fixnum
calc( "100", 100 )
      #=> NoMethodError
      #=> undefined method `/' for "100":String
```

通常，对于不同的异常采取不同的行动会有所帮助。你可以通过添加多个`rescue`子句来实现这一点。每个`rescue`子句可以处理多种异常类型，异常类名用逗号分隔。在这里，我的`calc`方法在一个子句中处理 TypeError 和 NoMethodError 异常，并使用一个通用的 Exception 处理程序来处理其他类型的异常：

`multi_except2.rb`

```
def calc( val1, val2 )
    begin
        result = val1 / val2
    rescue TypeError, NoMethodError => e
        puts( e.class )
        puts( e )
        puts( "One of the values is not a number!" )
        result = nil
    rescue Exception => e
        puts( e.class )
        puts( e )
        result = nil
    end
    return result
end
```

这次，当处理 TypeError 或 NoMethodError（但没有其他类型的错误）时，我的附加错误信息将显示如下：

```
NoMethodError
undefined method `/' for "100":String
One of the values is not a number!
```

当处理多种异常类型时，你应该始终将处理特定异常的`rescue`子句放在前面，然后跟随处理更通用异常的`rescue`子句。

当处理特定的异常，例如 TypeError 时，`begin..end`异常块会退出，这样执行流程就不会“滴漏”到更通用的`rescue`子句中。然而，如果你首先放置一个通用的异常处理`rescue`子句，那么它将处理所有异常，所以任何更具体的子句都不会执行。

例如，如果我在`calc`方法中反转了`rescue`子句的顺序，将通用的异常处理程序放在前面，这将匹配所有异常类型，因此特定类型错误（TypeError）和 NoMethodError 异常的子句将永远不会被执行：

*multi_except_err.rb*

```
# This is incorrect...
rescue Exception => e
      puts( e.class )
      result = nil
   rescue TypeError, NoMethodError => e
      puts( e.class )
      puts( e )
      puts( "Oops! This message will never be displayed!" )
      result = nil
   end
calc( 20, 0 )        #=> ZeroDivisionError
calc( 20, "100" )    #=> TypeError
calc( "100", 100 )   #=> NoMethodError
```

# ensure: 无论是否发生错误都执行代码

在某些情况下，你可能希望在发生异常与否的情况下都执行某些特定的操作。例如，无论何时你处理某种不可预测的输入/输出——比如说，当你在磁盘上的文件和目录工作时——总有可能位置（磁盘或目录）或数据源（文件）根本不存在，或者可能提供其他类型的问题——例如，当你尝试写入时磁盘已满，或者当你尝试读取时文件包含错误类型的数据。

无论你是否遇到任何问题，你可能都需要执行一些最终的“清理”程序，比如登录特定的工作目录或关闭之前打开的文件。你可以通过在`begin..rescue`代码块后面跟着另一个以`ensure`关键字开始的代码块来实现。`ensure`块中的代码将始终执行，无论之前是否发生了异常。

让我们看看两个简单的例子。在第一个例子中，我尝试登录磁盘并显示目录列表。在这一点上，我想要确保我的工作目录（由`Dir.getwd`给出）总是恢复到其原始位置。我通过在`startdir`变量中保存原始目录，并在`ensure`块中再次将其设置为工作目录来实现这一点：

*ensure.rb*

```
startdir = Dir.getwd

begin
   Dir.chdir( "X:\\" )
   puts( `dir` )
rescue Exception => e
   puts e.class
   puts e
ensure
   Dir.chdir( startdir )
end
```

当我运行这个程序时，显示以下内容：

```
We start out here: C:/Huw/programming/bookofruby/ch9
Errno::ENOENT
No such file or directory - X:\
We end up here: C:/Huw/programming/bookofruby/ch9
```

现在我们来看看如何处理从文件中读取错误数据的问题。这可能发生在数据损坏、你意外打开了错误的文件，或者——非常简单地说——如果你的程序代码中存在错误。

这里有一个文件，*test.txt*，包含六行。前五行是数字；第六行是字符串，“six。”我的代码打开这个文件并读取所有六行：

*ensure2.rb*

```
f = File.new( "test.txt" )
begin
    for i in (1..6) do
        puts("line number: #{f.lineno}")
        line = f.gets.chomp
        num = line.to_i
        puts( "Line '#{line}' is converted to #{num}" )
        puts( 100 / num )
    end
rescue Exception => e
    puts( e.class )
    puts( e )
ensure
    f.close
    puts( "File closed" )
end
```

行被读取为字符串（使用`gets`），代码尝试将它们转换为整数（使用`to_i`）。当转换失败时不会产生错误；相反，Ruby 返回值 0。问题出现在下一行代码中，它尝试除以转换后的数字。

在一开始打开数据文件后，我想确保无论是否发生错误，文件都会被关闭。例如，如果我只通过编辑 `for` 循环中的范围来读取前五行，那么就不会有异常。但我仍然希望关闭文件。但是将文件关闭代码（`f.close`）放在 `rescue` 子句中是没有用的，因为在这种情况下，它不会执行。然而，通过将其放在 `ensure` 子句中，我可以确保无论是否发生异常，文件都会被关闭。

# else: 当没有错误发生时执行代码

如果 `rescue` 部分在发生错误时执行，而 `ensure` 部分无论是否发生错误都会执行，那么如何具体执行仅在错误 *不* 发生时的一些代码？

要做到这一点，可以在 `rescue` 部分之后和 `ensure` 部分之前（如果有）添加一个可选的 `else` 子句，如下所示：

```
begin
        # code which may cause an exception
rescue [Exception Type]
else    # optional section executes if no exception occurs
ensure  # optional exception always executes
end
```

这是一个例子：

*else.rb*

```
def doCalc( aNum )
   begin
      result = 100 / aNum.to_i
   rescue Exception => e     # executes when there is an error
      result = 0
      msg = "Error: " + e.to_s
   else                      # executes when there is no error
      msg = "Result = #{result}"
   ensure                    # always executes
      msg = "You entered '#{aNum}'. " + msg
   end
   return msg
end
```

尝试运行前面的程序并输入一个不会引起错误的数字，例如 10，这样 `msg` 将在 `else` 子句中赋值；然后尝试输入 0，这将引起错误，因此 `msg` 将在 `rescue` 子句中赋值。无论是否有错误，`ensure` 部分都将执行以创建一个以“您输入了”开头，后跟任何其他信息的 `msg` 字符串。例如：

```
You entered '5'. Result = 20
You entered '0'. Error: divided by 0
```

# 错误号

如果你之前运行了 *ensure.rb* 程序并且密切关注，当你尝试登录一个不存在的驱动器（例如，在我的系统中可能是 *X:\* 驱动器）时，你可能已经注意到了一些异常情况。通常，当发生异常时，异常类是一个特定命名的类型的实例，例如 ZeroDivisionError 或 NoMethodError。然而，在这种情况下，异常的类显示为 `Errno::ENOENT`。

结果表明，在 Ruby 中存在相当多的 `Errno` 错误。尝试 *disk_err.rb*。这个文件定义了一个方法，`chDisk`，它尝试通过字符 `aChar` 识别的磁盘登录。所以如果你将“A”作为参数传递给 `chDisk`，它将尝试登录到 *A:\* 驱动器。我已经调用了 `chDisk` 方法三次，每次传递不同的字符串：

*disk_err.rb*

```
def chDisk( aChar )
    startdir = Dir.getwd
    begin
        Dir.chdir( "#{aChar}:\\" )
        puts( `dir` )
    rescue Exception => e
        #showFamily( e.class ) # to see ancestors, uncomment
        puts e.class           # ...and comment out this
        puts e
    ensure
        Dir.chdir( startdir )
    end
end

chDisk( "F" )
chDisk( "X" )
chDisk( "ABC" )
```

当然，你可能需要编辑你电脑上的路径以适应不同的环境。在我的电脑上，*F:\* 是我的 DVD 驱动器。目前它是空的，当我的程序尝试登录到它时，Ruby 返回这种类型的异常：`Errno::EACCES`。

我电脑上没有 *X:\* 驱动器，当我尝试登录到那个驱动器时，Ruby 返回这种类型的异常：`Errno::ENOENT`。

在前面的例子中，我传递了字符串参数“ABC”，它作为磁盘标识符是无效的，Ruby 返回这种类型的异常：`Errno::EINVAL`。

这种类型的错误是 SystemCallError 类的子类。你可以通过取消注释 *disk_err.rb* 源代码中指示的行来轻松验证这一点。这会调用相同的 `showFamily` 方法，你之前在 *exception_tree.rb* 程序中使用过。

这些错误类实际上封装了底层操作系统返回的整数错误值。常量的名称和值可能根据操作系统和 Ruby 的版本而有所不同。在这里，`Errno` 是包含常量的模块的名称，例如 `EACCES` 和 `ENOENT`，它们与整数错误值相匹配。

要查看 `Errno` 常量的完整列表，请运行以下命令：

*errno.rb*

```
puts( Errno.constants )
```

要查看任何给定常量的对应数值，请将 `::Errno` 追加到常量名称之后，例如：

```
Errno::EINVAL::Errno
```

你可以使用以下代码显示所有 `Errno` 常量的列表及其数值（在这里，`eval` 方法评估传递给它的表达式——你将在 第二十章 中了解它是如何工作的）：

```
for err in Errno.constants do
   errnum = eval( "Errno::#{err}::Errno" )
   puts( "#{err}, #{errnum}" )
end
```

# retry: 在错误后再次尝试执行代码

如果你认为错误条件可能是瞬时的或者可能被纠正（可能是用户？），你可以在 `begin..end` 块中重新运行所有代码，使用关键字 `retry`，如下例所示，如果发生像 ZeroDivisionError 这样的错误，会提示用户重新输入值：

*retry.rb*

```
def doCalc
    begin
        print( "Enter a number: " )
        aNum = gets().chomp()
        result = 100 / aNum.to_i
    rescue Exception => e
        result = 0
        puts( "Error: " + e.to_s + "\nPlease try again." )
        retry           # *`retry on exception`*
    else
        msg = "Result = #{result}"
    ensure
        msg = "You entered '#{aNum}'. " + msg
    end
    return msg
end
```

### 注意

当你想将异常对象（如 `e`）的消息附加到字符串（如 `"Error: "`）时，Ruby 1.9 强制你显式地将 `e` 转换为字符串（`"Error: " + e.to_s`），而 Ruby 1.8 会为你完成转换（`"Error: " + e`）。

当然，错误可能并不像你想象的那么短暂，所以如果你使用 `retry`，你可能想提供一个明确定义的退出条件，以确保代码在固定次数尝试后停止执行。

你可以在 `begin` 子句中增加一个局部变量。（如果你这样做，确保它在任何可能生成异常的代码之前增加，因为一旦发生异常，`rescue` 子句之前的代码将跳过！）然后在 `rescue` 部分测试该变量的值，如下所示：

```
rescue Exception => e
    if aValue < someValue then
        retry
    end
```

这里有一个完整的示例，我测试了一个名为 `tries` 的变量的值，以确保在异常处理块退出之前，代码在没有错误的情况下不超过三次尝试运行：

*retry2.rb*

```
def doCalc
    tries = 0
    begin
        print( "Enter a number: " )
        tries += 1
        aNum = gets().chomp()
        result = 100 / aNum.to_i
    rescue Exception => e
        msg = "Error: " + e.to_s
        puts( msg )
        puts( "tries = #{tries}" )
        result = 0
        if tries < 3 then # set a fixed number of retries
           retry
        end
    else
        msg = "Result = #{result}"
    ensure
        msg = "You entered '#{aNum}'. " + msg
    end
    return msg
end
```

如果用户连续三次输入 0，这将产生以下输出：

```
Enter a number: 0
Error: divided by 0
tries = 1
Enter a number: 0
Error: divided by 0
tries = 2
Enter a number: 0
Error: divided by 0
tries = 3
You entered '0'. Error: divided by 0
```

# raise: 重新激活已处理错误

有时候你可能想在异常被捕获后仍然保持异常“活跃”，即使它已经被异常处理块捕获。例如，你可以通过将其传递给其他方法来延迟处理异常，比如传递给名为 `handleError` 的方法。你可以使用 `raise` 方法来做这件事。然而，你需要意识到，一旦抛出，异常需要被重新处理；否则，它可能会导致你的程序崩溃。以下是一个抛出 `ZeroDivisionError` 异常并将异常传递给名为 `handleError` 的方法的简单示例：

*raise.rb*

```
begin
    divbyzero
rescue Exception => e
    puts( "A problem just occurred. Please wait..." )
    x = 0
    begin
        raise
    rescue
        handleError( e )
    end
end
```

在这里，`divbyzero` 是一个方法的名字，其中包含了除以零的操作，而 `handleError` 是一个打印异常更详细信息的方法：

```
def handleError( e )
    puts( "Error of type: #{e.class}" )
    puts( e )
    puts( "Here is a backtrace: " )
    puts( e.backtrace )
end
```

注意，这使用了 `backtrace` 方法，它显示了一个字符串数组，显示了错误发生的文件名和行号，以及在这种情况下调用产生错误 `divbyzero` 方法的行。这是该程序输出的一个示例：

```
A problem just occurred. Please wait...
Error of type: ZeroDivisionError
divided by 0
Here is a backtrace:
C:/Huw/programming/bookofruby/ch9/raise.rb:11:in `/'
C:/Huw/programming/bookofruby/ch9/raise.rb:11:in `divbyzero'
C:/Huw/programming/bookofruby/ch9/raise.rb:15:in `<main>'
```

你也可以专门抛出异常来强制错误条件，即使程序代码本身没有引发异常。单独调用 `raise` 会引发一个 `RuntimeError` 类型的异常（或者全局变量 `$!` 中存储的任何异常）：

```
raise         # raises RuntimeError
```

默认情况下，这不会与任何描述性消息相关联。你可以添加一个消息作为参数，如下所示：

```
raise "An unknown exception just occurred!"
```

你可以抛出一个特定的错误类型：

```
raise ZeroDivisionError
```

你还可以创建一个特定异常类型的对象，并用自定义消息初始化它：

```
raise ZeroDivisionError.new( "I'm afraid you divided by Zero" )
```

这是一个简单的例子：

*raise2.rb*

```
begin
    raise ZeroDivisionError.new( "I'm afraid you divided by Zero" )
rescue Exception => e
    puts( e.class )
    puts( "message: " + e.to_s )
end
```

这将输出以下内容：

```
ZeroDivisionError
message: I'm afraid you divided by Zero
```

如果标准异常类型不符合你的需求，你当然可以通过继承现有异常来创建新的异常。为你的类提供一个 `to_str` 方法，以便为它们提供一个默认消息。

*raise3.rb*

```
class NoNameError < Exception
    def to_str
        "No Name given!"
    end
end
```

这是一个关于如何抛出自定义异常的示例：

```
def sayHello( aName )
    begin
        if (aName == "") or (aName == nil) then
             raise NoNameError
        end
    rescue Exception => e
        puts( e.class )
        puts( "error message: " + e.to_s )
        puts( e.backtrace )
    else
        puts( "Hello #{aName}" )
    end
end
```

如果你现在输入 `sayHello( nil )`，这将产生以下输出：

```
NoNameError
error message: NoNameError
C:/Huw/programming/bookofruby/ch9/raise3.rb:12:in `sayHello'
C:/Huw/programming/bookofruby/ch9/raise3.rb:23:in `<main>'
```

深入挖掘

在捕获异常时，在某些情况下，可以省略 `begin` 关键字。在这里，你将了解这种语法。我还会澄清一些关于 `catch` 和 `throw` 的潜在混淆。

省略 begin 和 end

在方法、类或模块内部捕获异常时，你可以选择性地省略 `begin` 和 `end`。例如，以下所有这些都是合法的：

*omit_begin_end.rb*

```
def calc
        result = 1/0
    rescue Exception => e
        puts( e.class )
        puts( e )
        result = nil
    return result
end

class X
        @@x = 1/0
    rescue Exception => e
        puts( e.class )
        puts( e )
end

module Y
        @@x = 1/0
    rescue Exception => e
        puts( e.class )
        puts( e )
end
```

在所有之前的案例中，如果以通常的方式在异常处理代码的开始和结束处放置 `begin` 和 `end` 关键字，异常处理也将正常工作。

catch..throw

在某些语言中，异常是通过使用关键字`catch`来捕获的，并且可以使用关键字`throw`来引发。尽管 Ruby 提供了`catch`和`throw`方法，但这些方法与其异常处理没有直接关系。相反，`catch`和`throw`用于在满足某些条件时跳出定义的代码块。当然，您可以使用`catch`和`throw`在发生异常时跳出代码块（尽管这可能不是处理错误的最优雅方式）。例如，如果发生`ZeroDivisionError`，此代码将退出由大括号分隔的代码块：

*catch_except.rb*

```
catch( :finished) {
   print( 'Enter a number: ' )
   num = gets().chomp.to_i
   begin
      result = 100 / num
   rescue Exception => e
      throw :finished        # jump to end of block
   end
   puts( "The result of that calculation is #{result}" )
}     # end of :finished catch block
```

详见第六章了解`catch`和`throw`的更多内容。
