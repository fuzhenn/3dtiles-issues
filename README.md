# @maptalks/3dtiles
[![NPM Version](https://img.shields.io/npm/v/@maptalks/3dtiles.svg)](https://github.com/fuzhenn/3dtiles-issues)

maptalks的3DTiles渲染图层插件，用于加载Cesium的3DTiles格式数据。

特点：
* 个头小：gzip压缩前只有100多K（目前200多K是因为开启了源代码格式化）
* 性能高：可以通过调整maximumScreenSpaceError来获得很高的渲染性能
* 支持全：对所有3DTiles 1.0的格式均提供了支持
* 测试全：包含了Cesium所有相关格式的测试用例，以及实际项目中的数据用例，您提交的错误数据在您的允许下也会增加到测试用例中，保证未来的稳定性。
* 可与其他maptalks三维图层（例如矢量瓦片图层）融合渲染

支持的功能:
- [X] [B3DM格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Batched3DModel) 批量模型格式，一般用于倾斜摄影
- [X] [PNTS格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/PointCloud)，点云格式
- [X] [I3DM格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Instanced3DModel)，示例三维模型格式，一般用于大量重复的小品模型加载
- [X] [CMPT格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Composite)，复合格式，其中包含单个或多个其他格式瓦片
- [X] [3DTiles的Draco扩展](https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Khronos/KHR_draco_mesh_compression/README.md) Draco压缩扩展
- [X] CRN图片纹理格式
- [ ] KTX2图片纹理格式
- [ ] 3DTiles Next标准

如果遇到bug或者功能建议，请直接在本仓库中提交。

# 示例代码
```html
<script type="text/javascript" src="https://unpkg.com/maptalks/dist/maptalks.min.js"></script>
<script type="text/javascript" src="https://unpkg.com/@maptalks/gl/dist/maptalksgl.js"></script>
<script type="text/javascript" src="https://unpkg.com/@maptalks/3dtiles/dist/maptalks.3dtiles.js"></script>
<script>
const layer = new maptalks.Geo3DTilesLayer('3dtiles', {        
    maxGPUMemory: 512, //最大缓存数，单位 M bytes
    // loadingLimitOnInteracting : 1, //地图交互过程中瓦片请求最大数量
    // loadingLimit : 0, //瓦片请求最大数量
    services : [
        {
            url: 'path/to/tileset.json',
            // maximumScreenSpaceError值越小，加载的模型越清晰，但加载的数据量会变大
            // 清晰度可以接受的情况下，推荐把这个值设得越大越好，性能会越好
            maximumScreenSpaceError: 24.0,
            // 数据请求的额外参数
            // urlParams: 'v=0.0',
            // ajax请求的额外参数
            // ajaxOptions : { credentials : 'include' },
            // 把模型降低1200米
            heightOffset: -1200,
            // 环境光照值，倾斜摄影可以设为[1.0, 1.0, 1.0]获得最清晰的效果，非倾斜摄影可以适当降低，例如设为 [0.2, 0.2, 0.2]
            // 如果不设置，则采用地图上的默认光照值
            ambientLight: [1.0, 1.0, 1.0],
            // maxExtent: maxExtent
        },
        // 其他的3dtiles数据源
    ]
});
// 添加到GroupGLLayer中
// GroupGLLayer能实现抗锯齿等后处理，也能加入其他三维图层，让子图层都融合到同一个三维空间中
const groupLayer = new maptalks.GroupGLLayer('group', [layer]);
groupLayer.addTo(map);
    
layer.once('loadtileset', e => {
    const extent = layer.getExtent(e.index);
    map.fitExtent(extent, 0, { animation: false });
});
</script>
```
## npm安装
```
npm i @maptalks/3dtiles
```
### 使用
esm方式:
```js
import { GroupGLLayer } from '@maptalks/gl';
// 可选的draco插件
// import '@maptalks/transcoders.draco';
import { Geo3DTilesLayer } from '@maptalks/3dtiles';
```
commonjs方式：
```js
const { GroupGLLayer } = require('@maptalks/gl');
// 可选的draco插件
// require('@maptalks/transcoders.draco');
const { Geo3DTilesLayer } = require('@maptalks/3dtiles');
```

## 坐标系适配

我们可以通过给图层设置一个动态的 `offset` 选项，来适配不同的坐标系，例如 `cgcs2000`, `gcj02` 等。

坐标系转换已经有不少库，例如 [coordtransform](https://github.com/wandergis/coordtransform), [gcoord](https://github.com/hujiulong/gcoord)。

示例中用的是 [chinese_coordinate_conversion](https://github.com/fuzhenn/chinese_coordinate_conversion)。

示例代码：

```js
<script type="text/javascript" src="https://fuzhenn.github.io/chinese_coordinate_conversion/chncrs.js"></script>
<script>
const layer = new maptalks.Geo3DTilesLayer('3dtiles', {        
    // 动态 offset 选项
    offset : function (center) {
        const res = map.getGLRes();
        // 适配GJC02底图
        const c = maptalks.CRSTransform.transform(center.toArray(), 'GCJ02', 'WGS84');
        const coord = map.coordToPointAtRes(new maptalks.Coordinate(c), res);
        const offset = map.coordToPointAtRes(center, res)._sub(coord);
        return offset._round().toArray();
    },
    services : [
        {
            url : 'path/to/tileset.json',
            //模型载入精度，在可接受尽量设置的大一些，以提升效率
            maximumScreenSpaceError : 16.0,
            //额外的模型url请求参数
            // urlParams : '',
            //高度偏移量，单位米，可以把模型整体
            heightOffset : 0,
            //环境光参数
            ambientLight : [1.0, 1.0, 1.0],
        },
    ]
});
</script>
```

## Draco解码插件
因为Draco解码程序体积较大，采用通用插件形式提供，即所有maptalks的插件都共用同一个Draco插件。

默认情况下，没加载解码插件时，如果模型是Draco格式编码，控制台会报错无法找到draco解码插件。
```
KHR_draco_mesh_compression is required but @maptalks/transcoders.draco is not loaded
```
解决方案需要加载@maptalks/gl同时，加载通用draco解码插件即可。
```html
<script type="text/javascript" src="https://unpkg.com/maptalks/dist/maptalks.min.js"></script>
<script type="text/javascript" src="https://unpkg.com/@maptalks/gl/dist/maptalksgl.js"></script>
<!-- draco插件，必须写在gl后面，其他插件的前面，es方式加载时同理 -->
<script type="text/javascript" src="https://unpkg.com/@maptalks/transcoders.draco/dist/transcoders.draco.js"></script>
<script type="text/javascript" src="https://unpkg.com/@maptalks/3dtiles/dist/maptalks.3dtiles.js"></script>
```
### npm安装draco插件
```
npm i @maptalks/transcoders.draco
```
### 使用
esm方式:
```js
import { GroupGLLayer } from '@maptalks/gl';
import '@maptalks/transcoders.draco';
import { Geo3DTilesLayer } from '@maptalks/3dtiles';
```
commonjs方式：
```js
const { GroupGLLayer } = require('@maptalks/gl');
require('@maptalks/transcoders.draco');
const { Geo3DTilesLayer } = require('@maptalks/3dtiles');
```
## CRN纹理支持
和Draco一样，crn纹理也是采用通用插件方式实现的，添加crn解码插件即可
```html
<script type="text/javascript" src="https://unpkg.com/@maptalks/transcoders.crn/dist/transcoders.crn.js"></script>
```
## KTX2纹理支持
开发中。
和Draco一样，ktx2纹理也是采用通用插件方式实现的，添加ktx解码插件即可
```html
<script type="text/javascript" src="https://unpkg.com/@maptalks/transcoders.crn/dist/transcoders.ktx.js"></script>
```

## 抗锯齿
默认情况下3dtiles绘制时会有很多锯齿，可以在GroupGLLayer上开启抗锯齿来解决。
```js
    const sceneConfig = {
        //开启后处理
        postProcess: {
            enable: true,
            //开启抗锯齿后处理
            antialias: {
                enable: true
            }
        }
    };
    
    const groupLayer = new maptalks.GroupGLLayer(id, [layer], { sceneConfig });
    groupLayer.addTo(map);
```
# API 说明
### `Constructor`
```javascript
new maptalks.Geo3DTilesLayer(id, options);
```
* id **String** layer id
* options **Object** options
  * options.maxGPUMemory **Number** 最大缓存占用内存，单位 M bytes，默认为512
  * options.loadingLimitOnInteracting **Number** 地图交互过程中瓦片请求最大数量，默认为1
  * options.loadingLimit  **Number** 瓦片请求最大数量，默认为0，即不受限制
  * options.services **[Object]** 3dtiles服务定义，可以一次定义多个，具体字段参考上面的示例代码

### `getExtent(index)`
获得指定service的地理范围，可以用于快速定位到指定的3dtiles service
```js
layer.once('tilesetload', e => {
  const extent = layer.getExtent(e.index);
  map.fitExtent(extent, 0, { animation: false });
});
```
* index **Number** service的序号

**Returns** maptalks.Extent

# 事件说明
### `rootready`
初始化根节点结束事件
#### 事件参数
* roots: 根节点对象

### `loadtileset`
成功加载tileset.json事件
#### 事件参数
* tileset: tileset对象
* index: tileset对应的3dtiles service的序号
* url: tileset.json的url，绝对地址

### `canvasisdirty`
图层canvas上有绘制时的事件，一般用于单元测试时判断图层是否产生绘制
#### 事件参数
* renderCount: 绘制指令数

### `tileload`
成功加载一个3dtiles瓦片事件
#### 事件参数
* node 瓦片节点对象

### `tileerror`
加载3dtiles瓦片失败事件
#### 事件参数
* node 节点对象
* error 错误对象

### `workerready`
图层worker初始化成功事件

# 已知问题
* I3DM中包含自定义旋转数据时，因为Cesium默认的ECS投影系与maptalks的平面投影系转换问题，相比在Cesium中加载，I3DM模型的旋转方向可能会出现错误
