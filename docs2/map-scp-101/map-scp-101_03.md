# 第三章。地理编码

![无标题图片](img/httpatomoreillycomsourcenostarchimages671943.png.jpg)

我在这本书的前两章中演示了一些有趣的地图示例，但房间里有一头大象。我在描述地球上的一个点中解释的描述地球上的一个点中的经纬度点显然对计算机很有用，但对人类来说并不如此。你有多少次被邀请到一个仅用地理坐标指定的地点参加派对？

在本章中，我将展示如何将对你有意义的事物——地址、城市名称，甚至邮政编码——转换成 Mapstraction 所需的纬度和经度点。这种转换称为*地理编码*，我们将探讨你可以在自己的地图项目中使用的几种方法。

# 地理编码器是如何工作的？

初看起来，地理编码器就像一个先知。你给它一个地址，它看看它的水晶球，然后给出一个回复。就像魔法一样，当这些坐标在地图上标出时，它们正好位于地址所在的位置。让我们揭开幕布，看看地理编码器是如何工作的。

就像大多数魔法一样，你可能会失望地发现，大多数地理编码实际上只是估计。地理编码并不优雅——它有点像蛮力。

首先，地理编码器会将地址分解成各个部分。例如，考虑格莱美庄园的地址：

> 3734 Elvis Presley Blvd, Memphis, TN
> 
> 街道号码，街道名称，后缀，城市，州

城市很重要，因为信不信由你，其他城市也有 Elvis Presley Blvd。当你想到更常见的名称（比如，Main Street）时，你可以看到这一步是如何变得重要的。事实上，在城市内部，道路的后缀可能是一个问题，同一条街道可能被称为 Avenue、Street、Circle 等等。如果你认为这些变化在导航时很令人困惑，那么想想它们如何让地理编码器困惑。

街道名称匹配可能是修复位置中最困难的部分。处理拼写错误和其他名称格式化方式可能很困难。像许多其他大城市一样，我家乡俄勒冈州的波特兰市有一条以马丁·路德·金爵士命名的街道。这条街道在波特兰和其他城市都通常被称为*MLK*。是否应该让地理编码器识别这个缩写是有争议的，但这让你了解你必须考虑的一些问题。

现在你已经确定了正确的街道并知道它的城市，你需要找到街道上的实际地址。存储世界上所有的地址是不必要的，而且由于新地址不断产生，这会很困难。相反，大多数地理编码器使用街道段，如图图 3-1 所示。

![地理编码器使用街道段来估计位置。](img/httpatomoreillycomsourcenostarchimages671983.png.jpg)

图 3-1. 地理编码器使用街道段来估计位置。

以我们的格拉辛德（Graceland）为例，我们可能知道伊丽莎白·泰勒大道（Elvis Presley Blvd）的一段从 3700 号开始，到 3799 号结束。我们还知道该段两端的地纬度和经度点。使用这些信息，我们可以估计格拉辛德（3734 号）大约位于 3700 号和 3799 号之间三分之一的位置。使用这两个点，地理编码器可以计算出大致的位置。

# JavaScript 与 HTTP 地理编码

在本章中，我将介绍两种从地址中检索纬度和经度结果的主要方法：通过 JavaScript 地理编码 API 或通过 HTTP 上的地理编码网络服务。表面上，这些方法非常相似，因为返回的数据是相同的。区别在于你可以用结果做什么。

*JavaScript 地理编码器*使用客户端代码调用外部服务器，这意味着代码在浏览器中运行。这种方法与 Mapstraction 和每个映射 API 所使用的方法相同。从这个意义上说，JavaScript 地理编码器对于网络地图开发者来说非常方便。

*HTTP 地理编码器*也调用外部服务器，但这是从你的服务器进行的，所以代码在浏览器外运行。这种方法与我在第九章（ch09.html "第九章。服务器端编程"）中使用的网页编程语言 PHP 所使用的方法类似。事实上，你可能会使用 PHP 来解释 HTTP 地理编码器的结果。

为什么你会选择使用一种类型的地理编码器而不是另一种？假设地理编码器来自同一提供商（例如，谷歌），数据质量是相似的。JavaScript 地理编码器通常只是服务器端地理编码器的一个包装器。换句话说，两种类型最终都会调用相同的数据集。

关于使用哪种地理编码器的决定取决于你对输入和输出的自由度有多少。你只能以非常有限的方式使用 JavaScript 地理编码器。输入很可能来自用户通过表单。输出几乎肯定是要发送到网页或网页上的地图。

