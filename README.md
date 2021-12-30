# 3dtiles插件bug
[![NPM Version](https://img.shields.io/npm/v/@maptalks/3dtiles.svg)](https://github.com/fuzhenn/3dtiles-issues)

本仓库用于报告@maptalks/3dtiles的bug或功能建议

支持的功能:
- [X] [B3DM格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Batched3DModel) 批量模型格式，一般用于倾斜摄影
- [X] [PNTS格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/PointCloud)，点云格式
- [X] [I3DM格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Instanced3DModel)，示例三维模型格式，一般用于大量重复的小品模型加载
- [X] [CMPT格式](https://github.com/CesiumGS/3d-tiles/tree/main/specification/TileFormats/Composite)，复合格式，其中包含单个或多个其他格式瓦片
- [X] [3DTiles的Draco扩展](https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Khronos/KHR_draco_mesh_compression/README.md) Draco压缩扩展
- [X] CRN图片格式
- [ ] KTX2图片格式
- [ ] 3DTiles Next标准

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
// 添加一个GroupGLLayer
const groupLayer = new maptalks.GroupGLLayer('group', [layer]);
groupLayer.addTo(map);
</script>
```
## npm安装
```
npm i @maptalks/3dtiles
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
