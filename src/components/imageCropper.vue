<style lang="scss" scoped>
$themeColor: #45b795;
.box {
  display: flex;
  position: relative;
}
.box-f1 {
  flex: 1;
}
.box-ac {
  align-items: center;
}
.box-jc {
  justify-content: center;
}
.box-ver {
  flex-direction: column;
}
.cropper-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  background-color: #fff;
  overflow: hidden;
}
.cover {
  color: #fff;
  font-size: 0.4rem;
  background-color: rgba(0, 0, 0, 0.2);
}
.cropper-box {
  border: 1px dashed #fff;
}
.cropper-img {
  position: absolute;
  z-index: -1;
}
input[type="file"] {
  opacity: 0;
  position: fixed;
  top: -1000px;
  left: -1000px;
}
.btn {
  font-size: 0.3rem;
  padding: 0.1rem 0.3rem;
  border: 0.02rem solid;
  color: #fff;
  border-radius: 0.1rem;
  &.sure {
    color: $themeColor;
  }
}
</style>

<template>
  <div
    ref="cropperPage"
    class="cropper-page"
    v-show="isShow"
  >
    <div
      @click="cancel()"
      class="icon icon-back"
    ></div>
    <input
      ref="file"
      type="file"
      accept="image/*"
      @change="readImage"
    >
    <img
      alt=""
      class="cropper-img"
      :style="imageStyle"
      ref="img"
    >
    <div
      class="cover box box-ac box-jc"
      :style="{height: coverHeight + 'px'}"
    >
      请调整图片
    </div>
    <div
      ref="cropBox"
      class="cropper-box"
      @touchstart.prevent="touchStart"
      @touchmove.prevent="touchMove"
      @touchend.prevent="touchEnd"
    ></div>
    <div
      class="cover cover box box-ac box-jc"
      :style="{height: coverHeight + 'px'}"
    >
      <div class="box box-f1 box-jc box-fh">
        <div
          class="btn"
          @click="checkPhoto"
        >重选</div>
      </div>
      <div class="box box-f1 box-jc box-fh">
        <div
          class="btn sure"
          @click="confirm"
        >确定</div>
      </div>
    </div>
  </div>
</template>

<script>
import EXIF from "../assets/js/exif-small";
function selectFileImage (file, callback) {
  // if (window.EXIF !== undefined) {
  //   var EXIF = window.EXIF;
  // }
  // 图片方向角 added by lzk
  var Orientation = null;

  // var inputID="file";
  if (!(window.FileReader && window.File && window.FileList && window.Blob)) {
    alert("您的浏览器不支持fileReader");
    return false;
  }
  var reader = new FileReader();

  // 获取照片方向角属性，用户旋转控制
  EXIF.getData(file, function () {
    Orientation = EXIF.getTag(this, "Orientation");
  });

  reader.onload = function (e) {
    var image = document.createElement("img");
    image.onload = function () {
      var canvas = document.createElement("canvas");
      console.log(Orientation);
      var x, y;
      if (Orientation === 6) {
        if (this.width > this.height) {
          x = this.height / 4;
          y = this.width / 4;
        } else {
          x = this.width / 4;
          y = this.height / 4;
        }
      } else {
        x = this.width / 4;
        y = this.height / 4;
      }

      var width = x;
      var height = y;

      canvas.width = x;
      canvas.height = y;

      var context = canvas.getContext("2d");
      context.clearRect(0, 0, width, height);

      // context.drawImage(this, 0, 0, x, y);

      switch (Orientation) {
        case 3:
          context.rotate((180 * Math.PI) / 180);
          context.drawImage(this, -x, -y, x, y);
          break;
        case 6:
          context.rotate((90 * Math.PI) / 180);
          context.drawImage(this, 0, -x, y, x);
          break;
        case 8:
          context.rotate((270 * Math.PI) / 180);
          context.drawImage(this, -y, 0, y, x);
          break;
        case 2:
          context.translate(x, 0);
          context.scale(-1, 1);
          context.drawImage(this, 0, 0, x, y);
          break;
        case 4:
          context.translate(x, 0);
          context.scale(-1, 1);
          context.rotate((180 * Math.PI) / 180);
          context.drawImage(this, -x, -y, x, y);
          break;
        case 5:
          context.translate(x, 0);
          context.scale(-1, 1);
          context.rotate((90 * Math.PI) / 180);
          context.drawImage(this, 0, -x, y, x);
          break;
        case 7:
          context.translate(x, 0);
          context.scale(-1, 1);
          context.rotate((270 * Math.PI) / 180);
          context.drawImage(this, -y, 0, y, x);
          break;
        default:
          context.drawImage(this, 0, 0, x, y);
      }

      var quality = 0.9; // 可以理解为压缩程度
      // quality :图片质量。0 到 1 之间的数字，并且只在格式为 image/jpeg 或 image/webp 时才有效，如果参数值格式不合法，将会被忽略并使用默认值
      var data;
      var result;
      var length;
      // 处理图片过大问题
      // 控制图片上传的大小，注意若图片过大 ajax上传的时候会出现参数为null的错误
      // 这里如果图片过大会将图片压缩程度放大
      data = canvas.toDataURL("image/jpeg", quality);
      // do {
      //      // 压缩图片
      //     length = data.length
      //     result = ((length / 4) * 3 + 1023) / 1024 // 计算压缩后图片的大小（单位：KB）
      //     quality -= 0.05
      // } while (result > 450)
      // data：base64
      let blob = base64ToBlob(data);
      // 回调函数
      callback(data, blob, file);
    };
    image.src = e.target.result;
  };
  reader.readAsDataURL(file);
}