当然，你大部分的位置数据最终都会出现在地图上。然而，在发生这种情况之前，你可能想做很多事情。获取输出控制是你会使用 HTTP 地理编码器的主要原因之一。当结果在服务器端检索时，你有选项将其存储到数据库中，缓存数据，通过短信或电子邮件共享数据，或者将其发送到第三方服务。其中一些选项可能使用 JavaScript 地理编码器是可能的，但它们不容易实现，因为它们超出了 JavaScript 地理编码器的正常用途——在浏览器内部。

通过使用 HTTP 地理编码器，您控制着输入的来源。例如，您可能从数据库、第三方服务或地址列表中检索它。使用 JavaScript 地理编码器时，输入很可能直接来自用户在网站上的输入框中输入的数据。对于许多用途，这已经足够了，但有时您可能需要更多的自由度。

如果您花足够的时间制作网络地图，您可能会结合使用 JavaScript 和 HTTP 地理编码。本章的其余部分将重点介绍您在将可读地址转换为地理坐标（反之亦然）时可以使用的不同服务。

# #12: 使用 JavaScript 进行地理编码

当您想要在网页浏览器内进行地理编码时，您会使用 JavaScript，这是 Mapstraction 和它所支持的每个地图服务提供商所使用的相同编程语言。您可以从用户那里获取城市名称或地址，并将其发送进行地理编码。响应将发送到您声明的 JavaScript 回调函数。

与地图 API 一样，您可以选择自己的 JavaScript 地理编码器。不出所料，我将展示 Mapstraction 的地理编码器，它将使用您指定的任何提供商，并为您提供使用 Mapstraction 创建地图时相同的灵活性：编写一次代码，轻松切换提供商。

