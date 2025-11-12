# 第八章 数据格式

![无标题图片](img/httpatomoreillycomsourcenostarchimages671943.png.jpg)

与地图一起工作通常意味着与大量地理数据交互。除了描述性信息外，你还需要知道一个地方的位置，以便你可以在地图上绘制它。很可能大多数时候你都需要从其他人那里获取数据。你也可能需要与他人共享你的数据。

为了使地理数据的传递更加方便，已经采用了几种标准格式。在本章中，我将介绍几种分享地图上基本地理元素（点、线和形状）的方法。

我们还将介绍一些在网络上用于交换信息的流行格式。越来越多地，你的数据源将是提供 API 以使其数据可用的网站。在大多数情况下，你需要的格式在本章中都有涵盖。

让我们开始学习一些数据格式。

# #52: 使用 XML

可扩展标记语言（XML）是本章讨论的许多 Web 数据和格式的基础。它看起来很像 HTML，因为一些 HTML 实际上就是 XML。然而，本节将避免具体 XML 变体，因为我会单独在其他章节中介绍它们。在这里，我将专注于如何识别和使用通用 XML。

首先，它看起来是什么样子？XML 由标签组成，这些标签是尖括号`<`和`>`内的单词。理解括号内的单词通常很容易，但有时它们是缩写或首字母缩略词。标签可以包含其他标签，以及*`key`*`=`*`value`*对，这些对被称为*属性*。包含其他标签或文本的标签以匹配的结束标签结束，该结束标签在标签名之前包含一个`/`。

考虑这个简短的 XML 文件示例：

```
❶ ?xml version="1.0" encoding="UTF-8"?>
❷ root>
    <child ❸name="first">
      ❹<grandchild>text could go here</grandchild>
    </child>
    <child name="second" ❺/>
  </root>
```

XML 文件通常以类似的方式开始，有一个处理指令声明其为 XML❶并提供版本和编码信息。这个特殊标签没有对应的结束标签。移除这个头部后，我们就可以直接进入数据部分。

根元素❷可以命名为任何名称，但只能有一个。例如，HTML 只有一个`<html>`标签。在起始和结束根标签之间是真正的 XML 内容。在这种情况下，XML 有两个子元素。再次强调，标签可以命名为任何名称，但在这个例子中，我使用了有助于描述 XML 术语的名称。

XML 是分层的，访问数据需要理解其结构。第一个子元素有一个单一属性❸以及它自己的子元素❹。第二个子元素也有一个属性，但它不包含任何子元素。在这种情况下，我们可以缩写结束标签❺。

现在我们想要获取 XML 中的这些数据。读取标签并将它们转换为计算机可以理解的结构称为 *解析*。大多数语言都有一些内置的解析 XML 的方法。接下来，我将展示两个 JavaScript 示例和一个使用 PHP 的示例，PHP 是一种服务器端编程语言。

## 使用 JavaScript 解析 XML

每个现代浏览器都提供了一种读取 XML 内容的方法，这在很大程度上是因为网络的大部分内容都是基于这项技术构建的。不幸的是，各种浏览器之间存在差异。此外，获取深层嵌套的元素可能会很麻烦。

在展示更简单的方法之前，我们将使用前面描述的 XML 示例在这个部分尝试一下。我们不会加载 XML 文件（我将在下一节中介绍），而是使用存储为文本字符串的 XML。

由于我们使用 JavaScript，文件需要位于网页内部。我们将在 JavaScript 部分进行所有操作，因此网页将保持空白。将这些行添加到新文件中：

```
<html>
  <head>
  <script>
❶   var xmltxt = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n";
        xmltxt += "<root>\n";
        xmltxt += " <child name=\"first\" />\n";
        xmltxt += " <child name=\"second\">\n";
        xmltxt += " <grandchild>text goes here</grandchild>\n";
        xmltxt += " </child>\n";
        xmltxt += "</root>\n";
    var x = null;
    var output = "";

❷   if (window.ActiveXObject) { // Internet Explorer
      x = new ActiveXObject("Microsoft.XMLDOM");
      x.async = false;
      x.loadXML(xmltxt);
    }
❸   else if (window.DOMParser) { // Other browsers
      var p = new DOMParser();
      x = p.parseFromString(xmltxt, "text/xml");
    }
    else {
        // Can't load XML
    }

    if (x) {
❹     `var children = x.getElementsByTagName("child");`
❺     `for (i=0; i<children.length; i++) {`
❻       `output += children[i].getAttribute("name") + "\n";`
      `}`
    }
    alert(output);
  </script>
  </head>
  <body></body>
  </html>
```

粗体行是获取数据的那一行。其余的都是纯设置。公平地说，准备 XML 文本 ❶ 确实需要七行代码。我们可以将其缩减为单行，但我将其展开以增强可读性。

解析 XML 需要一个多步骤的方法。首先，我们需要尝试 Internet Explorer 的方法 ❷。如果我们使用的是其他浏览器，这种方法将失败。然后，我们将使用更广泛采用的方法 ❸。希望这种方法能奏效，因为如果它不起作用，我们就无法解析 XML。

在这两次尝试之后，`x` 变量应该包含一个 XML 对象。这个变量被粗体部分用来提取子元素的名称。首先，我们遍历 XML 中的所有子标签 ❹。然后，我们循环遍历所有这些标签 ❺。我们可以通过 `i` 变量来判断我们处于哪个步骤，它从零开始，每次循环增加一步。每次循环中，我们将当前子元素的名称添加到我们将要输出的文本中 ❻。

如果你将此文件加载到网页浏览器中，你应该会看到一个包含 "first" 和 "second" 名称的 JavaScript 提示框。你已经成功使用纯 JavaScript 解析了 XML。现在让我们看看如何使用 JavaScript 库 jQuery 来解析它。

## 使用 jQuery JavaScript 库解析 XML

jQuery 的单一原则是编写更少的代码。艰苦的工作留给了库，而库本身非常小（目前小于 20K）。当涉及到获取和解析 XML 时，jQuery 保持简单且可预测。

与纯 JavaScript 示例类似，我们将在一个空白的 HTML 文件中解析 XML。然而，在这种情况下，我们将直接从文件中加载我们的 XML，这是一个常见的情况。确保你有一个名为 *example.xml* 的文件，其中包含之前提到的 XML，然后将这些行添加到同一目录下的新文件中：

```
<html>
  <head>
❶   <script src="http://ajax.googleapis.com/ajax/libs/
jquery/1.3/jquery.min.js"></script>
  <script>
    var output = "";
❷   $.get("example.xml", {}, ❸function(xml) {
❹     $("child", xml).❺each(function(i) {
        output += ❻this.getAttribute("name") + "\n";
    });
    alert(output);
  });
  </script>
  </head>
  <body></body>
  </html>
```