function base64ToBlob (base64Data) {
  var format = "image/jpeg";
  var base64 = base64Data;
  var code = window.atob(base64.split(",")[1]);
  var aBuffer = new window.ArrayBuffer(code.length);
  var uBuffer = new window.Uint8Array(aBuffer);
  for (var i = 0; i < code.length; i++) {
    uBuffer[i] = code.charCodeAt(i) & 0xff;
  }
  // console.info([aBuffer]);
  // console.info(uBuffer);
  // console.info(uBuffer.buffer);
  // console.info(uBuffer.buffer === aBuffer); //true

  var blob = null;
  try {
    blob = new Blob([uBuffer], {
      type: format
    });
  } catch (e) {
    window.BlobBuilder =
      window.BlobBuilder ||
      window.WebKitBlobBuilder ||
      window.MozBlobBuilder ||
      window.MSBlobBuilder;
    if (e.name === "TypeError" && window.BlobBuilder) {
      var bb = new window.BlobBuilder();
      bb.append(uBuffer.buffer);
      blob = bb.getBlob("image/jpeg");
    } else if (e.name === "InvalidStateError") {
      blob = new Blob([aBuffer], {
        type: format
      });
    } else {
    }
  }
  return blob;
}

const getDinstance = function (point0, point1) {
  return Math.sqrt(
    Math.pow(point0.pageY - point1.pageY, 2) +
      Math.pow(point0.pageX - point1.pageX, 2)
  );
};
export default {
  name: "imageCropper",
  props: {
    callback: {
      type: Function,
      default () {}
    },
    cropperConfig: {
      type: Object,
      default () {
        return {
          width: 1,
          height: 1,
          quality: 0.7,
          maxWidth: 640
        };
      }
    }
  },
  data () {
    return {
      coverHeight: 0,
      cropperHeight: 0,
      imgInitTop: 0,
      amplitude: 0,
      imageState: {
        left: 0,
        top: 0,
        scale: 1,
        width: 0,
        height: 0,
        originX: 0,
        originY: 0
      },
      distance: 0,
      imageStyle: {
        top: "0",
        transform: "translate3d(0px, 0px, 0px) scale(1)",
        transformOrigin: "left top"
      },
      cropBoxRect: {},
      touchPos: {
        x: 0,
        y: 0
      },
      isShow: false,
      minScale: 0,
      info: "",
      orientation: ""
    };
  },
  watch: {
    imageState: {
      handler (val, oldVal) {
        // console.log(val)
        this.imageStyle.transform =
          "translate3d(-" +
          val.left +
          "px, -" +
          val.top +
          "px, 0px) scale(" +
          val.scale +
          ")";
      },
      deep: true
    }
  },
  methods: {
    cancel () {
      this.file = null;
      this.isShow = false;
    },
    checkPhoto () {
      this.$refs.file.click();
    },
    readImage ($event) {
      this.$store.commit("isLoading", true);
      var self = this;
      var file = $event.target.files[0];
      var reader = new window.FileReader();
      selectFileImage($event.target.files[0], function (base64, blob) {
        reader.onload = () => {
          EXIF.getData(file, function () {
            let orientation = EXIF.getTag(this, 'Orientation')
            if (!orientation) orientation = 1
            console.log('orientation>>>', orientation)
            self.orientation = orientation
            self.$refs.img.onload = () => {
              self.$store.commit('isLoading', false)
              self.initCropper()
            }
            self.$refs.img.src = base64;
            $event.target.value = null
          })
        }
        reader.readAsDataURL(file)
      });
    },
    initCropper () {
      this.isShow = true; // 显示裁剪界面
      this.$nextTick(() => {
        let cropperPage = this.$refs.cropperPage;
        let pageWidth = cropperPage.clientWidth;
        let pageHeight = cropperPage.clientHeight;
        let cropBox = this.$refs.cropBox;
        let cropBoxWidth = cropBox.clientWidth;
        let cropBoxHeight = Math.floor(
          (cropBoxWidth * +this.cropperConfig.height) /
            +this.cropperConfig.width
        );
        this.$refs.cropBox.style.height = cropBoxHeight + "px";
        this.coverHeight = (pageHeight - cropBoxHeight) / 2;
        let cropBoxTop = this.coverHeight;
        this.imageState.left = 0;
        this.imageState.top = 0;

        this.imageStyle.top = cropBoxTop + "px";

        this.cropBoxRect = {
          left: 0,
          top: cropBoxTop,
          width: pageWidth,
          height: cropBoxHeight
        };
        let img = this.$refs.img;
        var width = (this.imageState.width = img.naturalWidth);
        var height = (this.imageState.height = img.naturalHeight);
        // 计算imageState
        if (width > height) {
          this.minScale = this.imageState.scale =
            this.cropBoxRect.height / height;
          this.imageState.left =
            (width * this.imageState.scale - this.cropBoxRect.width) / 2;
        } else {
          this.minScale = this.imageState.scale =
            this.cropBoxRect.width / width;
          this.imageState.top =
            (height * this.imageState.scale - this.cropBoxRect.height) / 2;
        }
      });
    },
    confirm () {
      let self = this;
      let imageState = this.imageState;
      let cropBoxRect = this.cropBoxRect;

      // 导出图片的最大宽度
      let maxWidth = this.cropperConfig.maxWidth;
      let scale2 = maxWidth / cropBoxRect.width;

      let scale = imageState.scale * scale2;
      let width = cropBoxRect.width * scale2;
      let height = cropBoxRect.height * scale2;
      let left = imageState.left * scale2;
      let top = imageState.top * scale2;
      let image = this.$refs.img;

      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      // ios 的照片有拍摄的角度信息 参考 http://www.bcty365.com/content-142-3055-1.html
      let orientation = this.orientation;
      switch (orientation) {
        case 1:
          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(
            image,
            left / scale,
            top / scale,
            width / scale,
            height / scale,
            0,
            0,
            width,
            height
          );
          break;
        case 6:
          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(
            image,
            left / scale,
            top / scale,
            width / scale,
            height / scale,
            0,
            0,
            width,
            height
          );
          break;
        case 8:
          canvas.width = height;
          canvas.height = width;
          ctx.rotate((-90 * Math.PI) / 180);
          ctx.drawImage(
            image,
            left / scale,
            top / scale,
            width / scale,
            height / scale,
            -width,
            0,
            width,
            height
          );
          break;
        case 3:
          canvas.width = width;
          canvas.height = height;
          ctx.rotate((180 * Math.PI) / 180);
          ctx.drawImage(
            image,
            left / scale,
            top / scale,
            width / scale,
            height / scale,
            -width,
            -height,
            width,
            height
          );
          break;
      }
      let dataUrl = canvas.toDataURL("image/jpeg", this.cropperConfig.quality);
      self.callback(dataUrl);
      self.isShow = false;
    },
    getFocalPoint (point0, point1) {
      return {
        x: (point0.pageX + point1.pageX) / 2,
        y: (point0.pageY + point1.pageY) / 2
      };
    },
    touchStart (event) {
      var fingerCount = event.touches.length;
      if (fingerCount) {
        // 记录触摸初始位置
        let touchEvent = event.touches[0];
        this.touchPos = {
          x: touchEvent.clientX,
          y: touchEvent.clientY
        };
      }
      if (fingerCount >= 2) {
        // 获取两点距离、中点位置；两点距离old/new=放大倍数；中点位置，缩放中心；
        let point0 = event.touches[0];
        let point1 = event.touches[1];
        this.distance = getDinstance(point0, point1);
        this.touchPos = this.getFocalPoint(point0, point1);
        // 设置缩放倍数，
      }
    },
    touchMove (event) {
      // 根据触摸点位移，移动图片，重置触摸点位置
      var fingerCount = event.touches.length;
      var touchEvent = event.touches[0];
      if (fingerCount === 1) {
        let distX = touchEvent.pageX - this.touchPos.x;
        let distY = touchEvent.pageY - this.touchPos.y;
        let newX = this.imageState.left - distX;
        let newY = this.imageState.top - distY;
        let scale = this.imageState.scale;
        // alert(scale)
        let maxX = this.imageState.width * scale - this.cropBoxRect.width;
        let maxY = this.imageState.height * scale - this.cropBoxRect.height;
        this.imageState.left = newX < 0 ? 0 : newX > maxX ? maxX : newX;
        this.imageState.top = newY < 0 ? 0 : newY > maxY ? maxY : newY;
        this.touchPos.x = touchEvent.pageX;
        this.touchPos.y = touchEvent.pageY;
      } else if (fingerCount > 1) {
        let point0 = event.touches[0];
        let point1 = event.touches[1];
        let distance = getDinstance(point0, point1);
        let zoom = distance / this.distance;
        let scale = zoom * this.imageState.scale;
        let maxX = this.imageState.width * scale - this.cropBoxRect.width;
        let maxY = this.imageState.height * scale - this.cropBoxRect.height;
        let touchPos = this.getFocalPoint(point0, point1);
        let newX = zoom * (this.imageState.left + touchPos.x) - touchPos.x;
        let newY =
          zoom * (this.imageState.top - this.imgInitTop + touchPos.y) -
          touchPos.y +
          this.imgInitTop;
        // 限制缩放
        // 图片新位置:由中点位置确认;(新位置到中点)/(旧位置到中点)=(new scale)/(old scale)
        // newLeft - touchPos.x = (distance / this.distance) * (oldLetf - touchPos.x)
        // oldLeft = 0 - this.imageState.left
        // oldTop = imgInitTop - this.imageState.top
        this.distance = distance;
        if (scale < this.minScale) {
          this.imageState.scale = this.minScale;
        } else {
          this.imageState.scale = scale;
          this.imageState.left = newX < 0 ? 0 : newX > maxX ? maxX : newX;
          this.imageState.top = newY < 0 ? 0 : newY > maxY ? maxY : newY;
        }
        this.touchPos = touchPos;
      }
    },
    touchEnd (event) {
      // console.log('end')
    }
  }
};
</script>