由于并非每个地图都需要地理编码，因此地理编码器被保留在主 Mapstraction 代码之外。此外，并非每个提供商都有地理编码器，因此每个都单独存储。在这个例子中，我们将使用 Google。您可以从 [`mapstraction.com/`](http://mapstraction.com/) 下载 `mxn.google.geocoder.js`。

您可以在代码中像包含您的地图提供商和 Mapstraction 本身一样包含 Mapstraction 地理编码器。假设文件位于当前目录中，在基本地图的 `<head>` 部分添加此行：

```
<script src="mxn.google.geocoder.js"></script>
```

包含脚本后，使用以下代码初始化地图、将地址进行地理编码并在地图上标记结果：

```
var mapstraction, geocoder;
  function create_map() {
    mapstraction = new mxn.Mapstraction('mymap', 'googlev3');
    geocoder = new MapstractionGeocoder(❶add_point, ❷'googlev3');
    // Create address object
❸   var address = {
      street : "38 Ringold Street",
      locality : "San Francisco",
      region : "CA",
      country : "US"
    };
❹   geocoder.geocode(address);
  }
  function add_point(loc) {
    var mk = new mxn.Marker(❺loc.point);
    mk.setInfoBubble(❻loc.address);
    mapstraction.addMarker(mk);
    mk.openBubble();
    mapstraction.autoCenterAndZoom();
  }
```

与本书中的大多数地图示例一样，我使用 `create_map` 函数来初始化地图。您可以给这个函数取任何名字，只要在页面加载时调用这个函数即可。

要创建 Mapstraction 地理编码器对象，我们需要提供一个回调函数 ❶ 并告诉 Mapstraction 我们想使用哪个提供商 ❷（提供商的 JavaScript 仍然需要包含）。回调函数接收地理编码的结果。首先，我们需要给地理编码器提供一个地址或城市名称。

我们创建一个通用的 JavaScript 对象 ❸ 来存储文本位置信息。然后，我们为地址的各个部分添加属性。名称可能有些奇怪，因为 Mapstraction 正在尝试在全球范围内工作，而不仅仅是您的国家。

一旦地址的部分被填写完整，你可以将其发送到地理编码器 ❹。然后，你的回调函数将作为位置对象传递结果，该对象本身也有一些有趣的属性。最重要的是地址的`LatLonPoint` ❺。在这个例子中，我使用了这个点来创建一个新的标记。

标记需要一个描述，我会把它放在一个消息框里。我使用的是完整的地址 ❻，这是地理编码器清理过的。正如你在图 3-2 中看到的，地理编码器添加了邮政编码，将“Street”转换为“St”，并将国家称为“USA”而不是“US”。

![No Starch Press 办公室的地理编码结果](img/httpatomoreillycomsourcenostarchimages671985.png.jpg)

图 3-2. No Starch Press 办公室的地理编码结果

当然，这个例子只具有这样的用途。地址是硬编码的。你很可能会从用户那里获取输入。

## 地理编码用户输入

JavaScript 地理编码器通常需要用户输入才能发挥作用。让我们调整上一节中的代码，从输入框中获取文本位置并进行地理编码。

首先，你需要一个地方让用户输入数据。让我们在我们的 HTML 中添加一个表单，要么在地图`div`上方，要么在下方：

```
<form id="addrform">
  <input type="text" id="newpt" />
  <input type="submit" id="butnew" value="Geocode" />
</form>
```

现在我们需要在用户输入地址或城市名称时做些事情。下面是`create_map`函数的新版本，其中做了一些修改：

```
function create_map() {
    mapstraction = new mxn.Mapstraction('mymap', 'googlev3');
    mapstraction.setCenterAndZoom(new mxn.LatLonPoint(0, 0), 1);
    mapstraction.addSmallControls();
    geocoder = new MapstractionGeocoder(add_point,'google');
❶     document.getElementById('addrform').onsubmit = function() {
      var address = {
❷     document.getElementById('newpt').value
      }
      geocoder.geocode(address);
      return false; // avoids posting form to the server
    };
  }
```

我们在得到输入之前不能进行地理编码，而输入只有在用户提交表单时才会发生。所以我们等待提交 ❶，然后使用匿名函数采取行动。我们也可以使用命名函数，但只有几行代码，直接在代码中编写是最容易的。

在上一节中，我分别处理了地址的每一部分。在用户输入的情况下，这并不总是可能的。相反，我已经将输入框的整个值提供给地理编码器 ❷。

回调函数`add_point`可以保持不变。输入一个地址，它会在地理编码的点处添加一个标记。再次操作，现在你有了两个标记。这种地理编码可能会上瘾。

# #13: 使用 HTTP 网络服务进行地理编码

当你希望在浏览器中进行地理编码时，JavaScript 非常方便。然而，通过使用网络服务，你可以获得更多对地理编码的控制。地址可以来自任何地方，包括地点列表。结果可以存储在任何地方，包括数据库，这样你就可以随时访问并在地图上绘制它们。

有几个地理编码器可供选择，包括来自 Google 和 Yahoo!的各一个。每个的输入略有不同，结果也是如此。在接下来的几节中，我将概述如何使用这两个地理编码器，并指向一些其他的地理编码器。

## 使用 Google 的地理编码网络服务

Google 是网络地图世界的王者，因此当然，Google 有一个地理编码网络服务。您可以使用它将地址或城市转换为经纬度点，并以几种数据格式之一获取结果。作为额外的好处，Google 为您美化了地址，并在查询模糊时提供了多个选项。

使用网络服务进行地理编码的额外自由之一是您可以在网页浏览器中查看结果，而无需编写任何实际代码。请在您的浏览器中查看以下 URL：

```
http://maps.google.com/maps/geo?`q`=38+Ringold+Street+San+Francisco+CA&`output`
=xml&`sensor`=false
```

您可以向 Google 发送的参数以粗体显示，以及所有在表 3-1 中显示的可能参数。整个地址作为`q`查询参数传递。您也可以简单地在这里包含一个城市名称或邮政编码。

表 3-1. Google 地理编码器参数

| 参数 | 描述 |
| --- | --- |
| `q` | *必需*。查询，例如地址或城市名称 |
| `sensor` | *必需*。是否来自移动设备：`true`或`false` |
| `output` | 结果格式：`json`（默认），`xml`，`kml`或`csv` |
| `gl` | 国家代码作为顶级域名。例如，`us`，`ca`，`uk` |

我们想要的`output`是 XML 格式，大多数编程语言都可以读取。除非您是从移动设备使用此地理编码器，否则将`sensor`设置为 false。

现在我们来看看从 Google 的 HTTP 地理编码器调用中得到的 XML 结果：

```
<?xml version="1.0" encoding="UTF-8" ?>
 <kml >
   <Response>
     <name>37.7740486,-122.4101883</name>
     <Status>
       <code>200</code>
       <request>geocode</request>
     </Status>
❶    <Placemark id="p1">
❷     <address>38 Ringold St, San Francisco, CA 94103, USA</address>
       <AddressDetails Accuracy="8"
                       >
        <Country>
         <CountryNameCode>US</CountryNameCode>
         <CountryName>USA</CountryName>
         <AdministrativeArea>
❸         <AdministrativeAreaName>CA</AdministrativeAreaName>
           <Locality>
❹           <LocalityName>San Francisco</LocalityName>
             <Thoroughfare>
❺             <ThoroughfareName>38 Ringold St</ThoroughfareName>
             </Thoroughfare>
             <PostalCode>
❻             <PostalCodeNumber>94103</PostalCodeNumber>
             </PostalCode>
           </Locality>
         </AdministrativeArea>
       </Country>
     </AddressDetails>
     <ExtendedData>
       <LatLonBox north="37.7773156" south="37.7710204"
                  east="-122.4071324" west="-122.4134276" />
     </ExtendedData>
     <Point>
❼     <coordinates>-122.4102800,37.7741680,0</coordinates>
     </Point>
   </Placemark>
  </Response>
 </kml>
```

结果实际上是在*Keyhole 标记语言*（KML）中，这是一种 XML 的变体（参见#55: 使用 KML）。地理编码地点的坐标和其他信息存储为 Placemark。在我们的例子中，我们只有一个 Placemark❶，因为对于完整的地址只有一个可能的结果。在模糊的情况下（比如说我们搜索了简单的“Springfield”——许多地方都有这个名称），最佳结果将作为第一个 Placemark 列出，其他结果将获得递增的 id（即，p2，p3 等）。

事实上，如果您想让您的应用程序显示可能的结果，就像[`maps.google.com/`](http://maps.google.com/)所做的那样，请使用每个 Placemark 的完整格式化地址❷。您可以看到，谷歌甚至清理了我的具体地址，将“Street”转换为“St”并添加了邮政编码。

地址的各个部分也可以单独访问，但标签名称可能对您来说很奇怪。这是因为 Google 使它们通用，所以标签不会使不在美国的人感到困惑。例如，州缩写❸被称为*行政区域名称*。

单独访问值可以更轻松地显示城市❹或仅地址❺。此外，单独访问它们是快速确定一个地方的邮政编码❻（在美国称为 ZIP Code）的方法。

最后，地理编码最重要的部分是纬度和经度点。这些点存储在一个单独的标签 ❼ 中。你可以使用分割函数（下一节将展示 PHP 中的一个）来检索坐标的各个部分。

你注意到谷歌提供了三个数字而不是两个吗？第三个数字代表海拔，是 KML 格式的一个属性。地理编码器不会发送这个值，所以它总是为零。

如果你需要帮助将这些地理编码结果引入你的应用程序，第九章可以展示如何在 PHP 中这样做。或者，如果你只需要坐标，请继续阅读，了解谷歌对真正简单地理编码的方法。

### 其他数据格式

我喜欢 XML，但它并不总是首选的数据格式。谷歌的地理编码网络服务提供了多种格式选择，包括 JavaScript 对象表示法 (JSON) 和逗号分隔值 (CSV)。后者在你只想“只看事实”时非常棒。

在 URL 中放入一个 `output` 参数，并将其值设置为所需的格式，例如：

```
http://maps.google.com/maps/geo?q=38+Ringold+St+San+Francisco
+CA&output=`csv`&sensor=false
```

在这里，我们请求 CSV。与其他结果格式不同，我们不会得到重写的地址、邮编或其他任何好处。然而，我们确实得到了四个最重要的值，由逗号分隔：

```
200,8,37.7741680,-122.4102800
```

第一部分是来自服务器的代码。`200` 表示我们有一个好的结果。其他任何东西，我们可能有一个错误。

第二部分是一个数字，代表我们结果的粒度。它是街道级别（一个地址）、邮政级别还是城市级别？可能的结果大致相当于缩放级别，如表 3-2 所示。

表 3-2. 地理编码精度级别代码

| 代码 | 描述 |
| --- | --- |
| 0 | 未知 |
| 1 | 国家 |
| 2 | 州（或类似区域） |
| 3 | 县（或其他子区域） |
| 4 | 城市 |
| 5 | 邮编 |
| 6 | 街道 |
| 7 | 交叉点 |
| 8 | 地址 |
| 9 | 建筑物（如地标） |

CSV 结果的最后两个数字可能看起来很熟悉。它们是纬度和经度点（按此顺序）。这些结果可能是最重要的，因为地理编码的全部内容就是将城市名称或地址转换为可绘制的坐标。

这里有一些简单的 PHP 代码，它调用谷歌地理编码网络服务，解析 CSV 结果，并将坐标保存到变量中：

```
<?
  $url = "http://maps.google.com/maps/geo?q=38+Ringold+St+San+Francisco+CA";
❶ $url += "&output=csv&sensor=false";
  $csvtxt = ❷get_url($url);
  $llarray = ❸explode(",", $csvtxt);
  if (❹count($llarray) == 4 && ❺$llarray[0] == "200") {
    $lat = $llarray[2];
    $lon = $llarray[3];
    // Now do something here with the $lat and $lon variables
  }
  // Additional PHP code/functions could go here
 ?>
```

这段代码只是一个片段，以给你一个如何分离简单 CSV 结果（如本例中所示）的思路。如果你运行这段代码，什么也不会发生，因为我所展示的只是将结果存储在变量中。

首先，我们创建一个变量来保存我们将用于调用谷歌的 URL。因为变量很长，而这本书的页面只有那么宽，所以我将其拆分为两行❶，但对于 PHP 来说，这个变量是一个完整的文本字符串。

然后将 URL 传递给 `get_url` 函数❷，这是一个我将在 #61: 获取网页 中向你展示如何编写的函数。#61: 获取网页。你需要包含一个包含该代码的文件或在你 PHP 文件的底部附近粘贴该函数的副本。

一旦我们从谷歌得到结果，我们就将文本 ❸ 分解成几个部分，所有这些部分都存储在一个单独的数组变量中。因为数据是用逗号分隔的，所以我们将使用逗号作为分隔符来分割文本。

将部分存储为数组的元素后，我们几乎准备好获取我们的纬度和经度了。我们需要确保数组变量有四个结果 ❹，正如预期的那样。此外，结果中的第一个数字必须是 `200` ❺，这是良好结果的代码。

因为 PHP 中的数组从零开始计数，所以我们的纬度和经度被存储在 explode 结果变量的第二个和第三个索引中。用很少的 PHP 代码和更少的文本，你现在已经成功地将一个地址转换成了地理坐标。

## 使用 Yahoo! 地理编码网络服务

虽然谷歌可能获得了大部分的媒体报道，但 Yahoo 的地理开发者工具非常出色。这种情况适用于其易于使用、功能齐全的地理编码网络服务。你传递一个城市名称或完整的地址，Yahoo! 就会输出带有坐标和其他地理数据的简单 XML。

因为结果只是普通的 XML，你可以在你的网络浏览器中查看它，以了解该服务的工作方式。访问这个 URL：

```
http://local.yahooapis.com/MapsService/V1/geocode?`appid`=*`YOURKEY`*&
`street`=38+Ringold+St&`city`=San+Francisco&`state`=CA
```

参数以粗体显示。你需要将你的 API 密钥作为 `appid`。这个 ID 与 Yahoo! 地图 API 中的 ID 相同。我在 创建 Yahoo! 地图 中向你展示了如何注册 ID。创建 Yahoo! 地图。

在这个示例中，地址的部分被分割成 `street`、`city` 和 `state`。你也可以用一个参数表示地址，类似于 Google 的地理编码器：

```
http://local.yahooapis.com/MapsService/V1/geocode?appid=*`YOURKEY`*&
`location`=38+Ringold+St+San+Francisco+CA
```

`location` 参数包含上一个示例中的所有部分，但将它们放在一个地方。如果你从用户那里接收地址作为输入，你将更喜欢这个选项，除非你有方法将地址分解成各个部分（例如多个表单字段）。

无论你以何种方式调用 API，结果将以相同的方式格式化：

```
<?xml version="1.0"?>
<ResultSet ...>
  <Result precision="address">
    <Latitude>37.774155</Latitude>
    <Longitude>-122.410230</Longitude>
    <Address>38 Ringold St</Address>
    <City>San Francisco</City>
    <State>CA</State>
    <Zip>94103-4403</Zip>
    <Country>US</Country>
  </Result>
</ResultSet>
```

与 Google 的 XML 结果相比，这些结果非常简单。纬度和经度分别显示，地址的各个部分也是如此（即使你发送的是一个文本字符串，Yahoo! 也会为你分开）。只要你在美国对地址或城市进行地理编码，每个字段都有意义。例如，在加拿大，你必须知道省份存储在 `<state>` 标签中。

如果你的搜索结果模糊，例如城市名称不唯一，Yahoo! 将将最佳结果放在首位。其他结果将跟随在其自己的 `<Result>` 标签内。

由于你在这里使用的是简单的 XML，你可以像解析任何其他 XML 一样解析它们。#52: 使用 XML 在 #52: 使用 XML 中展示了这个过程在 PHP 和 JavaScript 中的实现。如果你需要其他格式，Yahoo! 提供了 JSON 或序列化 PHP 格式的结果。第一种格式在 #53: 使用 JSON 中介绍，而第二种格式在 [`developer.yahoo.com/common/phpserial.html/`](http://developer.yahoo.com/common/phpserial.html/) 中解释。

## 其他地理编码网络服务

之前的例子展示了地理编码器的两种最可能的选择，但你还有其他选择，尤其是如果你愿意为服务付费的话。为什么要在 Google 和 Yahoo! 免费提供地理编码的情况下花钱呢？你的选择实际上取决于服务条款和速率限制，这些限制可能会限制你在高流量、商业用途中对地理编码器的使用。

你不一定需要打破那个储蓄罐来使用付费地理编码器。例如，*geocoder.us* 只需收取四分之一美分来对地址进行地理编码。有关地理编码服务器的最新列表，请参阅 [`mapscripting.com/geocoders/`](http://mapscripting.com/geocoders/)。

# #14: 反向地理编码：从点获取地址

到目前为止，我们使用的是人类可读的信息——城市名称或地址——来检索经纬度坐标点，这对于计算机来说更容易理解。有时，你可能想反过来操作。如果你只有一组坐标，你可以使用它们进行反向地理编码，以获取对人类有意义的地址和其他地理信息。

正常地理编码既复杂又不精确，但反向地理编码更是如此。首先，地理编码器找到离坐标最近的街道；然后，它确定哪个地址属于该点。说实话，结果往往是地址范围。

考虑到它并不完美，反向地理编码可能看起来有点荒谬。但随着位置信息在互联网上的普及，反向地理编码将会变得更加常见。例如，考虑 #48: 使用 JavaScript 获取位置 在 #48: 使用 JavaScript 获取位置。在许多情况下，GPS 或其他报告某人位置的设备只会提供经纬度坐标点。这些信息足以在地图上标出，但对于查看信息的人类来说，信息并不充分。

在以下部分，我将展示两个服务，这两个服务都来自 Google，它们将提供反向地理编码，帮助你从可读数据中创建地理信息。

## 使用 JavaScript 进行反向地理编码

如果你使用 Google 作为你的地图服务提供商，反向地理编码可以在你的 JavaScript 代码中发生，甚至不需要加载 Mapstraction 的地理编码器（它只支持 *正向* 地理编码）。在这个例子中，我们仍然使用 Mapstraction，因为反向地理编码只是地图代码的一个小部分。

让我们创建一个基本的 Mapstraction 地图，使用 Google 作为提供者。我们将地图的中心转换为 Google 点，并发送它进行反向地理编码。

假设你的 HTML 设置如 创建 Mapstraction 地图 中所述，以下是创建地图并调用 Google 地理编码器的 JavaScript 代码：

```
var mapstraction;
 function create_map() {
   mapstraction = new mxn.Mapstraction('mymap', 'googlev3');
   mapstraction.setCenterAndZoom(
     new mxn.LatLonPoint(37.7740486,-122.4101883), 15);
   // Google-specific calls
❶  var geocoder = new GclientGeocoder();
   geocoder.getLocations(❷mapstraction.getCenter().toProprietary(mapstraction.api),
      ❸found_address);
 }
 function found_address(response) {
   if (response && ❹response.Status.code == 200) {
❺    var pt = response.Placemark[0].Point;
     var marker = new mxn.Marker(new mxn.LatLonPoint(pt.coordinates
[1], pt.coordinates[0]));
     marker.setInfoBubble(❻response.Placemark[0].address);
     mapstraction.addMarker(marker);
     marker.openBubble();
   }
 }
```

如承诺的那样，大部分代码都是由 Mapstraction 函数组成的。特定于 Google 的调用被分离出来。例如，我们创建一个地理编码器对象 ❶，然后调用以获取位置。甚至这个调用也包含一些 Mapstraction，因为我们使用它来获取地图的中心 ❷，并将该点转换为 Google 可以理解的点。

在调用地理编码器时，我们需要提供一个回调函数 ❸。这个函数在结果从 Google 返回时使用。因为我们创建了一个命名函数，所以我们还需要用那个名字创建该函数。

`found_address` 函数接受一个参数，即 Google 地理编码器发送给我们的结果对象。一旦我们确定我们有一个良好的响应 ❹（状态码为 `200`），我们就可以获取点 ❺，它包含我们的坐标。

你可能会想知道为什么这个点甚至有必要，因为这是你开始的地方的数据。在许多情况下，Google 无法在您的 *确切* 点找到地址（例如，想象一下一个大公园的中心），因此它会选择一个附近的地址。在这种情况下，您将想要知道它使用的点，以便相应地绘制。

Google 实际上可能会发送多个结果，这些结果将存储在 `response.Placemark` 数组中。第一个是它的最佳猜测，并且可能是应该使用的那个，尽管在某些情况下，您可能允许用户选择最准确的结果。

`found_address` 中剩余的大部分代码看起来很熟悉，因为它来自 第二章 的标准 Mapstraction 函数。我们在 Google 返回的位置放置一个标记。然后，我们使用最重要的信息，即地址 ❻，作为标记框内的信息。

为了更好地了解反向地理编码，尝试通过改变地图的中心来更改传递给 Google 的坐标。或者，继续阅读以创建一个点击任何地方都会进行反向地理编码的地图。

## 点击进行反向地理编码

想要尝试反向地理编码吗？尝试精确点击你的地址，看看你能多接近你的确切位置，这可以很有趣。此外，快速访问反向地理编码也可以是一个好的开发者工具，帮助你更好地了解这个过程。

你可以使用大多数之前的示例。实际上，`found_address`可以保持完全相同。将`create_map`函数替换为这个略微修改的版本：

```
function create_map() {
   mapstraction = new mxn.Mapstraction('mymap', 'googlev3');
❶  mapstraction.addSmallControls();
   mapstraction.setCenterAndZoom(
     new mxn.LatLonPoint(37.7740486,-122.4101883), 15);
❷  mapstraction.addEventListener('click', function(clickpoint) {
     // Google-specific calls
     var geocoder = new GclientGeocoder();
     geocoder.getLocations(❸clickpoint.toProprietary
(mapstraction.api), found_address);
   });
 }
```

我在地图上添加了一些缩放控件❶，这样你可以找到特定的点击位置（除非你更改地图的中心或滚动地图到另一个位置，否则是在旧金山）。然后我编写了一些代码，等待地图上的点击事件❷。当点击发生时，它调用一个匿名内联函数。这个函数可以被命名，但只有几行简单的代码，内联编写更容易。

在匿名函数中，我们调用谷歌地理编码器，与之前使用的非常相似。在这个例子中，我们传递用户点击的点❸而不是地图的中心。请注意，我们需要将点转换为专有的谷歌坐标类型，因为 Mapstraction 捕获点击点，然后需要将其传递给谷歌地理编码器。Mapstraction 说谷歌话，但谷歌不说 Mapstraction 话。

由于其他代码相同，点击地图会在打开的消息框中添加带有地址的标记。点击几次更多，地图上会出现额外的标记，包含反向地理编码器提供的地理信息。

你开始看到反向地理编码器的有用性了吗？在下一节中，你将能够使用谷歌的 HTTP 地理编码器在 JavaScript 之外访问这些数据。

## 使用谷歌网络服务进行反向地理编码

正如我在本章的其他地方提到的，力量和灵活性来自于能够控制地理编码器的输入和输出。你可以使用谷歌提供的反向地理编码服务获得相同的自由。

通过对 URL 进行微调，谷歌的地理编码器变成了反向地理编码器：

```
http://maps.google.com/maps/geo?q=`37.7740486,-122.4101883`&output=xml&sensor=false
```

我们仍然使用`q`查询参数，就像在第十三部分：使用 HTTP 网络服务进行地理编码中谷歌部分所做的那样。我们发送的是纬度和经度，按照这个顺序，用逗号分隔（在上面的 URL 中用粗体标出）。

结果几乎与使用正向地理编码器相同：

```
<?xml version="1.0" encoding="UTF-8" ?>
<kml >
  <Response>
    <name>37.7740486,-122.4101883</name>
    <Status>
      <code>200</code>
      <request>geocode</request>
    </Status>
    <tPlacemark id="p1">
      <address>38 Ringold St, San Francisco, CA 94103, USA</address>
      <AddressDetails Accuracy="8"
                      >
        ...
      </AddressDetails>
      <Point>
        <coordinates>-122.4102800,37.7741680,0</coordinates>
      </Point>
    </Placemark>
    <Placemark id="p2">
      ...
    </Placemark>
    ...
 </Response>
</kml>
```

最大的不同之处在于你可能会拥有多个地标，因为反向地理编码比标准地理编码要粗糙得多。否则，每个地标内接收到的内容都是相同的，包括邮政编码——当然，地址（或范围）也是，这是这个过程最初的目的。

你再也不必让用户尝试解读为计算机理解而设计的奇怪数字了。无论你选择 JavaScript 还是服务器端网络服务，你都可以通过快速调用反向地理编码器从坐标转换为文本。

# #15: 获取邮政编码坐标

你是否曾经访问过要求你输入 ZIP 代码以查找商店最近位置的网站？可能吧，我猜。本节将帮助你迈出创建类似功能的第一步。你需要一种将邮政编码转换为地理坐标的方法。

你可能认为大面积不能转换成一个地理点。你是对的，尽管对于任何地理编码结果倾向于街道附近点的地址来说，也可以这么说。那么你的后院呢？

记住，地理编码不是一门精确的科学。对于一个地址，选择一个有意义的点。对于邮政编码，最合理的点通常位于中心附近。然而，对于一些边界模糊的地方，确定中心点仍然很困难。因此，纬度和经度代表的是中心附近的一个点。

获取邮政编码坐标的最简单方法是通过地理编码服务进行搜索。例如，如果你想使用 Yahoo!地理编码器查找比弗利山最著名的 ZIP 代码，你会使用这个 URL：

```
http://local.yahooapis.com/MapsService/V1/geocode?appid=*`YOURKEY`*&location=90210
```

你的结果可能看起来像这样：

```
<?xml version="1.0"?>
<ResultSet ...>
  <Result precision="zip">
    <Latitude>34.092807</Latitude>
    <Longitude>-118.411115</Longitude>
    <Address />
    <City>Beverly Hills</City>
    <State>CA</State>
    <Zip>90210</Zip>
    <Country>US</Country>
  </Result>
</ResultSet>
```

注意，精度是 ZIP 级别的，地址标签为空。否则，结果与搜索完整地址时返回的结果相似。

## 安装邮政编码数据库

如果你需要执行大量的查找，或者想要更快地访问结果，拥有一个无需使用其他服务的数据库表来地理编码邮政编码是有意义的。美国有不到 50,000 个 ZIP 代码，这是一个相当小的记录数量，便于存储和访问。其他国家有更多独特的邮政编码（例如，加拿大有近一百万，这仍然足够小，值得这样做）。

你需要一个数据库来存储你的邮政编码及其对应的坐标。在第九章中，我描述了如何安装 MySQL 并从 CSV 文件导入数据。本书的网站包含可以免费下载邮政编码数据库的链接。请参阅[`mapscripting.com/postal-code-database`](http://mapscripting.com/postal-code-database)。

数据库中包含的字段可能会有所不同，但这里是一个美国 ZIP 代码数据库的示例结构：

| **ZIP 代码** 邮政编码 |
| --- |
| **名称** 该 ZIP 代码的文本描述，例如街区或城市名称 |
| **纬度** 坐标的南/北部分 |
| **经度** 坐标的东/西部分 |

一个非常基础的数据库甚至可能不包含名称字段，因为邮政代码数据库最重要的部分是将代码转换为点。

### 注意

你需要注意 ZIP 代码字段是存储为文本还是数字。有些人更喜欢文本，因为文本可以更好地表示以零开头的 ZIP 代码。然而，数据库能够更有效地搜索数字，所以你需要确保从用户输入中去除任何开头的零。

当你将完整的邮政编码数据库加载到`zipcoord`表（这是我取的名字）中时，查找比弗利山 90210 的坐标的 SQL 语句可能看起来像这样：

```
select latitude, longitude from zipcoord where zipcode='90210';
```

你应该只从数据库调用中获取一组坐标，因为只有一个 90210 的邮政编码。要了解更多关于使用 PHP 访问 SQL 结果的信息，请参阅第六十五部分：从 PHP 中使用 MySQL。

现在你已经可以获取邮政编码坐标了，你可以开始使用它们了。在这个项目的开始阶段，我提到了那些有搜索框以查找你 ZIP 代码附近位置的网站。将你的邮政编码结果与第四十六部分：从你的数据库中获取最近的位置结合，你将构建了一个商店定位器，就像你在那些网站上看到的那样。
