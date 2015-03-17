翻译内容：


| 英文 | 中文 |
| - | - |
| set the view |  |
| draw toolbar | 绘图工具栏 |
| control | 控件 |
| view | 区域 |
| Leaflet.draw plugin | 插件 |



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
一旦成功将 Leaflet.draw 插件添加到地图上，我们会想要响应用户可以发起到动作。
Once you have successfully added the Leaflet.draw plugin to your map you will want to respond to the different actions users can initiate. The following events will be triggered on the map:

#### draw:created

| Property | Type | Description
| --- | --- | ---
| layer | [Polyline](http://leafletjs.com/reference.html#polyline)/[Polygon](http://leafletjs.com/reference.html#polygon)/[Rectangle](http://leafletjs.com/reference.html#rectangle)/[Circle](http://leafletjs.com/reference.html#circle)/[Marker](http://leafletjs.com/reference.html#marker) | Layer that was just created.
| layerType | String | The type of layer this is. One of: `polyline`, `polygon`, `rectangle`, `circle`, `marker`


Triggered when a new vector or marker has been created.

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

| Property | Type | Description
| --- | --- | ---
| layers | [LayerGroup](http://leafletjs.com/reference.html#layergroup) | List of all layers just edited on the map.

Triggered when layers in the FeatureGroup, initialised with the plugin, have been edited and saved.

````js
map.on('draw:edited', function (e) {
	var layers = e.layers;
	layers.eachLayer(function (layer) {
		//do whatever you want, most likely save back to db
	});
});
````

#### draw:deleted

Triggered when layers have been removed (and saved) from the FeatureGroup.

| Property | Type | Description
| --- | --- | ---
| layers | [LayerGroup](http://leafletjs.com/reference.html#layergroup) | List of all layers just removed from the map.

#### draw:drawstart

Triggered when the user has chosen to draw a particular vector or marker.

| Property | Type | Description
| --- | --- | ---
| layerType | String | The type of layer this is. One of: `polyline`, `polygon`, `rectangle`, `circle`, `marker`

#### draw:drawstop

Triggered when the user has finished a particular vector or marker.

| Property | Type | Description
| --- | --- | ---
| layerType | String | The type of layer this is. One of: `polyline`, `polygon`, `rectangle`, `circle`, `marker`

#### draw:editstart

Triggered when the user starts edit mode by clicking the edit tool button.

| Property | Type | Description
| --- | --- | ---
| handler | String | The type of edit this is. One of: `edit`

#### draw:editstop

Triggered when the user has finshed editing (edit mode) and saves edits.

| Property | Type | Description
| --- | --- | ---
| handler | String | The type of edit this is. One of: `edit`

#### draw:deletestart

Triggered when the user starts remove mode by clicking the remove tool button.

| Property | Type | Description
| --- | --- | ---
| handler | String | The type of edit this is. One of: `remove`

#### draw:deletestop

Triggered when the user has finished removing shapes (remove mode) and saves.

| Property | Type | Description
| --- | --- | ---
| handler | String | The type of edit this is. One of: `remove`

<a name="options" />
## Advanced options

You can configure the plugin by using the different options listed here.

### Control.Draw

These options make up the root object that is used when initialising the Leaflet.draw control.

| Option | Type | Default | Description
| --- | --- | --- | ---
| position | String | `'topleft'` | The initial position of the control (one of the map corners). See [control positions](http://leafletjs.com/reference.html#control-positions).
| draw | [DrawOptions](#drawoptions) | `{}` | The options used to configure the draw toolbar.
| edit | [EditOptions](#editoptions) | `false` | The options used to configure the edit toolbar.

<a name="drawoptions" />
### DrawOptions

These options will allow you to configure the draw toolbar and its handlers.

| Option | Type | Default | Description
| --- | --- | --- | ---
| polyline | [PolylineOptions](#polylineoptions) | `{ }` | Polyline draw handler options. Set to `false` to disable handler.
| polygon | [PolygonOptions](#polygonoptions) | `{ }` | Polygon draw handler options. Set to `false` to disable handler.
| rectangle | [RectangleOptions](#rectangleoptions) | `{ }` | Rectangle draw handler options. Set to `false` to disable handler.
| circle | [CircleOptions](#circleoptions) | `{ }` | Circle draw handler options. Set to `false` to disable handler.
| marker | [MarkerOptions](#markeroptions) | `{ }` | Marker draw handler options. Set to `false` to disable handler.

### Draw handler options

The following options will allow you to configure the individual draw handlers.

<a name="polylineoptions" />
#### PolylineOptions

Polyline and Polygon drawing handlers take the same options.

| Option | Type | Default | Description
| --- | --- | --- | ---
| allowIntersection | Bool | `true` | Determines if line segments can cross.
| drawError | Object | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Polyline.js#L10) | Configuration options for the error that displays if an intersection is detected.
| guidelineDistance | Number | `20` | Distance in pixels between each guide dash.
| shapeOptions | [Leaflet Polyline options](http://leafletjs.com/reference.html#polyline-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Polyline.js#L20) | The options used when drawing the polyline/polygon on the map.
| metric | Bool | `true` | Determines which measurement system (metric or imperial) is used.
| zIndexOffset | Number | `2000` | This should be a high number to ensure that you can draw over all other layers on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="polygonoptions" />
#### PolygonOptions

Polygon options include all of the Polyline options plus the option to show the approximate area.

| Option | Type | Default | Description
| --- | --- | --- | ---
| showArea | Bool | `false` | Show the area of the drawn polygon in m², ha or km². **The area is only approximate and become less accurate the larger the polygon is.**

<a name="rectangleoptions" />
#### RectangleOptions

| Option | Type | Default | Description
| --- | --- | --- | ---
| shapeOptions | [Leaflet Path options](http://leafletjs.com/reference.html#path-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Rectangle.js#L7) | The options used when drawing the rectangle on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="circleoptions" />
#### CircleOptions

| Option | Type | Default | Description
| --- | --- | --- | ---
| shapeOptions | [Leaflet Path options](http://leafletjs.com/reference.html#path-options) | [See code](https://github.com/Leaflet/Leaflet.draw/blob/master/src/draw/handler/Draw.Circle.js#L7) | The options used when drawing the circle on the map. 
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="markeroptions" />
#### MarkerOptions

| Option | Type | Default | Description
| --- | --- | --- | ---
| icon | [Leaflet Icon](http://leafletjs.com/reference.html#icon) | `L.Icon.Default()` | The icon displayed when drawing a marker.
| zIndexOffset | Number | `2000` | This should be a high number to ensure that you can draw over all other layers on the map.
| repeatMode | Bool | `false` | Determines if the draw tool remains enabled after drawing a shape.

<a name="editoptions" />
### EditOptions

These options will allow you to configure the draw toolbar and its handlers.

| Option | Type | Default | Description
| --- | --- | --- | ---
| featureGroup | [Leaflet FeatureGroup](http://leafletjs.com/reference.html#featuregroup) | `null` | This is the FeatureGroup that stores all editable shapes. **THIS IS REQUIRED FOR THE EDIT TOOLBAR TO WORK**
| edit | [EditHandlerOptions](#edithandleroptions) | `{ }` | Edit handler options. Set to `false` to disable handler.
| remove | [DeleteHandlerOptions](#deletehandleroptions) | `{ }` | Delete handler options. Set to `false` to disable handler.

<a name="edithandleroptions" />
#### EditHandlerOptions

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
#### DeleteHandlerOptions

| Option | Type | Default | Description
| --- | --- | --- | ---

<a name="drawlocal" />
#### Customizing language and text in Leaflet.draw

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
## Common tasks

The following examples outline some common tasks.

### Example Leaflet.draw config

The following example will show you how to:

1. Change the position of the control's toolbar.
2. Customize the styles of a vector layer.
3. Use a custom marker.
4. Disable the delete functionality.

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

### Disabling a toolbar

If you do not want a particular toolbar in your app you can turn it off by setting the toolbar to false.

````js
var drawControl = new L.Control.Draw({
	draw: false,
	edit: {
		featureGroup: editableLayers
	}
});
````

### Disabling a toolbar item

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
## Thanks

Thanks so much to [@brunob](https://github.com/brunob), [@tnightingale](https://github.com/tnightingale), and [@shramov](https://github.com/shramov). I got a lot of ideas from their Leaflet plugins.

All the [contributors](https://github.com/Leaflet/Leaflet.draw/graphs/contributors) and issue reporters of this plugin rock. Thanks for tidying up my mess and keeping the plugin on track.

The icons used for some of the toolbar buttons are either from http://glyphicons.com/ or inspired by them. <3 Glyphicons!

Finally, [@mourner](https://github.com/mourner) is the man! Thanks for dedicating so much of your time to create the gosh darn best JavaScript mapping library around.