你首先会注意到我们需要加载 jQuery JavaScript 文件 ❶。你可以从 [`jquery.com/`](http://jquery.com/) 下载到自己的服务器上，或者像我在这个例子中那样引用由 Google 托管的版本。在任一情况下，我们都可以访问库的许多功能，包括使用 JavaScript 通过 Ajax 加载文件以及解析 XML。

jQuery 库利用了许多技术来减少你需要编写的 JavaScript 量。其中之一是 *美元符号对象*，它允许你使用非常简单的语法访问 jQuery 的许多功能。例如，加载 XML 文件是通过调用 `$.get` 函数 ❷ 来实现的。

要在没有 jQuery 的情况下实现 Ajax 调用，需要根据浏览器尝试不同的方法，就像我们在上一节中的 XML 解析示例一样。相反，jQuery 做了工作，确保我们可以获取数据。

加载 XML 还展示了另一种减少代码的技术：内联的、*匿名函数*。这些函数是标准 JavaScript 语言的一部分，但与 jQuery 简化代码的方式结合使用时变得特别有用。当执行 Ajax 调用，例如我们用来加载 XML 文件的调用时，JavaScript 需要一个回调函数。而不是创建一个命名函数仅用于接收 XML 结果，我们可以直接编写一个内联函数 ❸。

在匿名函数（之所以这样命名是因为它没有名字）内部，我们使用另一个 jQuery 简写来解析 XML。解析发生得如此之快，你可能甚至都没有意识到它在发生。美元符号函数传递了我们想要的标签名称，以及包含我们从 Ajax 调用中获取的内容的 XML 变量 ❹。然后我们将 jQuery 的 `each` 函数链接到结果，我们可以遍历所有子元素 ❺。我们不需要使用显式的 `for` 循环，也不需要确定子元素的数量。这都在 jQuery 内部完成。

我们每次通过 jQuery 循环所做的事情由另一个匿名函数决定。同样，我们只是将一切保持内联，因为为了一行代码而创建一个命名函数是没有意义的。当然，代码只有一行，因为我们使用了 jQuery。`this` 变量持有当前子元素，然后我们使用在非 jQuery 示例中使用的相同的 `getAttribute` 函数来获取名称属性 ❻。

在大约是前一个例子一半的行数中，我们达到了相同的结果。如果你在网页浏览器中加载该文件，JavaScript 警告将打印出子标签的名称，“first” 和 “second”。jQuery 使得在处理 API 和解析数据格式时，你经常要做的事情变得容易，其中许多使用 XML。

## 使用 PHP 解析 XML

在许多情况下，你可能会想在服务器上检索 XML。为此，你不会使用 JavaScript，因为你通常在 Web 浏览器中编写 JavaScript；你会使用 PHP。PHP 是一种流行的编程语言，用于编写服务器端应用程序。有关 PHP 以及确保你的服务器上有 PHP 的更多信息，请务必阅读第九章。

让我们使用 PHP 解析上一节中的示例 XML。确保你的服务器上有一个名为*example.xml*的文件。在同一目录下创建一个新的 PHP 文件，并添加以下行：

```
<?
❶ $xmltxt = join("\n", file("example.xml"));
❷ $xmlobj = simplexml_load_string($xmltxt);
  foreach (❸$xmlobj->child as $childobj) {
    print($childobj->❹attributes()->name . "\n");
  }
  ?>
```

我们首先将示例 XML 文件加载到`$xmltxt`变量中❶。在许多情况下，我们实际上是从 API 加载 XML。无论如何，XML 内容最终会存储在一个变量中，以便进行解析。

我们将解析 XML 的工作交给 PHP 的`SimpleXML`类，这个类在 PHP 5 中是自动包含的。`simplexml_load_string`函数将文本 XML 转换为有用的对象❷，以便访问 XML 内部的数据。还有一个`simplexml_load_file`函数，但通常情况下，你会将从一个 API 获取的字符串转换为字符串。

一旦 XML 以对象形式存在，我们就可以在子元素中查找名称属性。我们需要遍历所有子元素❸，将当前子元素放入它自己的对象中。然后，我们获取属性❹并找到名为 name 的属性。

如果你看到 PHP 创建的对象的样子，查询 XML 的代码将更有意义。使用`print_r($xmlobj)`来查看层次对象的文本表示：

```
SimpleXMLElement Object (
❶   [child] => Array (
      [0] => SimpleXMLElement Object (
❷       [@attributes] => Array (
          [name] => first
        )
      )
      [1] => SimpleXMLElement Object
        [@attributes] => Array (
          [name] => second
        )
❸       [grandchild] => text goes here
      )
    )
  )
```

首先，所有内容都在一个`SimpleXMLElement`对象内部，就像所有 XML 都在根标签内一样。还包括额外的`SimpleXMLElement`对象，这类似于标签内有标签。SimpleXML 类本质上将 XML 转换为一系列数组。

首先，创建一个包含所有子元素的数值数组❶。在这种情况下，只包含两个子元素，编号为 0 和 1，因为，就像 JavaScript 一样，PHP 中的数组索引从零开始。每个子元素都有一个属性数组❷，它是关联数组，意味着它包含键值对。键是属性名，在这种情况下是`name`。

最后，如果标签内部存在标签，它们会被列出。在这种情况下，第二个子标签包含一个孙子标签❸。这个标签只包含文本，所以它也被表示为一个键值对。如果它包含下层的标签或属性，我们就会有另一个`SimpleXMLElement`。再次强调，`SimpleXML`类主要是为了在 PHP 对象中找到表示 XML 的方法。

### 更简单的 XML 与 XPath

在基本情况下，如果 XML 文件较短且不包含深层嵌套的标签，遍历`SimpleXML`对象是可行的。如果你被大量的 XML 内容淹没，你可能会发现使用 XPath 查询更简单。

与 XML 一样，XPath 是一个网络标准。您可以使用 XPath 遍历 XML 到您想要的数据。您需要做的就是调用 `SimpleXML` 对象上的 `xpath` 函数，并告诉它要访问的“路径”。

以下三个示例都找到了相同的元素，即孙子标签，它嵌套在两层层次结构中。

您可以使用元素的完整路径：

```
$xmlobj->xpath("/root/child/grandchild")
```

或者，在前面加上一个双斜杠以获取所有孙子标签，无论它们周围有什么标签：

```
$xmlobj->xpath("//grandchild")
```

或者混合匹配。在这里，我们获取任何位于子标签下的孙子标签：

```
$xmlobj->xpath("//child/grandchild")
```

XPath 可以帮助您以更多方式快速访问 XML，例如查询特定值，但这里不会涵盖它们。您可以在 [`php.net/simplexml`](http://php.net/simplexml) 上了解更多关于 XPath 和 `SimpleXML` 的信息。

我已经向您展示了几个访问 XML 的方法：JavaScript、jQuery 库和 PHP。您使用什么取决于您从哪里获取 XML，XML 的复杂程度以及您已经使用的语言。

或者，您可能会厌倦解析 XML 并将其移植到您的 JavaScript 地图。许多程序员更喜欢直接使用一种称为 JSON 的格式，这种格式更接近真正的 JavaScript。继续阅读以了解该格式，并查看在 #57: 将 XML 转换为 JSON 中的 #57: 将 XML 转换为 JSON，了解如何将 XML 转换为更易于使用的 JSON。

# #53: 使用 JSON

随着 JavaScript 在网络上的日益流行，JSON 正迅速成为开发者的首选数据格式。这是因为 JSON 代表 *JavaScript Object Notation*，在 JavaScript 中使用它几乎不需要解析。此外，JSON 比表示相同数据所需的 XML 字符少，因为它没有闭合标签。

您不受任何一种语言的限制。您可以在许多服务器端编程语言中解析 JSON。我将在稍后使用 PHP 举一个例子。大多数现代语言都有一种数据结构，使得将 JSON 转换变得容易。这，加上 JavaScript 的流行，使得这种格式在数据交换中得到了广泛的应用。

关于 JSON 的有用性就说到这里：让我们看看 JSON 的一个例子。以下显示了上一个项目中 XML 的表示方式：

```
❶ {"child": [
❷   {"attributes": {"name": "first"}},
❸   {"attributes": {"name": "second"}, "grandchild": "text goes here"}
  ]}
```

这个基本示例比它需要的复杂一些，但它展示了数据在 JSON 中可以组织的多种方式。构建块是一系列键值对，位于大括号内，在 JavaScript 中称为对象。乐趣在于值的定义。

在这个例子中，我们的主要对象 ❶ 只有一个键，`child`。值是一个数组，由括号声明。数组本身可以包含一系列值。在这种情况下，值是更多的对象。

数组中的第一个对象 ❷ 包含一个单独的键，`attributes`，以及它内部的另一个对象。最终，现在有三个级别的新的对象包含一个键 `name` 和值为 `first`。数组中的第二个对象 ❸ 有一个类似的第一键值对，然后是一个第二键，`grandchild`，它有一个文本值。

因此，一个值可以是一个数组、另一个对象或纯文本。它也可以是一个数字、布尔值或 null，尽管我没有在这个例子中展示这一点。

你是否被“值是什么”的循环定义弄糊涂了？这种复杂性是有意为之的，但实际上它实际上是一种表达多种类型数据简单的方法。因为一个对象可以包含数组、对象，甚至是其他对象的数组，所以许多类型分层的数据可以用 JSON 在非常小的空间内表达。

现在你已经了解了 JSON 的样子，让我们开始使用它。

## 使用 JavaScript 和 jQuery 解析 JSON

你还记得 JSON 代表什么吗？JavaScript 对象表示法。这种数据格式不仅是为了 JavaScript 而创建的，也是从它那里发展而来的。

如果你将 JSON 直接硬编码到 JavaScript 中，你不需要做任何事情来使用它内部的数据。它已经准备好了。在这里，我们使用 JavaScript 访问示例 JSON 中的第一个子元素：

```
`var obj =` {"child": [
  {"attributes": {"name": "first"}},
  {"attributes": {"name": "second"}, "grandchild": "text goes here"}
]}`;`
`alert(obj.child[0].attributes.name);`
```

我添加了粗体的代码部分。否则，这段代码就是之前提到的确切 JSON。我所做的就是将它赋给一个变量（`obj`），用分号（`;`）结束声明，然后从对象中警报一个特定的值。

当然，JSON 很可能不会直接写入 JavaScript。相反，你可能会从 API 接收它。换句话说，你可能会以文本形式拥有 JSON。

如果你信任数据，你可以使用 JavaScript 的 `eval` 函数将 JSON 从文本转换为对象。然而，确保你有良好的数据是一个明智的想法，因为 `eval` 将执行任何 JavaScript 文本，而不仅仅是 JSON 格式的文本。

为了避免可能的大型安全问题，一些浏览器中添加了 `parseJSON` 函数。但这个函数只有在它在每个浏览器中都有效时才真正有用。你可以在等待每个浏览器都支持最新的 JavaScript 版本的同时，使用 [`json.org/`](http://json.org/) 上可用的 JavaScript 文件来填补空白。

另一个选项是使用 jQuery JavaScript 库，它有一个简单的方法通过 Ajax 获取数据。实际上，你可以在 jQuery 中单行检索和解析 JSON。

将这些行添加到一个新的 HTML 文件中：

```
<html>
  <head>
❶ <script src="http://ajax.googleapis.com/ajax/libs/
jquery/1.3/jquery.min.js"></script>
  <script>
  $.getJSON(❷"example.json", ❸function(jobj) {
      alert(❹jobj.child[0].attributes.name);
    });
  </script>
  </head>
  <body>
  </body>
  </html>
```

要访问许多有用的 jQuery 函数，我们需要包含 jQuery JavaScript 文件 ❶。虽然你可以从 [`jquery.com/`](http://jquery.com/) 下载此文件到你的服务器，但你也可以引用一个由 Google 托管的版本，就像我在这里所做的那样。

jQuery 为你做了很多困难的工作，使得编写执行高级功能的非常短的 JavaScript 成为可能。它减少代码的一种更明显的方式是引入美元符号对象。jQuery 中发生的大部分事情都通过`$`进行。

例如，我们使用 jQuery 的`$.getJSON`函数来创建一个 Ajax 调用，下载并解析一个 JSON 文件。我们需要提供的重要信息是 JSON URL ❷。这个 URL 可以是一个本地文件，或者是对外部 API 的调用。

接下来，jQuery 需要一个函数引用。在这种情况下，我们使用一个内联的匿名函数 ❸ 来描述我们想要对 JSON 结果做什么。同样，jQuery 关于减少代码，但理解这里发生的事情仍然很重要。Ajax 获取我们的 JSON，然后将其解析为对象。该对象返回给匿名函数，我们可以在其中做任何我们想做的事情。在这种情况下，我创建了一个带有第一个子元素名称的警告 ❹，就像当数据是硬编码时一样。

### 注意

如果你调用返回 JSON 的外部 API，出于安全原因，该 API 需要接受一个回调函数名称。要查看这个功能的示例，请查看我在第六十九部分：创建天气地图中如何从 Yahoo! Pipes 检索 JSON 的#69: 创建天气地图。

在 jQuery 的基础上构建可以节省你的时间，并允许你专注于映射项目的高级问题。你也会得到一个额外的复杂性层，因为你需要在 HTML 中包含更多的 JavaScript。希望它的好处可以弥补这个加载时间上的小成本。

## 使用 PHP 解析 JSON

有时候你只需要服务器上的数据。如果数据是 JSON 格式，你将无法使用 JavaScript，因为它几乎总是写在网页浏览器中。尽管如此，大多数语言都可以轻松读取 JSON，所以你会发现它是一个在服务器上以及客户端上使用都合理的格式。

我将再次使用 PHP 作为示例服务器端编程语言，因为它在大多数网络主机上都是可用的。如果你是 PHP 的新手，我在第九章中提供了 PHP 用于地理项目的介绍。

在这个项目的开始阶段，我说过大多数语言都有类似 JSON 的数据结构。JavaScript 对象，通过其键值对，在 PHP 中表现为关联数组。同样，PHP 也有标准数组，除了文本和数字字符串。换句话说，所有这些部分都是为了完全表示 JSON。

下面是一些示例 PHP 代码，它声明了我在示例 JSON 文件中使用的数据：

```
<?
$obj = array("child" => array(
  array("attributes" => array("name" => "first")),
  array(
  "attributes" => array("name" => "second"),
  "grandchild" => "text goes here")
));
?>
```

既然我们知道在 PHP 中表示 JSON 是可能的，我们如何从文本解析到关联数组呢？从 PHP 5 开始，你可以通过单个调用解析 JSON。

这里有一个例子，通过硬编码的 JSON 文本来访问第一个子元素的名字。你的可能来自 API，或者可能是一个文件：

```
<?
$jtxt = "{\"child\":[" .
        "{\"attributes\":{\"name\":\"first\"}}," .
        "{\"attributes\":{\"name\":\"second\"}," .
        "\"grandchild\":\"text goes here\"}]}";
$jobj = ❶json_decode($jtxt);
print ($jobj->❷child[0]->attributes->name);
?>
```

是的，所有的工作都交由内部 PHP 函数 ❶ 来执行。与之前我们使用的关联数组不同，`json_decode` 使用 PHP 对象。这个对象略有不同，但表达数据的方式相似。

如 `child` ❷ 这样的键是对象的实例变量，并且通过 `->` 箭头引用。所有其他类型的数据，包括常规数组，都按原样传递。就像所有其他例子一样，第一个子元素的名字可以在三个级别以下找到。

好奇的读者可能会想知道是否还有另一个函数可以从 PHP 数据结构中创建 JSON 文本。当然！`json_decode` 的反义词是 `json_encode`。你可以传递第一个示例中的 `$obj` 变量或第二个示例中的 `$jobj` 变量，结果将与存储在 `$jtxt` 变量中的 JSON 文本相同。

你可能需要比编码更频繁地解码 JSON。话虽如此，当你需要它时，你会很高兴这个函数存在。关于编码 JSON 的例子，请查看 #71: 通过位置搜索音乐活动。

尽管我最近的例子使用了 PHP，但 JSON 是数据格式中的新星，因为它很容易与 JavaScript 集成——JSON 实质上 *就是* JavaScript。现在你知道如何安全地读取 JSON 数据，你可能会寻找使用该格式的 API。JSON 使得从数据解析中解脱出来变得容易，这样你就可以做你真正想做的事情：创建充满数据的精彩网络地图。

# #54: 使用 GeoRSS

位置只是网络中传递的众多信息中的一小部分。如果你包括它们的意义的上下文，一个点的列表就更有用了。GeoRSS 是一种将位置和其他地理信息添加到内容源的方法，从而创建地理标记的内容。

内容本身通常是博客文章或照片，尽管它可以是一切。博客是地理标记的理想候选者，因为大多数博客已经通过 RSS 源进行聚合，这是一种获取最新帖子而不必访问网站的方法。

虽然 GeoRSS 是以 RSS 命名的，但它可以用于除了 RSS 之外的其他格式。例如，美国地质调查局发布了一个最近的地震 Atom 源，包括每个地震的位置和深度。GeoRSS 可以添加到任何 XML 源，以将地理数据附加到其他内容。

让我们看看 RSS 源中 GeoRSS 的一个例子：

```
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" >
  <channel>
    <link>http://mapscripting.com</link>
    <title>Feed Title</title>
    <description>Feed Description</description>
    <item>
      <pubDate>Thu, 01 Jan 2010 00:01:23 +0000</pubDate>
      <title>Item Title</title>
      <description>Item Description</description>
      <author>Item Author</author>
      **`<georss:point>45.256 −71.92</georss:point>`**
    </item>
    ...
  </channel>
</rss>
```

``大部分文本是标准的 RSS。粗体部分是 GeoRSS 插件，它们为源添加位置数据。在顶部，你需要包含 GeoRSS 命名空间，这允许你使用 `georss:` 前缀为标签。``

`在这个例子中，我们声明了一个点，这是一个地理坐标。在标签内，我们首先放置纬度，然后是一个空格，接着是经度。有时你可能会看到数字之间有一个逗号。两者都是允许的。`

`GeoRSS 也有几种声明形状的方式。这些形状由多个点组成，通常代表路线、边界或其他边界。GeoRSS 将它们称为线条、多边形和矩形。`

`线条和多边形都声明为纬度和经度点的序列：`

```
<georss:line>45.256 −110.45 46.46 −109.48 43.84 −109.86</georss:line>
<georss:polygon>
  45.256 −110.45 46.46 −109.48 43.84 −109.86 45.256 −110.45
</georss:polygon>
```

`正如这个例子所示，一条线至少有两个坐标，但可以有更多。这样，一条线可以描述一条路线。`

`多边形也是以类似的方式声明的，但最后一个点必须与第一个点相同。换句话说，多边形是一个环形路线。例如，它可以用来描述房屋的外墙或国家的边界。`

`另一方面，一个矩形将始终创建一个矩形形状，并且只需要两个坐标来声明：`

```
<georss:box>42.943 −71.032 43.039 −69.856</georss:box>
```

``如果你感到困惑，那没关系。矩形有四个角，所以箱子不应该是四个坐标吗？这就像 Mapstraction 的 `BoundingBox`，在 #19: Draw a Rectangle to Declare an Area 中有所介绍。GeoRSS 只使用两个角来确定箱子的位置。你需要的最小数据是西南角和东北角。从这两个点，你可以外推西北角和东南角。` ``

`现在你对 GeoRSS 有了一些了解，让我们在另一种类型的源中使用它。以下是在 Atom 格式中 GeoRSS 的一个示例：`

```
<?xml version="1.0" encoding="utf-8"?>
<feed 
      ``>   <title>Feed Title</title>   <updated>2010-01-01T00:01:23Z</updated>   <author>     <name>Feed Author</name>     <email>feedemail@example.com</email>   </author>   <id>tag:mapscripting.com,2009-01-01:feedid</id>   <entry>     <title>Entry title</title>     <link href="http://example.org/entry_link"/>     <updated>2010-01-01T00:01:23Z</updated>     <summary>Entry summary</summary>     **`<georss:point>45.256 −71.92</georss:point>`**   </entry>   ... </feed>``
```

`` `Atom 是 RSS 的替代品，并且是一个广泛支持的格式。正如你所见，对我们来说，它与 RSS 非常相似。同样，GeoRSS 的部分被加粗。` ``

`` `你通常会在 RSS 和 Atom 格式中看到 GeoRSS。然而，我展示的却是 GeoRSS 的简单版本。格式有时看起来略有不同，但它仍然是 GeoRSS。继续阅读以了解一些替代 GeoRSS 编码的示例。` ``

## `` `使用替代 GeoRSS 编码` ``

`` `在上一节中展示的 GeoRSS 对于大多数需求来说是足够的，并且可能是你遇到的最常见的编码方式。然而，了解其不足之处并认识到其他表示位置数据的方式是很重要的。` ``

`` `*GML* 是 *地理标记语言*，并且是 GeoRSS 的超集。GML 是为了表达任何形式的地理信息而创建的，包括拓扑和除我们一直在使用的纬度/经度系统（称为 WGS84）之外的其他坐标系统。` ``

`` `为了让你的 GeoRSS 与 GML 兼容，你需要额外的标签。例如，声明一个点所需的单个标签变成了三个标签：` ``

```
`<georss:where>   <gml:Point>     <gml:pos>45.256 −71.92</gml:pos>   </gml:Point> </georss:where>`
```

``` ``The data communicated with these tags is the same as the simple GeoRSS example. The additional tags are not extraneous, but included because GML allows for more specific uses. For example, the `<gml:Point>` tag is where you would declare another coordinate system.`` ```

``` ``GML equivalents of the geometric objects used in the simple GeoRSS are available. The method for polygons, lines, and boxes is similar to points. The GML code is wrapped in `<georss:where>` tags. You can find out more about all the options at [`www.georss.org/gml`](http://www.georss.org/gml).`` ```

`` `无论何时你编写包含冒号命名的标签的 XML，你都需要确保冒号前的单词（命名空间）在 XML 的顶部被声明。因为这个例子使用了 GeoRSS 和 GML，我们需要包含这两个命名空间：` ``

``` ``Add this code inside the root tag of your XML. For RSS, the root is `<rss>`, and for Atom, it is `<feed>`. Both are necessary because the GML version of GeoRSS uses `georss:` and `gml:` tags.`` ```

`` `到目前为止展示的两种 GeoRSS 格式是你在新数据源中最可能遇到的编码方法。尽管旧版本仍在一定程度上被广泛使用。` ``

`` `基本地理词汇表是由万维网联盟（W3C）开发的编码，该组织负责监督 HTML 和 CSS 等其他标准的发展。GeoRSS 的开发使得 W3C 的地理标签变得过时，但你经常会遇到它们，因此需要能够识别它们。` ``

```
`<geo:lat>55.701</geo:lat> <geo:long>12.552</geo:long>`
```

``` ``The biggest difference between the W3C geo-tags and the ones shown earlier is that the latitude and longitude is declared separately. Because this encoding is popular, this method is yet another allowed to write GeoRSS. You'll need a different namespace to be able to use the `geo:` tags, however:`` ```

```
`` `Now that you know about the many encodings of GeoRSS, let's see how to use GeoRSS directly with your map.` ``
```

## `` `在地图上显示 GeoRSS` ``

`` `Mapstraction 使得将 GeoRSS 添加到地图变得简单。通过一个单一的功能，你可以将 GeoRSS 层叠到地图上，而无需自己解析 XML。` ``

`` `要显示 GeoRSS，你只需要一个公开可访问的源和一张可以显示它的地图。将这些行放入一个新的 HTML 文件中，以查看 GeoRSS 的实际应用：` ``

```
``<html >     <head>       <title>Example GeoRSS Map</title>       <script src="*`http://maps.google.com/maps/api/js?`* *`sensor=false`*" type="text/javascript"></script>       <script src="mxn.js?(googlev3)"></script>       <style>       div#mymap {         width: 550px;         height: 450px;       }       </style>       <script type="text/javascript">       var mapstraction;       function create_map() {         mapstraction = new Mapstraction('mymap', 'google');         **`mapstraction.addOverlay(`** ❶         **`"http://mapscripting.com/example-georss.xml", ❷true);`**       }       </script>     </head>     <body onload="create_map()">       <div id="mymap"></div>     </body>   </html>``
```

``` ``Most of this is standard map code; the important lines are in bold. You can see we use Mapstraction's `addOverlay` function. The first argument is the GeoRSS URL ❶. This address must be available on the public Web, not on your local computer or a password-protected development server. The reason the feed has to be accessible is the underlying mapping provider will make an Ajax call to load the feed. The mapping provider can't make the call if it can't access the URL. If you don't have your own feed, you can use my example from the companion website.`` ```

``` ``The second argument ❷ is optional. This argument is a boolean, meaning the value is either `true` or `false`. It controls whether the map is auto-centered and zoomed in to show only the GeoRSS content.`` ```

`` `将之前的 HTML 文件加载到你的浏览器中，你应该能在地图上看到 GeoRSS 内容。如果你使用的是我的示例，你将看到跨越波特兰几座桥梁的路线，以及标识地标的位置标记。` ``

`` `现在你对 GeoRSS 格式及其三种编码方式有了更多的了解。在本节中，我展示了它们如何在 RSS 和 Atom 这两种最流行的网络订阅格式中使用。此外，你还学会了如何在 Mapstraction 代码的一行中，将你的 GeoRSS 数据叠加到地图上。要查看 GeoRSS 的一个示例，请阅读 #70: Display Recent Earthquakes Worldwide。` ``

````` ````# #55: Use KML    Google Earth, a three-dimensional geographic browser, popularized KML as a language to share geo-data. The acronym *KML* stands for *Keyhole Markup Language*, named after the company (acquired by Google) who invented it. Nevertheless, KML is an open standard based on XML. KML stores single locations, lists of points, and polygon shapes, among other features. The biggest factor that separates KML from other geographic data formats is that KML can also include styling information, so you can stipulate the color of lines or use custom marker icons.    KML has a special schema, meaning elements are declared in a specific way.    Here's a very basic KML file, containing one location, called a *Placemark*:    ``` ❶ <?xml version="1.0" encoding="UTF-8"?> ❷ <kml >   <Document> ❸   <Placemark> ❹     <name>Eiffel Tower</name> ❺     <description>The most recognizable place in Paris</description>       <Point> ❻       <coordinates>2.29293460923931,48.85819570061303,0</coordinates>       </Point>     </Placemark>   </Document>   </kml> ```    As you put your KML files together, you can view them in Google Earth or on the Google Maps website, as long as the KML is accessible on the web. Try viewing this example at [`maps.google.com/?q=http%3A//mapscripting.com/example.kml`](http://maps.google.com/?q=http%3A//mapscripting.com/example.kml).    Now let's examine what's inside that example KML file. As with every XML file, a KML file starts with the XML declaration ❶. Then the file points to the KML namespace ❷ to clearly specify we're speaking a particular XML language. With those technical bits out of the way, we can dive into the actual KML content.    The geographic data in a KML file all falls within the `<Document>` tag. Inside that, I add a `Placemark` ❸, which will contain location and other data for a single place. Each Placemark has a name ❹, which is essentially the equivalent of a title in GeoRSS. Similarly, each Placemark also has a description ❺, which can be plain text (as shown) or HTML (with `<` and `>` brackets written as `&lt;` and `&gt;`).    Arguably the most important piece of data for a Placemark is the actual geographic point. To declare this, we use a `<Point>` tag and then include the coordinates ❻ within it. Note we include three numbers, as opposed to the usual two. The last number represents altitude and is actually optional. We could increase the number, for example, if we wanted our Placemark to declare the top of the Eiffel Tower.    One final, important note about the first two coordinates: Unlike most other geographic data formats, *KML lists longitude before latitude*. This setup is easy to recognize in examples like this and in North America, where longitudes are always negative. But you definitely want to make sure you get these numbers in the right order.    A document generally contains multiple Placemarks, but in this simple example, I only use one. Adding another is easy—just include an additional pair of `<Placemark>` tags.    Now that you've seen a simple location, let's look at some other ways KML marks up geographic data.    ## Lines in KML    The single point is the basic feature of geographic data—and that holds for KML, as well. Sometimes a point isn't the best way to describe a place. What single point represents a country or a city? A single point can't; you need many points. This is where lines and polygons become useful.    Rather than use a different tag, KML declares both lines and polygons as Placemarks. Unlike the ones shown previously, these Placemarks do not contain a `<Point>` tag because we're actually declaring multiple points at once to represent a single place.    Consider this example, which shows the path of the Golden Gate Bridge in San Francisco:    ``` <Placemark>     <name>Golden Gate Bridge</name>     <description>A San Francisco landmark, for sure.</description> ❶   <LineString> ❷     <coordinates> ❸       −122.479485,37.827675,0         −122.477562,37.811028,0       </coordinates>     </LineString>   </Placemark> ```    I've omitted the tags that declare this a KML document and instead focused on the Placemark. I include a name and description, just as in the single point example. To describe the bridge, we first add some KML to say this Placemark is a line ❶, and then we insert a series of coordinates ❷ to describe the line.    Of course, a bridge has two ends, so the line is a very simple one using two points. If this were a more advanced line, such as describing a trail or the route of a race, we would just continue adding coordinates in an order such that they could later be connected by line segments.    As in the single point example, we declare three coordinates ❸, listed in longitude, latitude, and altitude order. The altitude is optional, but this coordinate may provide interesting data in some cases, showing the gradient of a trail, for example. This data is hard to show on a two-dimensional web map, but remember KML data is used in other ways, such as in Google Earth.    ## Polygons in KML    You may recall that in GeoRSS, as well as Mapstraction, a polygon is simply a line that ends at the same point where it begins. The same is true in KML, though the definition can also get a little more advanced with its description of a shape.    Let's stick with something simple here and continue with the world landmark theme. Consider this bit of KML that describes the outer edges of the Parthenon in Greece:    ``` <Placemark>     <name>Parthenon</name>     <description>A symbol of ancient Greece.</description> ❶   <Polygon> ❷     <outerBoundaryIs> ❸       <LinearRing>         <coordinates>           23.726295,37.971539,0           23.726376,37.971287,0           23.727116,37.971420,0           23.727024,37.971672,0           23.726295,37.971539,0         </coordinates>       </LinearRing>     </outerBoundaryIs>   </Polygon> </Placemark> ```    Notice that inside the polygon declaration ❶ I've include several levels of tags before getting to the familiar coordinates list. The reason for those additional levels is that KML's polygons are much more powerful than this example can communicate.    For example, here I've declared an outer boundary ❷ for the Parthenon, meaning I'm describing the outside walls. If I also use an inner boundary, I can create a rectangular donut to show just the columns of the Parthenon.    Inside a boundary, I use a linear ring ❸ to tell KML that I am creating a line that ends at its starting point. Here KML's polygon begins to look similar to GeoRSS, but with different syntax.    As with the simple line example, the coordinates are a list of longitude, latitude, and altitude (optional) values. We have four corners of the Parthenon to connect, which requires five points. Why not four? The first and the last must be identical—to complete the ring.    Now that you understand basic Placemarks, including lines and polygons, let's see where KML diverges from other geographic data formats: let's get stylish.    ## Style KML    Describing points, lines, and shapes is a basic building block of communicating geography. You can put these three types of data together and get plenty of information about a place. KML also lets you describe how you want the data to look, which separates it from other formats. Read on to learn how to style your KML.    If you know how HTML and CSS interact, KML styles will seem familiar. As with HTML, you can create styles inline or reference them globally with declarations at the top of your KML file.    A style tag is available for each of the three types of geographic data we've seen so far: `<IconStyle>`, `<LineStyle>`, and `<PolyStyle>`. Within the tag, you can declare an icon graphic, color, line width, and opacity. Here's an example of styles added inside the Eiffel Tower Placemark:    ``` <Style>   <IconStyle>     <Icon>       <href>http://mapscripting.com/icons/modernmonument.png</href>     </Icon>   </IconStyle> </Style> ```    This code is the KML equivalent of #5: Create a Custom Icon Marker in #4: Show and Hide Message Boxes Without Clicking the Marker. However, if you have a document with many points, you could end up with a lot of redundancy if most points share the same marker graphic, which makes declaring styles at the top of the code useful.    You can move the entire `<Style>` block up, as an immediate child of the `<Document>` tag. Then, if you give the tag an `id` attribute, you can reference it below. For example, here is the Parthenon example filled in with white:    ``` <?xml version="1.0" encoding="UTF-8"?>   <kml >   <Document>     `<Style id="stoneBuilding">`       `<LineStyle>` ❶       `<color>cccccc</color>`       `</LineStyle>`       `<PolyStyle>`         `<color>ffffff</color>`         `<fill>1</fill>` ❷       `<outline>1</outline>`       `</PolyStyle>`     `</Style>`     <Placemark>       <name>Parthenon</name>       <description>A symbol of ancient Greece.</description> ❸     `<styleUrl>#stoneBuilding</styleUrl>`       <Polygon>         <outerBoundaryIs>           <LinearRing>             <coordinates>               23.726295,37.971539,0               23.726376,37.971287,0               23.727116,37.971420,0               23.727024,37.971672,0               23.726295,37.971539,0             </coordinates>           </LinearRing>         </outerBoundaryIs>       </Polygon>     </Placemark>   </Document>   </kml> ```    The portion of the KML that produces the style is in bold. As you can see, most of the styling is already declared by the time we get to the Placemark. Along with the white-shaded polygon, I add a `LineStyle` to give it a light gray outline ❶. Then, I also make sure the `PolyStyle` has outlines turned on ❷ via the boolean (`1` is on; `0` is off).    Finally, we reference the styles ❸ from the Placemark itself. This reference is created much like a reference to an `id` within CSS, by prepending a `#` in front of the style `id`.    ## Display KML on a Map    With your KML file on the Web, you can display it on a map in many ways. Earlier, I showed how you can use the Google Maps website to show KML. You can also open it in Google Earth. In this section, we'll see instead how to layer a KML file into your embedded map using Mapstraction.    Start with a brand-new HTML file and add the following code:    ``` <html >     <head>       <title>Example KML Map</title>       <script src="*`http://maps.google.com/maps/api/js?sensor=false`*"  type="text/javascript"></script>       <script src="mxn.js?(googlev3)"></script>       <style>       div#mymap {         width: 550px;         height: 450px;       }       </style>       <script type="text/javascript">       var mapstraction;       function create_map() {         mapstraction = new Mapstraction('mymap', 'google');         `mapstraction.addOverlay(` ❶         `"http://mapscripting.com/example.kml",` ❷`true`);       }       </script>     </head>     <body onload="create_map()">       <div id="mymap"></div>     </body>   </html> ```    Most of this is a pretty basic map. The part that loads the KML file is in bold. Just like for the GeoRSS, we use Mapstraction's `addOverlay` function. The first argument is the KML URL ❶. As I've mentioned, this address needs to be available on the public Web, not on your local computer or a password-protected development server. The reason the feed has to be accessible is the underlying mapping provider will make an Ajax call to load the feed. This call won't work if the provider can't access the URL. If you don't have your own feed, you can use my example from the companion website.    The second argument ❷, which is optional, is a boolean, meaning the value is either `true` or `false`. This controls whether the map is auto-centered and zoomed in to show only the KML content.    Load this HTML into your browser and you should see the KML content on your map. If you're using my example, you'll see a polyline surrounding the Parthenon.    # #56: Use GPX    Are you a hiker, a runner, or a mountain biker? You could use the GPS exchange format, GPX, to track your favorite trails and routes. Most modern GPS devices, which use satellites to triangulate their location, can store this data and output it in GPX format. Even if the data did not originate on a GPS device, this format is useful for sharing any sequence of latitude and longitude points.    GPX really is just another way to store *polylines*, a series of coordinates to connect on a map. Where the format is different is that it also contains useful metadata to make more sense of the dozens of points.    GPX is separated into three types of data:     | **Tracks** A record of a particular trip, including the time at each step | | **Routes** A suggested trip meant to be shared with others, which does not include time information | | **Waypoints** A single point, often used for landmarks or other points of interest |    From a technical standpoint, GPX is just XML. Its schema is special, however, and elements are declared in a specific way. Here is an example GPX file:    ``` ❶ <?xml version="1.0" encoding="UTF-8"?> ❷ <gpx version="1.0" xmlns:xsi="http://www.w3.org/2001 /XMLSchema-instance" xmlns="http://   www.topografix.com/GPX/1/0" xsi:schemaLocation="http://www.to pografix.com/GPX/1/0 http://www   .topografix.com/GPX/1/0/gpx.xsd">      <trk>       <name>Dog Walk</name> ❸     <trkseg> ❹       <trkpt lat="45.521270" lon="-122.626111"> ❺         <ele>7.125</ele> ❻         <time>2010-09-06T00:14:34Z</time>         </trkpt>         <trkpt lat="45.521292" lon="-122.625950">           <ele>6.831</ele>           <time>2010-09-06T00:14:37Z</time>         </trkpt>         ...       </trkseg>     </trk>    </gpx> ```    Most XML files begin in a similar way ❶ to show they contain XML. Then the code points to the GPX schema ❷. With the formalities out of the way, let's dive into the actual data.    This example is showing a track, so we begin by wrapping everything in a `<trk>` tag. A track has at least one segment ❸, which contains the individual track points ❹. The latitude and longitude are stored as attributes of the `<trkpt>` tag, with the elevation ❺ and timestamp ❻ as subelements.    A program going through the track points uses differences in the latitude and longitude to determine the distance between points. Similarly, the number of seconds or minutes between timestamps can be used to determine the approximate speed. The elevation numbers can determine the trail's grade. We can learn a lot from the metadata.    ## Examples of GPX    In addition to trail enthusiasts plotting and sharing their escapades, GPX has other even more useful applications. GPX is bringing geographic data that wasn't included before to rich content like photos and video. A world of volunteer cartographers also use it to map streets and cities.    Walking around snapping shots is something most photographers do with regularity, whether hiking through nature or walking urban streets. By synchronizing the internal times on a GPS and camera, you can get latitude and longitude coordinates for your photos.    Digital cameras usually timestamp each photo. If you have a record of a path, like the one stored in a GPX file, cross-referencing the two is straightforward. Just find the track point with a timestamp closest to the photo's. Update the photo metadata to include the coordinates, and you have now geo-tagged the photo. You can install many programs on your computer that will do this for you.    OpenStreetMap uses GPX to map the world. It sends volunteers to walk the streets with GPS units. Then, the track points, along with other information like street names, are used to create maps that are available for anyone—without licensing fees.    In many countries, such as the United States, much of this street data is already available. OpenStreetMap volunteers, in these cases, are checking accuracy and filling in what's missing. In some places, OpenStreetMap is all there is, so the GPX tracks become incredibly important to the project.    ## Display GPX Tracks on a Map    Once you have a GPX file, you'll want to do something with it, like show it on a map. Getting at the track points requires parsing the XML and then "connecting the dots" with polylines in Mapstraction.    XML parsing can be painful. I discussed it in detail in the earlier in this chapter. We'll use the jQuery method here, which is as easy as it gets, but it does require a small JavaScript library.    To start, let's lay the groundwork for the GPX map by preparing the basic HTML, CSS, and JavaScript to display a simple map. Put this code into a new HTML file:    ``` <html >   <head>     <title>GPX on a Map</title>     <script src="*`http://maps.google.com/maps/api/js?sensor=false`*"  type="text/javascript"></script> ❶   <script src="http://ajax.googleapis.com/ajax/libs/ jquery/1.3/jquery.min.js"></script>     <script src="mxn.js?(googlev3)"></script>     <style>       div#mymap {         width: 600px;         height: 450px;       }     </style>     <script type="text/javascript">     var mapstraction;     function create_map() {       mapstraction = new Mapstraction('mymap', 'google'); ❷     mapstraction.setCenterAndZoom(new LatLonPoint(0,0), 2);       mapstraction.addControls({"zoom": "large"}); ❸     parse_gpx("gpxfile.gpx");     }     </script>   </head>   <body onload="create_map()">     <div id="mymap"></div>   </body>   </html> ```    We haven't quite parsed the GPX yet. This code just prepares a basic map. Make sure you include your Google Maps API key. Otherwise, everything is ready to go. Before moving on, however, I'd like to point out a few bits.    First, I've included the jQuery JavaScript framework ❶. We'll use it for the Ajax call that will download the GPX file. Also, jQuery makes XML parsing easier, so we'll use it to get at the GPX data.    The eventual location for the map will be determined by the latitude and longitude values inside the GPX file. Because we don't know what those are yet, I centered the map in the middle of the globe ❷. Assuming we're able to load data, that location will only stay for a short time. If you know what city the data will be in, a good practice is to use the coordinates of the city center.    Finally, I make a call to the `parse_gpx` function ❸, passing a filename. Make sure *gpxfile.gpx* exists in the same directory and has some GPX data. You can find example files at [`mapscripting.com/gpx-files`](http://mapscripting.com/gpx-files).    But wait . . . where is the `parse_gpx` function? We haven't added it yet, so let's write it! Add these lines below the `create_map` function:    ``` function parse_gpx(filename) { ❶   jQuery.get(filename, {}, function(xmltxt) {       var pdata = {"color": "blue"};       var pts = []; ❷     jQuery("trkpt", xmltxt).each(function(i) {         var lat = ❸parseFloat($(this).attr("lat"));         var lon = parseFloat($(this).attr("lon")); ❹       var thispt = new LatLonPoint(lat, lon); ❺       pts.push(thispt);       }); ❻     mapstraction.addPolylineWithData(new Polyline(pts, pdata)); ❼     mapstraction.autoCenterAndZoom();     });   } ```    The `parse_gpx` function is really just a wrapper for the jQuery Ajax call ❶. It grabs the passed filename and returns the XML results to an anonymous, inline function. This is where the real stuff happens. You can see how one of my GPX files looks when added to the map in Figure 8-1.  ![Example GPX tracks from a walk in the park](img/httpatomoreillycomsourcenostarchimages672079.png.jpg)  Figure 8-1. Example GPX tracks from a walk in the park    After creating a few variables, the function uses jQuery to look for every `<trkpt>` in the GPX file ❷. For each track point, it calls yet another anonymous function, which is sort of equivalent to a `for` loop. Each time through the loop, the function parses the latitude and longitude of the current track point.    The `parseFloat` JavaScript function ❸ takes the text from the GPX file and turns it into the decimal number (also called a *floating point number*) needed. Again, the function uses jQuery to parse the GPX, but it uses the shorthand dollar sign method.    Before we draw the track on the map, we need to have all the data points to pass to Mapstraction at the same time, as shown in #16: Draw Lines on a Map in #16: Draw Lines on a Map. We'll use an array to collect the points. Once we have the latitude and longitude of the current track point, we create a `LatLonPoint` with the two values ❹. Then we add the point to the `pts` array ❺.    Once we're outside the loop, we pass our data to draw the Polyline ❻ and then zoom the map automatically to show our entire Polyline ❼. Now we have, in the case of most tracks, a tight view of the GPX data.    # #57: Convert from XML to JSON    As I've mentioned, XML is not always the easiest format to use with JavaScript. Yet, as this chapter has shown, most of the formats you'll be working with are flavors of XML. By now you probably prefer JSON, right? To make things easier on the JavaScript, we'll need to do a little extra work and convert from XML to JSON.    A number of ways are available for getting our data from XML to JSON. And really, the conversion is not that difficult an operation. For example, once we load either XML or JSON into PHP, the data is very similar. In this section, I'll show how you can convert on your own server and also introduce you to a nifty service from Yahoo! called Pipes.    ## Convert Using PHP    Most Unix or Linux servers have a copy of PHP already running, which makes it a great server-side language to learn. We'll use PHP to read in and parse some XML and then turn around and encode it into JSON. The whole process takes just a few lines, thanks to helper functions included in PHP 5.    Even if you don't have a server, or PHP is not accessible, you can likely install it on your own machine. For a more in-depth discussion of PHP, you'll want to check out Chapter 9.    To start converting, create a new PHP file and add these lines:    ``` <?   $xmltxt = ❶get_url("http://mapscripting.com/example.xml"); ❷ $xmlobj = simplexml_load_string($xmltxt);   $jtxt = ❸json_encode($xmlobj);   print $jtxt;   ?> ```    Like I said, this code isn't very complicated, is it? To get the XML text, I use a helper function ❶ I wrote, which is explained in #61: Retrieve a Web Page in #61: Retrieve a Web Page. You could also read in the file directly, as we did when parsing XML with PHP.    Once we have the XML text, we convert it first to a PHP object ❷. This parses the hierarchy of the XML file into a format PHP can understand directly. Because JSON is so close to data structures found within PHP, a simple call ❸ is all that's needed to encode the JSON. Then we print it and voilà —we have converted data formats.    ## Convert Using Yahoo! Pipes    Don't have server-side chops, or just don't want to deal with additional code? Many folks are turning to a service from Yahoo! called Pipes. Pipes reads in various data formats, lets you massage the data if you want, and then outputs it as either XML or JSON.    Why use Yahoo! Pipes instead of your own server? For one, the conversion process is even easier than using PHP. Plus, no code is required because Pipes has a graphical, drag-and-drop interface. You also get the benefit of Yahoo!'s infrastructure. Yahoo! decides how often to check for new content and deals with caching the most recent copy—something we didn't do at all in the PHP example.    The downside to relying on Pipes? It introduces another point of failure. Even if your server is humming along, your map might not work if Pipes crashes. Although you can likely count on Yahoo! for uptime, what if the company decides the Pipes product isn't worth keeping around? You'd be hung out to dry.    To me, the ease of using Pipes outweighs the detractions. Let's see just how easy converting from XML to JSON with Pipes is.    You'll need a Yahoo! account to store your Pipes. Log in at [`pipes.yahoo.com/pipes`](http://pipes.yahoo.com/pipes) and click **Create a Pipe** to go to a blank canvas, with options on the left. First, Pipes requires a data source, so drag a Fetch Feed source on to the canvas. Paste your feed URL into the box, or use [`mapscripting.com/example.xml`](http://mapscripting.com/example.xml).    Now a second box called Pipe Output should also appear at the bottom of the workspace. Drag the circle at the bottom of the Fetch Feed box to the circle at the top of the Pipe Output. This connects the elements to create a complete Pipe, as shown in Figure 8-2. Go ahead and try it and you should see sample output in the debugger at the bottom of the workspace.  ![Yahoo! Pipes feed connected](img/httpatomoreillycomsourcenostarchimages672081.png.jpg)  Figure 8-2. Yahoo! Pipes feed connected    Here's where things get really interesting: *You're done*. Save the Pipe, and then click **Run Pipe**. You should see your content within the Pipes interface. At the top of your content, you'll see a Get as JSON link. That's the URL to your converted data run through Yahoo!'s servers. Now you can order new business cards because you're a certified data plumber!    # #58: Filter, Merge, and Sort Data with Yahoo Pipes!    APIs and RSS feeds are becoming commonplace. So much of the Web's content is now available in a format that programmers are quickly able to use, which is great. The downside is that another problem has been created: Getting at just the right information can be tough.    Sometimes a feed is a fire hose when a garden variety hose would do. Other times you have to combine multiple sources before you get the information you really need. Yahoo! Pipes has an easy interface for solving both issues—by filtering and merging data.    Much of this data-munging is stuff programmers have been doing manually using server-side scripts. Sometimes that will still be necessary, but Yahoo! Pipes is able to solve the common scenarios. Plus, for reasons described in the previous project, offloading some of this work onto Yahoo!'s servers provides some major benefits.    Before you create a Pipe, you'll need at least one data source. This source is often an RSS feed and, for map-related projects, may be GeoRSS. A Pipe is a way to transform data sources. The data comes in the Pipe, some stuff happens, and then the data flows out of the Pipe. If you have a data source, then let's start working on that "stuff" part. You can begin by editing the Pipe we created in the previous section.    ## Filter Your Feed's Content    Rather than simply running our feed through Pipes, let's try filtering out certain content. To do this, we'll need to drag a Filter box from the Operators menu. You can place the box anywhere in the workspace, but placing it between the Fetch box and the Output box may make the most sense.    Next, you'll need to connect the feed to the filter. Drag the circle at the bottom of the Fetch box to the top of the Filter box. Then connect the Filter box to the Pipe Output by dragging the filter's bottom circle to the output's top circle.    Of course, the filter isn't useful unless it's actually filtering some content. You can do this two ways with Pipes: You can filter *in* or filter *out*. At the top of the Filter box, you can select **Block** to filter out and **Allow** to filter in.    Pipes can filter based on any fields in the feed. A common choice is the title, which, for RSS, is *item.title*. Click the arrow next to the first text area in the filter box (see Figure 8-3) and you'll see a list of available fields. Then you can type the words you want to filter in the second text field. The drop-down box lets you perform some basic types of filtering on the field, including greater-than/less-than for numbers.  ![Filtering out items with map in the title](img/httpatomoreillycomsourcenostarchimages672083.png.jpg)  Figure 8-3. Filtering out items with *map* in the title    If you want additional filters, just click the plus sign next to Rules. Otherwise, your Pipe is complete. Save it, and then click **Run Pipe**. Your filtered feed will be shown within the Pipes interface. To get this new feed, choose **Get as RSS** or **Get as JSON**. If you are reading the feed in to use with JavaScript, JSON is probably your best choice.    ## Merge Two or More Feeds    What if you have two similar feeds that you want to combine? Pipes is very good at this! Create a new Pipe and drag two Fetch Feed boxes to the canvas (see Figure 8-4). Choose two feeds (if you're short of examples, you should be able to find an RSS feed from your favorite blogs or websites) and insert their URLs into the Fetch Feed boxes.  ![Merging and sorting two feeds with Yahoo! Pipes](img/httpatomoreillycomsourcenostarchimages672085.png.jpg)  Figure 8-4. Merging and sorting two feeds with Yahoo! Pipes    If the feeds are of the same variety and you don't plan to filter anything, you can actually use a single Fetch Feed box. Just click the plus sign next to URL. In many cases, you'll want to have the option to perform more advanced operations, so I recommend using a Fetch Feed for each individual feed.    To combine the two feeds, we'll need a Union box from the Operators menu. Drag the Union box to the canvas below the two Fetch Feed boxes. Drag the circle below each feed to one of the five circles at the top of the Union box. Finally, drag the circle at the bottom of the Union box to the Pipe Output.    Your feeds are combined in the order they were added to the Union box (left to right). In other words, the second feed's content is only seen after the entire contents of the first feed. This arrangement is not ideal. Most likely, you usually want to see the content in the order it was published. Pipes can do the sorting for you.    Drag a Sort operator to the canvas. Connect the bottom circle of the Union box to the top of the Sort box. Then connect the bottom of the Sort box to the Pipe Output. You'll need to choose a field to sort by clicking the arrow next to the text field within the Sort box. To use the date, select **item.pubDate**. Now save the Pipe and you're done.    You've now filtered, merged, and sorted with Yahoo! Pipes. You can transform data into whatever you want it to be. In fact, if you dig through the documentation a bit ([`pipes.yahoo.com/pipes/`](http://pipes.yahoo.com/pipes/)), you'll realize Pipes is even more powerful than the few examples I've shown. You can load feeds dynamically, use web services, and even extract location from text. Best of all, when you're done, the data is in a format that is easy to read in and plot on a map.```` `````
