## vue-image-cropper-pro 基于vue的图片压缩裁剪插件

### 1.功能说明
鉴于 vue-image-cropper 作者已经不再维护，本次进行版本升级进一步加强竖屏拍摄要求。提前压缩旋转图片。
在竖屏拍摄可随机调整大小。完美兼容手机剪裁，感谢原作者。

### 2.本地运行
```bash
cd vue-image-cropper-pro
yarn
yarn run dev
```
### 3.使用
1. copy本项目的 ./src/components/imageCropper.vue文件 (imageCropper.vue需要引入exif-small.js)
2. 在需要使用的页面直接引入imageCropper组件并绑定cropperConfig配置参数和裁剪之后的回调函数callback
```HTML
<image-cropper ref="imageCropper" :cropperConfig="cropperConfig" :callback="loadImage"></image-cropper>
```
```javascript
loadImage (data) {
  console.log(data)
  // data为base64字符串
}
```
### 4.参数说明
```javascript
cropperConfig: {
  width: 1, // 裁剪宽度（比例）
  height: 1, // 裁剪高度（比例）
  quality: 0.7, // 图片质量（0~1之间）
  maxWidth: 750 // 导出的图片的最大宽度
}
```



