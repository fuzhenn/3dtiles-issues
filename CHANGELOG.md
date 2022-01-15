## 更新日志

### V0.12.0
2022-01-15
* 增加了CRN纹理支持：需要加载CRN纹理插件
* 增加了KTX2纹理支持：需要加载KTX2纹理插件
```html
<script type="text/javascript" src="/gl/packages/transcoders.ktx/dist/transcoders.ktx2.js"></script>
<script type="text/javascript" src="/gl/packages/transcoders.crn/dist/transcoders.crn.js"></script>
```

### V0.11.5
2022-01-11
* 解决请求404时的无限死循环

### V0.11.4
2022-01-11
* 解决了一个CMPT模型的解析bug

### V0.11.3
2022-01-11
* 解决FireFox默认不支持OffscreenCanvas，导致纹理加载出错的bug

### V0.11.2
2022-01-11
* 解决一个boundingVolume重复转换，导致瓦片bbox计算错误的bug

### V0.11.1
2022-01-11
* 解决options.offset的方向错误 

### V0.11.0
2022-01-11
* 解决了大家发过来的错误模型的绘制（感谢大家！）
* 升级了新的模型算法，大幅度提升了模型构造性能
* 图层增加了offset选项，用于动态偏移模型，适配其他坐标系底图如gcj02底图，百度底图等