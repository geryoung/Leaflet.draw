翻译内容：



| 英文 | 中文 
| -- | -- 
| set the view |  
| draw toolbar | 绘图工具栏 
| marker | 标记 
| control | 控件 
| view | 区域 
| Leaflet.draw plugin | 插件 




# 重要
Leaflet.draw 0.2.3+ 需要 [Leaflet 0.7](https://github.com/Leaflet/Leaflet/archive/v0.7.zip) 或者更高版本。

#Leaflet.draw
在 [Leaflet maps](https://github.com/Leaflet/Leaflet) 增加对绘制和编辑矢量和标记物的编辑 ，查看示例 [demo](http://leaflet.github.com/Leaflet.draw/).

#### 从 Leaflet.draw 0.1 升级

Leaflet.draw 0.2.0 与 0.1 相比变化了很多。 请查看 [BREAKING CHANGES](https://github.com/Leaflet/Leaflet.draw/blob/master/BREAKINGCHANGES.md) 来获取升级的方法。


## 主要内容

[使用插件](#using)  
[高级选项](#options)  
[常见功能](#commontasks)  
[感谢](#thanks)

<a name="using" />
## 使用插件

绘图工具栏控件的默认状态是在缩放控件的下方. 绘图工具栏控件可以让用户绘制矢量，在地图上标记. **请注意：编辑工具栏默认是不开启的.**

通过设置地图选项的 `drawControl: true` 来将绘图工具栏添加到地图上。

````js
// 在 "map" div 标签中创建一个地图，通过给定的位置和缩放来设置区域

var map = L.map('map', {drawControl: true}).setView([51.505, -0.09], 13);

// 添加一层 OpenStreetMap 的瓦片层
L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);
````

### 添加编辑工具栏
使用编辑控件前，必须初始化 Leaflet.draw 控件并手动添加到地图

````js
// 在 "map" div 标签中创建一个地图，通过给定的位置和缩放来设置区域
var map = L.map('map').setView([51.505, -0.09], 13);

// 添加一层 OpenStreetMap 的瓦片层
L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

//初始化 FeatureGroup 来存储可编辑到涂层
var drawnItems = new L.FeatureGroup();
map.addLayer(drawnItems);

//初始化 绘图控件并将它作为 FeatureGroup 到可编辑图层
// Initialise the draw control and pass it the FeatureGroup of editable layers
var drawControl = new L.Control.Draw({
	edit: {
		featureGroup: drawnItems
	}
});
map.addControl(drawControl);
````
这里到关键点是 `featureGroup` 选项。这将告诉插件 `FeatureGroup` 包含的哪些图层应该被编辑。
The key here is the `featureGroup` option. This tells the plugin which `FeatureGroup` contains the layers that should be editable.

### 事件
一旦成功将 Leaflet.draw 插件添加到地图上，我们会想要响应用户可以发起到动作。下面提到到事件将会在地图上被触发：

#### draw:created

| Property | 类型 | 描述
| --- | --- | ---
| layer | [Polyline](http://leafletjs.com/reference.html#polyline)/[Polygon](http://leafletjs.com/reference.html#polygon)/[Rectangle](http://leafletjs.com/reference.html#rectangle)/[Circle](http://leafletjs.com/reference.html#circle)/[Marker](http://leafletjs.com/reference.html#marker) | Layer that was just created.
| layerType | String | 这种类型的图层是  `polyline`, `polygon`, `rectangle`, `circle`, `marker` 其中的一种

当一个新的矢量或标记被创建时。

````js
map.on('draw:created', function (e) {
	var type = e.layerType,
		layer = e.layer;

	if (type === 'marker') {
		// Do marker specific actions
	}

	// Do whatever else you need to. (save to db, add to map etc)
	map.addLayer(layer);
});
````

#### draw:edited

| Property | 类型 | 描述
| --- | --- | ---
| layers | [LayerGroup](http://leafletjs.com/reference.html#layergroup) | 列出地图上所有被编辑的图层

当图层在 FeatureGroup 中初始化或者被编辑或者被保存时触发。

````js
map.on('draw:edited', function (e) {
	var layers = e.layers;
	layers.eachLayer(function (layer) {
		//do whatever you want, most likely save back to db
	});
});
````

#### draw:deleted

当图层在 FeatureGroup 被移除并被保存时触发。

| Property | 类型 | 描述
| --- | --- | ---
| layers | [LayerGroup](http://leafletjs.com/reference.html#layergroup) | 列出所有被移除出地图当图层

#### draw:drawstart

当用户开始新建一个矢量或者标记时触发。

| Property | 类型 | 描述
| --- | --- | ---
| layerType | String | 图层的类型是 `polyline`, `polygon`, `rectangle`, `circle`, `marker` 中的一种

#### draw:drawstop

当用户完成新建一个矢量或者标记时触发

| Property | 类型 | 描述
| --- | --- | ---
| layerType | String | 图层的类型是 `polyline`, `polygon`, `rectangle`, `circle`, `marker` 中的一种

#### draw:editstart

当用户点击编辑工具按钮，开启编辑模式时启动。

| Property | 类型 | 描述
| --- | --- | ---
| handler | String | 编辑的类型是 `edit`

#### draw:editstop

当用户完成编辑并保存编辑时触发。

| Property | 类型 | 描述
| --- | --- | ---
| handler | String | 编辑的类型是 `edit`

#### draw:deletestart

当用户通过点击移除工具按钮，开启移除模式。

| Property | 类型 | 描述
| --- | --- | ---
| handler | String | 编辑的类型是 `remove`

#### draw:deletestop

当用户完成移除并保存时触发

| Property | 类型 | 描述
| --- | --- | ---
| handler | String | 编辑的类型是 `remove`

<a name="options" />
## 高级选项

可以通过下面列出的不同选项来配置。

### Control.Draw

下面这些选项组成了初始化 Leaflet.draw 控件根对象。

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| position | String | `'topleft'` | 控件的初始位置（地图上的某一个角落） 请查阅控件位置 [control positions](http://leafletjs.com/reference.html#control-positions).
| draw | [DrawOptions](#drawoptions) | `{}` | 配置绘图工具栏的选项.
| edit | [EditOptions](#editoptions) | `false` | 配置编辑工具栏的选项.

<a name="drawoptions" />
### DrawOptions

这些选项允许你配置
These options will allow you to configure the draw toolbar and its handlers.

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| polyline | [PolylineOptions](#polylineoptions) | `{ }` | Polyline draw handler options. Set to `false` to disable handler.
| polygon | [PolygonOptions](#polygonoptions) | `{ }` | Polygon draw handler options. Set to `false` to disable handler.
| rectangle | [RectangleOptions](#rectangleoptions) | `{ }` | Rectangle draw handler options. Set to `false` to disable handler.
| circle | [CircleOptions](#circleoptions) | `{ }` | Circle draw handler options. Set to `false` to disable handler.
| marker | [MarkerOptions](#markeroptions) | `{ }` | Marker draw handler options. Set to `false` to disable handler.

### Draw handler options

以下的选项允许你配置单独的
The following options will allow you to configure the individual draw handlers.

<a name="polylineoptions" />
#### PolylineOptions

Polyline and Polygon drawing handlers take the same options.

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| allowIntersection | Bool | `true` | Determines if line segments can cross.
| drawError | Object | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Polyline.js#L10) | Configuration options for the error that displays if an intersection is detected.
| guidelineDistance | Number | `20` | Distance in pixels between each guide dash.
| shapeOptions | [Leaflet Polyline options](http://leafletjs.com/reference.html#polyline-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Polyline.js#L20) | The options used when drawing the polyline/polygon on the map.
| metric | Bool | `true` | 确定使用哪种测量标准（公制的，英制的）.
| zIndexOffset | Number | `2000` | This should be a high number to ensure that you can draw over all other layers on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="polygonoptions" />
#### PolygonOptions 多边形选项

Polygon 选项包含所有 Polyline 选项，加上显示多边形面积的选项。

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| showArea | Bool | `false` | 已显示 m², ha 或者 km² 单位来显示多边形的面积. **面积范围大小是粗略估计，并且多边形越大就会变得越不准确**

<a name="rectangleoptions" />
#### RectangleOptions 矩形选项

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| shapeOptions | [Leaflet Path options](http://leafletjs.com/reference.html#path-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Rectangle.js#L7) | The options used when drawing the rectangle on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="circleoptions" />
#### CircleOptions 圆形选项

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| shapeOptions | [Leaflet Path options](http://leafletjs.com/reference.html#path-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Circle.js#L7) | The options used when drawing the circle on the map. 
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="markeroptions" />
#### MarkerOptions 标记选项

| Option | 类型 | 默认值 | 描述
| --- | --- | --- | ---
| icon | [Leaflet Icon](http://leafletjs.com/reference.html#icon) | `L.Icon.Default()` | The icon displayed when drawing a marker.
| zIndexOffset | Number | `2000` | This should be a high number to ensure that you can draw over all other layers on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="editoptions" />
### EditOptions 编辑选项

These options will allow you to configure the draw toolbar and its handlers.

| Option | Type | Default | Description
| --- | --- | --- | ---
| featureGroup | [Leaflet FeatureGroup](http://leafletjs.com/reference.html#featuregroup) | `null` | This is the FeatureGroup that stores all editable shapes. **THIS IS REQUIRED FOR THE EDIT TOOLBAR TO WORK**
| edit | [EditHandlerOptions](#edithandleroptions) | `{ }` | Edit handler options. Set to `false` to disable handler.
| remove | [DeleteHandlerOptions](#deletehandleroptions) | `{ }` | Delete handler options. Set to `false` to disable handler.

<a name="edithandleroptions" />
#### EditHandlerOptions 处理编辑事件的选项

| Option | Type | Default | Description
| --- | --- | --- | ---
| selectedPathOptions | [Leaflet Path options](http://leafletjs.com/reference.html#path-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/edit/handler/EditToolbar.Edit.js#L9) | The path options for how the layers will look while in edit mode. If this is set to null the editable path options will not be set.

**Note:** To maintain the original layer color of the layer use `maintainColor: true` within `selectedPathOptions`.

E.g. The edit options below will maintain the layer color and set the edit opacity to 0.3.

````js
{
	selectedPathOptions: {
		maintainColor: true,
		opacity: 0.3
	}
}
````

<a name="deletehandleroptions" />
#### DeleteHandlerOptions 处理删除事件的选项

| Option | Type | Default | Description
| --- | --- | --- | ---

<a name="drawlocal" />
#### 定制 Leaflet.draw 上的语言和显示的文字

Leaflet.draw uses the `L.drawLocal` configuration object to set any text used in the plugin. Customizing this will allow support for changing the text or supporting another language.

See [Leaflet.draw.js](https://github.com/Leaflet/Leaflet.draw/blob/master/src/Leaflet.draw.js) for the default strings.

E.g.

````js
// Set the button title text for the polygon button
L.drawLocal.draw.toolbar.buttons.polygon = 'Draw a sexy polygon!';

// Set the tooltip start text for the rectangle
L.drawLocal.draw.handlers.rectangle.tooltip.start = 'Not telling...';
````

<a name="commontasks" />
## Common tasks 常见例子

下面的示例概述一些常见的用法.

### 配置 Leaflet.draw 示例

下面的例子将会告诉你，如何来：

1. 改变控件工具栏的位置.
2. 定制矢量图层的样式.
3. 使用定制的图标进行标记.
4. 禁用删除功能.

````js
var cloudmadeUrl = 'http://{s}.tile.cloudmade.com/BC9A493B41014CAABB98F0471D759707/997/256/{z}/{x}/{y}.png',
	cloudmade = new L.TileLayer(cloudmadeUrl, {maxZoom: 18}),
	map = new L.Map('map', {layers: [cloudmade], center: new L.LatLng(-37.7772, 175.2756), zoom: 15 });

var editableLayers = new L.FeatureGroup();
map.addLayer(editableLayers);

var MyCustomMarker = L.Icon.extend({
	options: {
		shadowUrl: null,
		iconAnchor: new L.Point(12, 12),
		iconSize: new L.Point(24, 24),
		iconUrl: 'link/to/image.png'
	}
});

var options = {
	position: 'topright',
	draw: {
		polyline: {
			shapeOptions: {
				color: '#f357a1',
				weight: 10
			}
		},
		polygon: {
			allowIntersection: false, // Restricts shapes to simple polygons
			drawError: {
				color: '#e1e100', // Color the shape will turn when intersects
				message: '<strong>Oh snap!<strong> you can\'t draw that!' // Message that will show when intersect
			},
			shapeOptions: {
				color: '#bada55'
			}
		},
		circle: false, // Turns off this drawing tool
		rectangle: {
			shapeOptions: {
				clickable: false
			}
		},
		marker: {
			icon: new MyCustomMarker()
		}
	},
	edit: {
		featureGroup: editableLayers, //REQUIRED!!
		remove: false
	}
};

var drawControl = new L.Control.Draw(options);
map.addControl(drawControl);

map.on('draw:created', function (e) {
	var type = e.layerType,
		layer = e.layer;

	if (type === 'marker') {
		layer.bindPopup('A popup!');
	}

	drawnItems.addLayer(layer);
});
````

### 禁用一个工具栏

If you do not want a particular toolbar in your app you can turn it off by setting the toolbar to false.

````js
var drawControl = new L.Control.Draw({
	draw: false,
	edit: {
		featureGroup: editableLayers
	}
});
````

### 禁用工具栏的某一栏

If you want to turn off a particular toolbar item, set it to false. The following disables drawing polygons and markers. It also turns off the ability to edit layers.

````js
var drawControl = new L.Control.Draw({
	draw: {
		polygon: false,
		marker: false
	},
	edit: {
		featureGroup: editableLayers,
		edit: false
	}
});
````

### Changing a drawing handlers options

You can change a draw handlers options after initialisation by using the `setDrawingOptions` method on the Leaflet.draw control.

例： 改变矩形框的颜色
E.g. to change the colour of the rectangle:

````js
drawControl.setDrawingOptions({
    rectangle: {
    	shapeOptions: {
        	color: '#0000FF'
        }
    }
});
````

### Creating a custom build

If you only require certain handlers (and not the UI), you may wish to create a custom build. You can generate the relevant jake command using the [build html file](https://github.com/Leaflet/Leaflet.draw/blob/master/build/build.html). 

See [edit handlers example](https://github.com/Leaflet/Leaflet.draw/blob/master/examples/edithandlers.html) which uses only the edit handlers.

<a name="thanks" />
## 感谢

Thanks so much to [@brunob](https://github.com/brunob), [@tnightingale](https://github.com/tnightingale), and [@shramov](https://github.com/shramov). I got a lot of ideas from their Leaflet plugins.

All the [contributors](https://github.com/Leaflet/Leaflet.draw/graphs/contributors) and issue reporters of this plugin rock. Thanks for tidying up my mess and keeping the plugin on track.

The icons used for some of the toolbar buttons are either from http://glyphicons.com/ or inspired by them. <3 Glyphicons!

Finally, [@mourner](https://github.com/mourner) is the man! Thanks for dedicating so much of your time to create the gosh darn best JavaScript mapping library around.
