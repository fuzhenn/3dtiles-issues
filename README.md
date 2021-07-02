# 3dtiles插件bug
用于报告@maptalks/3dtiles的bug或反馈

# 示例代码
```html
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/maptalks@next/dist/maptalks.min.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@maptalks/gl/dist/maptalksgl.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@maptalks/3dtiles@latest/dist/maptalks.3dtiles.js"></script>
<script>
const layer = new maptalks.Geo3DTilesLayer('3dtiles', {        
    maxCacheSize : 1000, //缓存的最大瓦片数
    // loadingLimitOnInteracting : 1, //地图交互过程中瓦片请求最大数量
    // loadingLimit : 0, //瓦片请求最大数量
    services : [
        {
            url : 'path/to/tileset.json',
           
            maximumScreenSpaceError : 24.0,
            // urlParams : 'v=0.0',
            // ajaxOptions : { credentials : 'include' },
            // 把模型降低1200米
            heightOffset : -1200,
            ambientLight : [0.0, 0.0, 0.0],
            // maxExtent : maxExtent
            // ajaxInMainThread : true, //从主线程发起ajax请求，用于worker发起ajax报跨域错误
        },
        // 其他的3dtiles数据源
    ]
});
layer.addTo(map);
</script>

```
