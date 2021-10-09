# 3dtiles插件bug
[![NPM Version](https://img.shields.io/npm/v/@maptalks/3dtiles.svg)](https://github.com/fuzhenn/3dtiles-issues)

用于报告@maptalks/3dtiles的bug或反馈

# 示例代码
```html
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/maptalks@next/dist/maptalks.min.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@maptalks/gl/dist/maptalksgl.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@maptalks/3dtiles@latest/dist/maptalks.3dtiles.js"></script>
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
            // urlParams: 'v=0.0',
            // ajaxOptions : { credentials : 'include' },
            // 把模型降低1200米
            heightOffset: -1200,
            ambientLight: [0.0, 0.0, 0.0],
            // maxExtent: maxExtent
            // ajaxInMainThread: true, //从主线程发起ajax请求，用于worker发起ajax报跨域错误
        },
        // 其他的3dtiles数据源
    ]
});
layer.addTo(map);
</script>

```
