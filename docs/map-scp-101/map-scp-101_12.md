# 附录 B. MAPSTRACTION 参考

![无标题图片](img/httpatomoreillycomsourcenostarchimages671943.png.jpg)

Mapstraction 库使得创建与任何提供者兼容的地图变得容易。本附录提供了 Mapstraction 众多类和函数的详细信息。在您开发自己的地图时，请将其作为参考。

本附录按 Mapstraction 类组织——这是库本身分离的方式。在每个类中，我涵盖了其构造函数、其他函数以及任何类变量。这些类包括：

+   `mxn.Mapstraction`，用于创建和控制地图的主要类

+   `mxn.BoundingBox`，用于描述矩形区域（边界）

+   `mxn.LatLonPoint`，一个用于存储和引用单个坐标的类

+   `mxn.Marker`，用于维护单个兴趣点的数据

+   `mxn.Polyline`，一个用于存储连接点的集合（形状或线条）的类

+   `mxn.util`，一个包含有用函数的实用类

本节内容基于从 Mapstraction 源文件生成的官方文档。尽管我已经通过示例和更详细的描述扩展了大多数部分，但我还是要向维护文档的 Mapstraction 开发者社区表示衷心的感谢。

与所有 Mapstraction 事物一样，您可以在[`mapstraction.com`](http://mapstraction.com)找到最新的文档。

# 类 mxn.Mapstraction

这是主类；它是您使用 Mapstraction 时的第一个停靠点。您使用`mxn.Mapstraction`创建地图对象，然后通过该对象提供的主要接口来更改地图。

在本节中，您将找到构造函数，它创建一个新的地图对象。其他函数用于执行地图级别的操作，例如设置中心点和向地图添加图层。本类的大部分内容在第一章中有详细说明。

## 函数 mxn.Mapstraction

这是`Mapstraction`类的构造函数。该函数将一些 API 选择初始化到给定的 HTML 元素中。每个 Mapstraction 地图都将从这一行开始：

```
mxn.Mapstraction(*`element`*, *`api`*, *`debug`*)
```

### 参数

| **``*`element`*``** **(字符串)** 包含地图的 HTML DOM 元素或 HTML DOM 元素的`id`。 |
| --- |
| **``*`api`*``** **(字符串)** Mapstraction 使用的 API 名称。选项包括：`'cloudmade'`、`'geocommons'`、`'google'`、`'googlev3'`、`'map24'`、`'mapquest'`、`'microsoft'`、`'multimap'`、`'openlayers'`、`'openstreetmap'`和`'yahoo'`。 |
| **``*`debug`*``** **(布尔值)** 一个可选参数，用于开启调试支持。如果此参数为 true，Mapstraction 将使用警告面板来处理不支持的操作。这在开发期间可能很有用，但在生产中并不实用。默认为 false。 |

### 返回值

Mapstraction 对象，可用于调用`mxn.Mapstraction`字段和方法。在本节中称为`mxnobj`。

### 示例

```
var mxnobj = new mxn.Mapstraction('mymap', 'googlev3');
```

## 函数 addControls

此函数通过仅调用一次，向地图添加多个控件，如缩放和地图类型。您可以在唯一的参数对象中指定要添加的控件。

```
mxnobj.addControls(*`args`*)
```

### 参数

| **``*`args`*``** **(对象)** 要显示哪些地图控件。选项包括：`pan`，`zoom`（`'large'` 或 `'small'`），`overview`，`scale`，和`map_type`。除了`zoom`之外，所有选项都是布尔值（`true` 或 `false`）。 |
| --- |

### 示例

```
mxnobj.addControls({map_type: true, zoom: 'small'});
```

## 函数 addFilter

此函数添加一个标记过滤器，根据您创建的属性自动显示或隐藏标记。`addFilter` 函数准备过滤器并需要 `doFilter` 执行过滤。要了解更多关于 Mapstraction 的过滤选项，请参阅 #9: 过滤掉某些标记 在 #9: 过滤掉某些标记。

```
mxnobj.addFilter(*`name`*, *`operator`*, *`value`*)
```

### 参数

| **``*`name`*``** **(字符串)** 要放置过滤器的属性名称。通过 *`markerobject`*`.addAttribute` 添加。 |
| --- |
| **``*`operator`*``** **(字符串)** 用于比较属性与值的运算符。选项包括：`'ge'`（大于等于），`'le'`（小于等于），或`'eq'`（等于）。 |
| **``*`value`*``** **(数字，字符串)** 要比较的值。 |

### 示例

```
mxnobj.addFilter('age', 'ge', 21);
mxnobj.doFilter();
```

## 函数 addImageOverlay

此函数在给定位置在地图上添加一个地理参考图像。图形显示在地图图像之上，但在标记和多段线之下，因此可以用作替代或增强图像。

为了确保你的图像覆盖正确的区域，你需要对其进行 *校正*，如 #25: 在地图上叠加图像 在 #25: 在地图上叠加图像 中所示。

```
mxnobj.addImageOverlay(*`unique`*, *`url`*, *`opacity`*, *`west`*, *`south`*, *`east`*, *`north`*);
```

### 参数

| **``*`unique`*``** **(字符串)** 该覆盖层的唯一标识符，用作添加到地图中的新对象的 DOM `id`。 |
| --- |
| **``*`url`*``** **(字符串)** 要叠加的图像的文件位置。此 URL 可以是本地或远程的，文件可以是浏览器支持的任何格式。透明图形（如 PNG 文件）最适合增强（而不是替换）地图图像。 |
| **``*`opacity`*``** **(数字)** 一个介于 0（透明）和 100（完全不透明）之间的值，用于确定原始地图图像背后显示你图形的程度。 |
| **``*`west`*``** **(数字)** 图形边界框最西端的经度。 |
| **``*`south`*``** **(数字)** 图形边界框最南端的纬度。 |
| **``*`east`*``** **(数字)** 图形边界框最东端的经度。 |
| **``*`north`*``** **(数字)** 图形边界框最北端的纬度。 |

### 示例

```
mxnobj.addImageOverlay('centralpark', 'centralpark.png', 100,
                       −73.9867415, 40.7622753, −73.9460798, 40.8032834);
```

## 函数 addLargeControls

此函数向地图添加提供者的大缩放控件。在某些情况下，它可能还包括其他控件，如平移控件和比例尺。

```
mxnobj.addLargeControls()
```

## 函数 addMapTypeControls

此函数允许用户使用提供者的控件从可用的地图类型中选择。

```
mxnobj.addMapTypeControls()
```

## Function addMarker

此函数根据标记对象中的标准将标记对象（之前使用 `mxn.Marker` 创建）添加到地图中。标记的常见选项在第二章中进行了介绍。

```
mxnobj.addMarker(*`marker`*);
```

### Parameter

| **``*`marker`*``** (**`mxn.Marker`**) 要添加到地图中的标记。 |
| --- |

### Example

```
var mk = new mxn.Marker(new mxn.LatLonPoint(45, −122));
mxnobj.addMarker(mk);
```

## Function addMarkerWithData

此函数将一个标记对象（之前使用 `mxn.Marker` 创建）添加到地图中，并附带一系列选项。

```
mxnobj.addMarkerWithData(*`marker`*, *`options`*);
```

### Parameters

| **``*`marker`*``** (**`mxn.Marker`**) 要添加到地图中的标记。 |
| --- |
| **``*`options`*``** **(object)** 标记的选项哈希对象，包括 `draggable`、`groupName`、`hover`、`hoverIcon`、`infoBubble`、`icon`、`iconShadow`、`infoDiv`、`label` 和 `openBubble`。 |

### Example

```
var mk = new mxn.Marker(new mxn.LatLonPoint(45, −122));
var opt = {infoBubble: 'Message box content', draggable: false};
mxnobj.addMarkerWithData(mk, opt);
```

## Function addOverlay

此函数将 GeoRSS 或 KML 覆盖添加到地图中。这两种数据格式在第八章中进行了详细说明。

```
mxnobj.addOverlay(*`url`*, *`autoCenterAndZoom`*);
```

### Parameters

| **``*`url`*``** **(string)** GeoRSS 或 KML 文件的完整、公开 URL。 |
| --- |
| **``*`autoCenterAndZoom`*``** **(boolean)** 是否自动将地图中心对准覆盖内容。默认为 `false`。 |

### Example

```
mxnobj.addOverlay('http://mapscripting.com/example.kml');
```

## Function addPolyline

此函数根据多段线对象中的标准将多段线对象（之前使用 `mxn.Polyline` 创建）添加到地图中。多段线的常见选项在第四章中进行了介绍。

```
mxnobj.addPolyline(*`polyline`*);
```

### Parameter

| **``*`polyline`*``** (**`mxn.Polyline`**) 要添加到地图中的多段线。 |
| --- |

### Example

```
var poly = new mxn.Polyline(
 [new mxn.LatLonPoint(45, −122), new mxn.LatLonPoint(46, −121)]);
mxnobj.addPolyline(poly);
```

## Function addPolylineWithData

此函数将一个多段线对象（之前使用 `mxn.Polyline` 创建）添加到地图中，并附带一系列选项。

```
mxnobj.addPolylineWithData(*`polyline`*, *`options`*);
```

### Parameters

| **``*`polyline`*``** (**`mxn.Polyline`**) 要添加到地图中的多段线。 |
| --- |
| **``*`options`*``** **(array)** 多段线的选项哈希对象，包括 `closed`、`color`、`fillColor`、`opacity` 和 `width`。 |

### Example

```
var poly = new mxn.Polyline(
          [new mxn.LatLonPoint(45, −122), new mxn.LatLonPoint(46, −121)]);
var opt = {color: '#ffcc99', width: 3};
mxnobj.addPolylineWithData(poly, opt);
```

## Function addSmallControls

此函数将提供者的小型缩放控件添加到地图中。在某些情况下，它还可能包括其他控件的小型版本，例如平移。

```
mxnobj.addSmallControls()
```

## Function addTileLayer

此函数使用参数化 URL 将瓦片层添加到地图中。在第二十六部分：使用自定义瓦片中进行了详细说明。

```
addTileLayer(*`tile_url`*, *`opacity`*, *`copyright`*, *`min_zoom`*, *`max_zoom`*, *`map_type`*)
```

### Parameters

| **``*`tile_url`*``** **(string)** 瓦片的 URL 模板。需要参数化 URL，用于 `{X}` 和 `{Y}` 坐标以及缩放级别 `{Z}`。 |
| --- |
| **``*`opacity`*``** **(number)** 一个介于 0（透明）和 1（完全不透明）之间的十进制值，用于确定在您的瓦片层后面显示原始地图影像的程度。 |
| **``*`copyright`*``** **(string)** 包含在地图版权中的文本。 |
| **``*`min_zoom`*``** **(number)** 瓦片层可见的最小缩放级别。 |
| **``*`max_zoom`*``** **(数字)** 瓦片层可见的最大缩放级别。 |
| **``*`map_type`*``** **(布尔值)** 如果瓦片层是图层调色板中的可选地图类型，则为 `true`。默认为 `false`。 |

### 示例

```
mxnobj.addTileLayer('http://tile.openstreetmap.org/{Z}/{X}/{Y}.png',
                     1.0, "OSM", 1, 19, true);
```

## 函数 applyOptions

此函数应用当前选项设置。

```
mxnobj.applyOptions()
```

## 函数 autoCenterAndZoom

此函数将地图的中心和缩放设置为包含所有标记和多边形的最大边界框。

```
mxnobj.autoCenterAndZoom()
```

## 函数 centerAndZoomOnPoints

此函数将地图的中心和缩放设置为包含传递给函数的所有点的最小边界框。

```
mxnobj.centerAndZoomOnPoints(*`points`*);
```

### 参数

| **``*`points`*``** **(数组)** 用于确定地图的新中心和缩放的点（作为 `mxn.LatLonPoint` 对象）。 |
| --- |

### 示例

```
mxnobj.centerAndZoomOnPoints([new mxn.LatLonPoint(45, −122),
                              new mxn.LatLonPoint(46, −121)]);
```

## 函数 declutterMarkers

您可以使用此函数在地图上清理标记并将重叠的标记分组在一起。尽管如此，它并不广泛支持，但在 #11: 处理标记簇 中展示了另一种方法。

## 函数 doFilter

`doFilter` 函数根据使用 `addFilter` 函数先前创建的属性和标准隐藏或显示标记。有关 Mapstraction 的过滤选项的更多信息，请参阅 #9: 过滤掉某些标记。

```
mxnobj.doFilter()
```

### 示例

```
mxnobj.addFilter('age', 'ge', 21);
mxnobj.doFilter();
```

## 函数 getAttributeExtremes

此函数查找在标记属性中设置的最低和最高值。

```
mxnobj.getAttributeExtremes(attribute)
```

### 参数

| **``*`attribute`*``** **(字符串)** 标记属性的名称。 |
| --- |

### 返回值

包含传递的属性的最小和最大值的两个元素数组。

### 示例

```
var minmax = mxnobj.getAttributeExtremes('age'); // return [min, max]
```

## 函数 getBounds

此函数检索当前可视地图的 `BoundingBox`。

```
mxnobj.getBounds()
```

### 返回值

一个 `mxn.BoundingBox` 对象。

## 函数 getCenter

此函数检索当前可视地图的中心。

```
mxnobj.getCenter()
```

### 返回值

一个 `mxn.LatLonPoint` 对象。

## 函数 getMap

此函数检索当前地图的本地地图对象。如果需要执行专有地图调用，则非常有用。

```
mxnobj.getMap()
```

### 返回值

依赖于您所使用的提供者的本地对象。

## 函数 getMapType

此函数检索当前地图的图像类型，例如卫星或混合图像。

```
mxnobj.getMapType()
```

### 返回值

一个与地图类型相对应的数字。有关可用类型的说明，请参阅 设置地图类型。

## 函数 getZoom

此函数检索当前缩放级别。

```
mxnobj.getZoom()
```

### 返回值

与当前缩放级别相对应的整数。该主题在 设置缩放级别 中有详细说明。

## 函数 getZoomLevelForBoundingBox

此函数检索给定边界范围内的最佳缩放级别。

```
mxnobj.getZoomLevelForBoundingBox(*`bounds`*)
```

### 参数

| **``*`bounds`*``** (**`mxn.BoundingBox`**) 您想要找到最佳缩放级别的边界。 |
| --- |

## 函数 polylineCenterAndZoom

此函数将地图的中心和缩放设置为包含所有折线的最小边界框，忽略标记。

```
mxnobj.polylineCenterAndZoom()
```

## 函数 removeAllFilters

`removeAllFilters` 函数移除之前添加的所有过滤器，但不会显示之前过滤的标记（为此您需要再次调用 `doFilter` 函数）。因为过滤器是累加的，所以此函数在从一种过滤方法切换到另一种过滤方法时非常有用。

```
mxnobj.removeAllFilters()
```

## 函数 removeAllMarkers

此函数永久地从地图中移除所有标记。要临时移除，请在 `mxn.Marker` 中使用 `hide` 函数。

```
mxnobj.removeAllMarkers()
```

## 函数 removeAllPolylines

此函数永久地从地图中移除所有折线。要临时移除，请在 `mxn.Polyline` 中使用 `hide` 函数。

```
mxnobj.removeAllPolylines()
```

## 函数 removeFilter

此函数移除之前添加的过滤器，但不会显示之前由移除的过滤器过滤的标记（为此您需要再次调用 `doFilter` 函数）。

```
mxnobj.removeFilter(*`name`*, *`operator`*, *`value`*)
```

### 参数

| **``*`name`*``** **(字符串)** 要移除过滤器的属性名称。 |
| --- |
| **``*`operator`*``** **(字符串)** 之前用于比较属性和值的运算符。 |
| **``*`value`*``** **(数字，字符串)** 要比较的值。可以是数字或文本。 |

### 示例

```
mxnobj.removeFilter('age', 'ge', 21);
mxnobj.doFilter();
```

## 函数 removeMarker

此函数从地图中移除标记对象（之前使用 `mxn.Marker` 创建）。

```
mxnobj.removeMarker(*`marker`*);
```

### 参数

| **``*`marker`*``** (**`mxn.Marker`**) 要从地图中移除的标记。 |
| --- |

### 示例

```
mxnobj.removeMarker(mxnobj.markers[0]); // removes first marker
```

## 函数 removePolyline

此函数从地图中移除一个折线对象（之前使用 `mxn.Polyline` 创建），并永久删除。

```
mxnobj.removePolyline(*`polyline`*);
```

### 参数

| **``*`polyline`*``** (**`mxn.Polyline`**) 要从地图中移除的折线。 |
| --- |

### 示例

```
mxnobj.removePolyline(mxnobj.polylines[0]); // removes first polyline
```

## 函数 resizeTo

此函数通过 CSS 调整地图对象的大小到提供的尺寸。

```
mxnobj.resizeTo(*`width`*, *`height`*)
```

### 参数

| **``*`width`*``** **(数字)** 地图的新宽度。 |
| --- |
| **``*`height`*``** **(数字)** 地图的新高度。 |

### 示例

```
mxnobj.resizeTo(800, 600); // 800 pixels wide, 600 pixels tall
```

## 函数 setBounds

此函数根据 `BoundingBox` 设置地图到适当的中心和缩放。

```
mxnobj.setBounds(*`bounds`*)
```

### 参数

| **``*`bounds`*``** (**`mxn.BoundingBox`**) 用于确定地图新中心和缩放级别的 `BoundingBox`。 |
| --- |

## 函数 setCenter

此函数根据提供的点设置地图的中心。

```
mxnobj.setCenter(*`point`*)
```

### 参数

| **``*`point`*``** (**`mxn.LatLonPoint`**) 要用作地图新中心的点。 |
| --- |

### 示例

```
mxnobj.setCenter(new mxn.LatLonPoint(45, −122));
```

## 函数 setCenterAndZoom

此函数根据提供的点和缩放级别设置地图的中心和缩放级别。

```
mxnobj.setCenterAndZoom(*`point`*, *`zoom`*)
```

### 参数

| **``*`point`*``** (**`mxn.LatLonPoint`**) 要用作地图新中心的点。 |
| --- |
| **``*`zoom`*``** **(数字)** 要用于地图的新缩放级别。缩放级别在设置缩放级别中有详细说明。 |

### 示例

```
mxnobj.setCenterAndZoom(new mxn.LatLonPoint(45, −122), 10);
```

## 函数 setDebug

此函数确定是否开启调试支持。当开启调试支持时，Mapstraction 将使用警告面板来处理不支持的操作。

```
mxnobj.setDebug(*`debug`*)
```

### Parameter

| **``*`debug`*``** **(boolean)** 如果此参数为 `true`，则开启调试支持。这在开发期间可能很有用，但在生产环境中则没有用。 |
| --- |

## Function setMapType

此函数声明当前地图的图像类型，例如卫星图或混合图。

```
mxnobj.setMapType(*`type`*)
```

### Parameter

| **``*`type`*``** **(integer)** 对应地图类型的数字。有关可用类型的描述，请参阅设置地图类型。 |
| --- |

### Example

```
mxnobj.setMapType(mxn.Mapstraction.SATELLITE);
```

## Function setOption

此函数设置单个选项，例如滚轮缩放或可拖动地图。

```
mxnobj.setOption(*`name`*, *`value`*)
```

### Parameters

| **``*`name`*``** **(string)** 要设置的选项的名称。目前支持选项：`'enableScrollWheelZoom'` 和 `'enableDragging'`。 |
| --- |
| **``*`value`*``** **(boolean)** 是否设置了选项（`true` 或 `false`）。 |

### Example

```
mxnobj.setOption('enableDragging', false); // map cannot be dragged
```

## Function setOptions

此函数使用对象一次性声明多个选项，例如滚轮缩放或可拖动地图。

```
mxnobj.setOptions(*`opt`*)
```

### Parameter

| **``*`opt`*``** **(object)** 包含要设置选项的名称和值的哈希对象。目前支持选项：`'enableScrollWheelZoom'` 和 `'enableDragging'`。 |
| --- |

### Example

```
mxnobj.setOptions({'enableDragging': false, 'enableScrollWheelZoom': true);
```

## Function setZoom

此函数设置当前缩放级别。

```
mxnobj.setZoom(*`zoom`*)
```

### Parameter

| **``*`zoom`*``** **(number)** 要设置的缩放级别的数字。该主题在设置缩放级别中有详细说明。 |
| --- |

### Example

```
mxnobj.setZoom(13); // Roughly city-level zoom
```

## Function swap

`swap` 函数隐藏之前的地图，并实时将活动地图更改为不同的 API 提供商。

```
mxnobj.swap(*`api`*, *`element`*)
```

### Parameters

| **``*`api`*``** **(string)** Mapstraction 用于新地图的 API 名称。选项包括：`'cloudmade'`、`'geocommons'`、`'google'`、`'googlev3'`、`'map24'`、`'mapquest'`、`'microsoft'`、`'multimap'`、`'openlayers'`、`'openstreetmap'` 和 `'yahoo'`。 |
| --- |
| **``*`element`*``** **(string, DOM element)** 包含新地图的 HTML 元素的 `id`，或元素本身。 |

### Example

```
mxnobj.swap('yahoo', 'secondmapdiv');
```

## Function toggleFilter

如果存在标记过滤器，此函数将移除它。否则，它将添加一个标记过滤器，根据您创建的属性自动显示或隐藏标记。它仍然需要 `doFilter` 函数来执行过滤。有关 Mapstraction 的过滤选项的更多信息，请参阅第九部分：过滤特定标记。 

```
mxnobj.toggleFilter(*`name`*, *`operator`*, *`value`*)
```

### Parameters

| **``*`name`*``** **(string)** 要放置过滤器的属性的名称。通过 *`markerobject`*`.addAttribute` 添加。 |
| --- |
| **``*`operator`*``** **(string)** 用于比较属性和值的运算符。选项：`'ge'`（大于等于）、`'le'`（小于等于）或 `'eq'`（等于）。 |
| **``*`value`*``** **(数字，字符串)** 要比较的值。可以是数字或文本（仅 `'eq'`）。 |

### 示例

```
mxnobj.toggleFilter('age', 'ge', 21);
mxnobj.doFilter();
```

## 函数 toggleTileLayer

`toggleTileLayer` 函数控制先前添加的瓦片层的可见性。如果瓦片层是可见的，此函数将使其不可见（反之亦然）。

```
mxnobj.toggleTileLayer(*`tile_url`*)
```

### 参数

| **``*`tile_url`*``** **(字符串)** 在 `addTileLayer` 函数中使用的确切 URL。 |
| --- |

### 示例

```
mxnobj.toggleTileLayer('http://tile.openstreetmap.org/{Z}/{X}/{Y}.png');
```

## 函数 visibleCenterAndZoom

此函数将地图的中心和缩放设置为包含所有 *可见* 标记和多段线的最小边界框。忽略已隐藏的标记或多段线。

```
mxnobj.visibleCenterAndZoom()
```

## mxn.Mapstraction 类中的变量

这些变量可以通过 Mapstraction 对象直接访问。例如，您可以使用 *`mxnobj`*`.markers` 变量遍历添加到地图上的所有标记，如第七部分：遍历所有标记中所示。

可用的变量有：

| **``*`api`*``** **(字符串)** 此 Mapstraction 对象的当前活动 API 名称。此值与 `mxn.Mapstraction` 构造函数中传递的 `api` 参数相同。 |
| --- |
| **``*`CurrentElement`*``** **(对象)** 包含地图的 DOM 元素。Mapstraction 将此值设置为 `document.getElementById(`*`mxnobj`*`.element)`。 |
| **``*`element`*``** **(字符串)** 将作为 `element` 参数传递给 `mxn.Mapstraction` 构造函数的原始元素 `id`。 |
| **``*`markers`*``** **(数组)** 包含当前地图上所有已加载的标记（包括隐藏的）。 |
| **``*`options`*``** **(对象)** 当前使用 `setOptions` 函数设置的地图选项。 |
| **``*`polylines`*``** **(数组)** 包含当前地图上所有已加载的多段线（包括隐藏的）。 |

# 类 mxn.BoundingBox

此类创建一个 `BoundingBox` 并提供访问框信息的方法。`BoundingBox` 对象用于描述由两个经纬度点定义的区域，如第十九部分：绘制矩形以声明区域中所述。

Mapstraction 还内部使用 `BoundingBox` 类来自动定位地图或检索当前地图的边界。此外，您还可以使用 `BoundingBox` 对象执行简单的碰撞测试（参见其他有用参数中的第七部分：遍历所有标记）。

## 函数 mxn.BoundingBox

这是 `BoundingBox` 类的构造函数。使用此函数根据西南角和东北角的坐标创建一个新的 `BoundingBox` 对象。

```
mxn.BoundingBox(*`swlat`*, *`swlon`*, *`nelat`*, *`nelon`*);
```

### 参数

| **``*`swlat`*``** **(数字)** 西南角的纬度。 |
| --- |
| **``*`swlon`*``** **(number)** 西南点的经度。 |
| **``*`nelat`*``** **(number)** 东北点的纬度。 |
| **``*`nelon`*``** **(number)** 东北点的经度。 |

### 返回

一个新的`BoundingBox`对象，可以用来调用`mxn.BoundingBox`函数。在本节中称为`bbobj`。

### 示例

```
var bbobj = new mxn.BoundingBox(45, −122, 46, −121);
```

## 函数包含

此函数用于执行简单的碰撞测试。也就是说，它查找给定点是否在`BoundingBox`内。

```
bbobj.contains(*`point`*)
```

### 参数

| **``*`point`*``** (**`LatLonPoint`**) 要测试的点。 |
| --- |

### 返回

一个布尔值（`true`或`false`），描述点是否在边界框内。

### 示例

```
var pt = new mxn.LatLonPoint(45.5, −121.5);
var hittest = bbobj.contains(pt);
```

## 扩展函数

如果一个点不在边界内，`extend`函数将扩展`BoundingBox`以包含该点。

```
bbobj.extend(*`point`*)
```

### 参数

| **``*`point`*``** (**`LatLonPoint`**) 要包含在边界内的新点。 |
| --- |

### 示例

```
var pt = new mxn.LatLonPoint(45.5, −121.5);
bbobj.extend(pt);
```

## 获取东北点函数

此函数检索边界的最东北点。

```
bbobj.getNorthEast()
```

### 返回

一个`mxn.LatLonPoint`对象。

## 获取西南点函数

此函数检索边界的最西南点。

```
bbobj.getSouthWest()
```

### 返回

一个`mxn.LatLonPoint`对象。

## 函数 isEmpty

此函数检查边界是否有零面积——也就是说，西南点和东北点是否相同。

```
bbobj.isEmpty()
```

### 返回

一个布尔值（如果为空则为`true`，如果边界有面积则为`false`）。

# `mxn.LatLonPoint`类

没有这个类，你无法在地图上放置很多东西。一个`mxn.LatLonPoint`对象描述地球上的一个点，并提供了一些用于使用数据的函数，例如在第 36 号：计算两点之间的距离中所示。

通常，`mxn.LatLonPoint`是一个实用程序。其他每个类都依赖于它来保存纬度和经度坐标。

## `mxn.LatLonPoint`函数

这是`mxn.LatLonPoint`类的构造函数。使用它来创建新的`LatLonPoint`对象，你可以将其作为地图的中心、标记的位置等传递。

```
mxn.LatLonPoint(*`lat`*, *`lon`*)
```

### 参数

| **``*`lat`*``** **(number)** 你正在创建的点的纬度。 |
| --- |
| **``*`lon`*``** **(number)** 你正在创建的点的经度。 |

### 返回

一个新的`LatLonPoint`对象，可以用来调用`mxn.LatLonPoint`函数。在本节中称为`llobj`。

### 示例

```
var llobj = new mxn.LatLonPoint(45, −122);
```

## 距离函数

此函数计算当前`LatLonPoint`与第二个`LatLonPoint`之间的距离，单位为公里。

```
llobj.distance(*`point`*)
```

### 参数

| **``*`point`*``** (**`LatLonPoint`**) 第二个`LatLonPoint`对象。 |
| --- |

### 返回

一个十进制数字，表示`llobj LatLonPoint`和参数之间的公里数。

### 示例

```
var llobj = new mxn.LatLonPoint(45, −122);
var newpt = new mxn.LatLonPoint(44, −121);
var kms = llobj.distance(newpt); // 136.578 km
```

## 函数 equals

`equals`函数测试此点是否与第二个`LatLonPoint`相同——纬度和经度是否都相等。

```
llobj.equals(*`point`*)
```

### 参数

| **``*`point`*``** (**`LatLonPoint`**) 第二个 `LatLonPoint` 对象。 |
| --- |

### 返回值

一个布尔值（`true` 或 `false`），回答点是否相同。

### 示例

```
var llobj = new mxn.LatLonPoint(45, −122);
var newpt = new mxn.LatLonPoint(44, −121);
var same = llobj.equals(newpt); // false
```

## 函数 latConv

此函数提供纬度转换——基于当前投影的一个纬度度数的距离。它在 Mapstraction 中用于诸如搜索半径等功能（参见在“设置填充颜色和透明度”中添加圆圈以显示搜索半径 设置填充颜色和透明度）。

```
llobj.latConv()
```

## 函数 lonConv

此函数类似于 `latConv`，但它返回基于当前投影的经度转换——一个经度度数的距离。

```
llobj.lonConv()
```

## 函数 toString

此函数给出点的文本表示。

```
llobj.toString()
```

### 返回值

一个显示包含在 `LatLonPoint` 中的纬度和经度的字符串（如 `45, −122`）。

# 类 mxn.Marker

虽然 `mxn.LatLonPoint` 声明点，但 `mxn.Marker` 是在地图上显示点的最常见方式。使用此类创建新的标记图钉，包括自定义图标图形。标记在第二章中有详细说明。

## 函数 mxn.Marker

这是 `mxn.Marker` 类的构造函数。使用它来创建新的标记，然后设置其选项。然后使用 `mxn.Mapstraction.addMarker` 函数将其添加到地图上。

```
mxn.Marker(point)
```

### 参数

| **``*`point`*``** (**`LatLonPoint`**) 在地图上放置标记的点。 |
| --- |

### 返回值

一个 `mxn.Marker` 对象，在本节中称为 `mkobj`。

### 示例

```
var mkobj = new mxn.Marker(new mxn.LatLonPoint(45, −122));
```

## 函数 addData

使用一个便捷的函数向标记添加数据，例如自定义图标图形。`addData` 函数类似于 `mxn.Mapstraction.addMarkerWithData` 函数。

```
mkobj.addData(*`options`*)
```

### 参数

| **``*`options`*``** **(array)** 标记的选项哈希对象，包括 `draggable`、`groupName`、`hover`、`hoverIcon`、`infoBubble`、`icon`、`iconShadow`、`infoDiv`、`label` 和 `openBubble`。 |
| --- |

### 示例

```
var opt = {infoBubble: 'Message box content', draggable: false};
mkobj.addData(opt);
```

## 函数 getAttribute

此函数从标记中检索特定属性的值。属性通常用于过滤，并包含有关特定标记的数据。

```
mkobj.getAttribute(*`name`*)
```

### 参数

| **``*`name`*``** **(string)** 你想要检索的属性名称。 |
| --- |

### 返回值

属性的值。

### 示例

```
var num = mkobj.getAttribute('age');
```

## 函数 setAttribute

这是 `getAttribute` 的另一面，此函数设置标记中特定属性的值。

```
mkobj.setAttribute(*`name`*, *`value`*)
```

### 参数

| **``*`name`*``** **(string)** 你想要设置的属性名称。 |
| --- |
| **``*`value`*``** **(varies)** 你想要设置的属性的值。 |

### 示例

```
mkobj.setAttribute('age', 21);
mkobj.setAttribute('address', '123 Main St');
```

## 函数 setDraggable

此函数声明标记是否可拖动——如果提供者支持此功能，则用户可以将其从一处移动到另一处。

```
mkobj.setDraggable(*`draggable`*)
```

### 参数

| **``*`draggable`*``** **(boolean)** 如果标记应该可拖动（否则为 `false`），则设置为 `true`。 |
| --- |

## 函数 setHover

此函数确定当鼠标悬停在标记上时，标记的消息框是否应该显示。

```
mkobj.setHover(*`hover`*)
```

### 参数

| **``*`hover`*``** **(布尔值)** 如果你想设置一个可悬停的标记，则设置为 `true`。否则，设置为 `false`。 |
| --- |

## 函数 setHoverIcon

此函数添加一个特殊图标，当鼠标悬停在标记上时显示。悬停图标继承主图标的大小。

```
mkobj.setHoverIcon(*`iconurl`*)
```

### 参数

| **``*`iconurl`*``** **(字符串)** 使用作为悬停图标的图形的文件名或完整 URL。 |
| --- |

### 示例

```
mkobj.setHoverIcon('highlighted.png');
```

## 函数 setIcon

此函数向标记添加自定义图标。它包含在 #5: 创建自定义图标标记 中，在 #4: 不点击标记显示和隐藏消息框。

```
mkobj.setIcon(*`iconurl`*, *`iconSize`*, *`iconAnchor`*)
```

### 参数

| **``*`iconurl`*``** **(字符串)** 使用作为图标的图形的文件名或完整 URL。 |
| --- |
| **``*`iconSize`*``** **(数组)** 包含图标宽度和高度（以像素为单位）的数组。 |
| **``*`iconAnchor`*``** **(数组)** 一个可选参数，用于描述图形指向地图的位置。该数组包含从图形的左上角到图形左下角的像素数。 |

### 示例

```
mkobj.setIcon('pin.png', [24, 36], [12, 36]);
```

## 函数 setIconAnchor

此函数直接设置图标的锚点，即图标指向地图的位置。

```
mkobj.setIconAnchor(*`iconAnchor`*)
```

### 参数

| **``*`iconAnchor`*``** **(数组)** 包含从图形的左上角到图形左下角的像素数。 |
| --- |

### 示例

```
mkobj.setIconAnchor([12, 36]);
```

## 函数 setIconSize

此函数直接设置图标的尺寸。

```
mkobj.setIconSize(*`iconSize`*)
```

### 参数

| **``*`iconSize`*``** **(数组)** 包含图标宽度和高度（以像素为单位）的数组。 |
| --- |

### 示例

```
mkobj.setIconSize([24, 36]);
```

## 函数 setInfoBubble

此函数声明当标记被点击（或如果启用了悬停选项，则鼠标悬停）时，在标记内显示的文本。大多数提供商允许此文本为 HTML。

```
mkobj.setInfoBubble(*`text`*)
```

### 参数

| **``*`text`*``** **(字符串)** 在消息框内显示的文本或 HTML。 |
| --- |

### 示例

```
mkobj.setInfoBubble('<em>This</em> is my message!');
```

## 函数 setInfoDiv

此函数声明当标记被点击时，在地图外部的 `div` 中显示的文本。

```
mkobj.setInfoDiv(*`text`*, *`div`*)
```

### 参数

| **``*`text`*``** **(字符串)** 在 `div` 内显示的文本或 HTML。 |
| --- |
| **``*`div`*``** **(字符串)** 显示消息的 `div` 的 `id`。 |

### 示例

```
mkobj.setInfoDiv('<em>This</em> is my message!', 'msgbox');
```

## 函数 setLabel

此函数声明当用户将鼠标悬停在标记上时，在标记上方显示的简短文本。

```
mkobj.setLabel(*`text`*)
```

### 参数

| **``*`text`*``** **(字符串)** 在悬停时显示的文本。 |
| --- |

### 示例

```
mkobj.setLabel('Name of Location');
```

## 函数 setShadowIcon

此函数在标记图标下方添加阴影图标（如果提供商支持），作为阴影图标。

```
mkobj.setShadowIcon(*`iconurl`*, *`iconSize`*)
```

### 参数

| **``*`iconurl`*``** **(字符串)** 使用作为阴影图标的图形的文件名或完整 URL。 |
| --- |
| **``*`iconSize`*``** **(数组)** 包含阴影图标宽度和高度（以像素为单位）的数组。 |

### 示例

```
mkobj.setShadowIcon('pin-shadow.png', [36, 42]);
```

# 类 mxn.Polyline

当一个单独的点不能描述你的数据时，转向`mxn.Polyline`类来描述线和形状。最常见的用途是描述路线，但在第四章中给出了许多其他示例。

## 函数 mxn.Polyline

这是`mxn.Polyline`类的构造函数。使用此函数创建一个新的折线，然后设置其选项。然后使用`mxn.Mapstraction.addPolyline`函数将其添加到地图上。

```
mxn.Polyline(*`points`*)
```

### 参数

| **``*`points`*``** **(数组)** 描述折线的`mxn.LatLonPoint`对象。 |
| --- |

### 返回值

一个`mxn.Polyline`对象，在本节中称为`pobj`。

### 示例

```
var pobj = new mxn.Polyline(
              [new mxn.LatLonPoint(45, −122), new mxn.LatLonPoint(44, −121)]);
```

## 函数添加数据

在一个方便的函数中添加数据，如线条颜色和透明度。`addData`函数类似于`mxn.Mapstraction.addPolylineWithData`函数。

```
pobj.addData(*`options`*)
```

### 参数

| **``*`options`*``** **(对象)** 一个包含折线选项的哈希对象，包括`closed`、`color`、`fillColor`、`opacity`和`width`。 |
| --- |

### 示例

```
var opt = {color: '#ffcc99', width: 3};
pobj.addData(opt);
```

## 函数设置闭合

此函数声明折线是否意味着要成为一个多边形——一个完整的形状。

```
pobj.setClosed(closed)
```

### 参数

| **``*`closed`*``** **(布尔值)** 如果折线表示一个完整的形状，设置为`true`。否则，`false`。 |
| --- |

## 函数设置颜色

此函数直接声明线的颜色。它不影响多边形内部的颜色（如果`Polyline`是闭合的）。

```
pobj.setColor(*`hex`*)
```

### 参数

| **``*`hex`*``** **(字符串)** 颜色值的十六进制表示，包括前面的`#`。 |
| --- |

### 示例

```
pobj.setColor('#ff0000'); // red
```

## 函数设置填充颜色

此函数直接声明多边形的内部颜色。它不影响未闭合的`Polyline`。

```
pobj.setFillColor(*`hex`*)
```

### 参数

| **``*`hex`*``** **(字符串)** 颜色值的十六进制表示，包括前面的`#`。 |
| --- |

### 示例

```
pobj.setFillColor('#0000ff'); // blue
```

## 函数设置宽度

此函数直接声明折线（或多边形边框）的宽度。

```
pobj.setWidth(*`pixels`*)
```

### 参数

| **``*`pixels`*``** **(数字)** 宽度（以像素为单位）。 |
| --- |

### 示例

```
pobj.setWidth(4);
```

## 函数简化

`simplify`函数减少了折线中的点数，基于你提供的容差删除那些靠近其他点的点。如果你在宽缩放级别显示一条大线，简化点是好主意。

```
pobj.simplify(*`tolerance`*)
```

### 参数

| **``*`tolerance`*``** **(数字)** 一个点需要与上一个点保持的距离（以公里为单位），以便包含在简化的折线中。 |
| --- |

### 示例

```
pobj.simplify(1.5);
```

# 命名空间 mxn.util

Mapstraction 有几个函数不适合放入之前的任何类中。这些实用函数位于`mxn.util`命名空间中。与类不同，`mxn.util`在使用其函数之前不需要用构造函数初始化。

## 函数 mxn.util.$m

`$m`函数类似于 jQuery 的`$`函数，但它仅用于通过`id`查找 HTML 元素。这是一个更优雅的`document.getElementById`函数。

```
mxn.util.$m(*`id [, id]*`*)
```

### 参数

| **``*`id`*``** **(字符串)** 元素的标识符。与 jQuery 不同，此参数不需要前面的`#`CSS 语法。可以传递多个`id`参数。 |
| --- |

### 返回

与元素`id`对应的 HTML 对象。如果提供了多个`id`参数，则返回匹配的元素数组。

### 示例

```
var mapobj = mxn.util.$m('mymap');
```

## 函数 mxn.util.getAvailableProviders

此函数检索当前加载的所有提供者。大多数时候，你只会使用一个提供者，但如果进行复杂的多映射，此结果很有用。

```
mxn.util.getAvailableProviders()
```

### 返回

包含提供者标识符的数组。

### 示例

```
var loaded = mxn.util.getAvailableProviders();
```

## 函数 mxn.util.getStyle

此函数检索 CSS 样式信息。由于不是所有浏览器都以相同的方式访问样式，Mapstraction 提供了这个跨浏览器函数。

```
mxn.util.getStyle(*`htmlobj`*, *`style`*)
```

### 参数

| **``*`htmlobj`*``** **(对象)** HTML 元素的对象版本，例如`mxn.util.$m`返回的值。 |
| --- |
| **``*`style`*``** **(字符串)** 要检索的样式名称。 |

### 返回

包含样式信息的字符串。

### 示例

```
var this_style = mxn.util.getStyle(mxn.util.$m('mymap'), 'border');
```

## 函数 mxn.util.KMToMiles

当然，你可以自己进行数学计算，但 Mapstraction 提供了几个函数来在测量系统之间进行转换。此函数将千米转换为英里。

```
mxn.util.KMToMiles(*`km`*)
```

### 参数

| **``*`km`*``** **(数字)** 千米数。 |
| --- |

### 返回

英里数。

## 函数 mxn.util.loadScript

此函数加载 JavaScript 文件。这用于内部读取适当的 Mapstraction 文件以供提供者使用，但你也可以使用它。

```
mxn.util.loadScript(*`scripturl`*)
```

### 参数

| **``*`scripturl`*``** **(字符串)** 脚本文件的地址，可以是单个文件或完整 URL。 |
| --- |

## 函数 mxn.util.loadStyle

此函数加载 CSS 样式表。

```
mxn.util.loadStyle(*`styleurl`*)
```

### 参数

| **``*`styleurl`*``** **(字符串)** 样式表的地址，可以是单个文件或完整 URL。 |
| --- |

## 函数 mxn.util.lonToMetres

地球的曲率使得经度一度长度的变化取决于位置的纬度。此函数将经度转换为海平面上的米。注意函数名称的英国拼写。

通过传递纬度距离作为*`lon`*参数和 0 作为*`lat`*参数，可以实现纬度到米的转换。

```
mxn.util.lonToMetres(*`lon`*, *`lat`*)
```

### 参数

| **``*`lon`*``** **(数字)** 要转换为米的经度度数。 |
| --- |
| **``*`lat`*``** **(数字)** 位置的纬度。 |

### 返回

在指定纬度下，指定经度度数的米数。

## 函数 mxn.util.metresToLon

这是`mxn.util.lonToMetres`函数的逆过程。它将米转换为经度度数。

纬度度数与纬度为 0 时的经度度数相同，因此可以将`lat`参数的值设为 0 以转换纬度的米。

```
mxn.util.metresToLon(*`meters`*, *`lat`*)
```

### 参数

| **``*`meters`*``** **(数字)** 要转换的米数。 |
| --- |
| **``*`lat`*``** **(数字)** 位置的纬度。 |

### 返回

经度度数。

## 函数 mxn.util.milesToKM

这是 `mxn.util.KMToMiles` 的逆操作。此函数将英里转换为千米。

```
mxn.util.milesToKM(*`miles`*)
```

### 参数

| **``*`miles`*``** **(数字)** 要转换的英里数。 |
| --- |

### 返回值

千米数。

## 函数 mxn.util.stringFormat

在 JavaScript 中将变量与文本混合连接可能会很麻烦，所以 Mapstraction 提供了这个函数来简化这个过程。传递一个参数化字符串以及一些变量，Mapstraction 就会创建一个单独的文本字符串。

```
mxn.util.stringFormat(*`text`*, var1, var2, ...)
```

### 参数

| **``*`text`*``** **(字符串)** 完整的文本，其中包含用 `{` 和 `}` 括号表示的特殊编号参数。 |
| --- |
| **``*`var1, var2, ...`*``** **(可变)** 任意数量的变量或值，与 `text` 变量中参数化值的数量相匹配。 |

### 返回值

完整的文本字符串，其中插入了变量/值。

### 示例

```
var message = mxn.util.stringFormat('There are {0} tiny {1}', num, 'eggs');
```
