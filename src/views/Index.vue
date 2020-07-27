<template>
  <div class="demo">
    <div>
      <div class="title">原图</div>
      <div class="move-content">
        <img :src="imageUrl" class="pic" />
      </div>
    </div>
    <div class="change-box">
      <div class="item">
        <div class="title">复古</div>
        <div class="move-content content1">
          <img :src="img1" class="pic" />
        </div>
      </div>
      <div class="item">
        <div class="title">黑白</div>
        <div class="move-content content1">
          <img :src="img2" class="pic" />
        </div>
      </div>
      <div class="item">
        <div class="title">高斯模糊</div>
        <div class="move-content content1">
          <img :src="img6" class="pic" />
        </div>
      </div>
      <div class="item">
        <div class="title">浮雕</div>
        <div class="move-content content1">
          <img :src="img4" class="pic" />
        </div>
      </div>
      <div class="item">
        <div class="title">底片</div>
        <div class="move-content content1">
          <img :src="img3" class="pic" />
        </div>
      </div>
      <div class="item">
        <div class="title">反像</div>
        <div class="move-content content1">
          <img :src="img5" class="pic" />
        </div>
      </div>
    </div>
    <canvas id="myCanvas" style="display: none;"></canvas>
  </div>
</template>

<script>
export default {
  name: "Demo2",
  data() {
    return {
      imageUrl: require("../assets/bg.jpg"),
      // imageUrl: null,
      // oldImg: "", // 复古
      img1: "", // 1复古
      img2: "", // 2黑白
      img3: "", // 3底片
      img4: "", // 4浮雕
      img5: "", // 5反像
      img6: "", // 6模糊
      imgIndex: 1, // 1复古 2黑白 3底片 4浮雕 5反像 6模糊
    };
  },
  mounted() {
    this.getImg(this.imageUrl);
  },
  methods: {
    getImg(src) {
      let that = this;
      let img = new Image();
      img.src = src;
      img.onload = function () {
        // 处理图像
        that.drawImages(src, img.width, img.height, 1);
        that.drawImages(src, img.width, img.height, 2);
        that.drawImages(src, img.width, img.height, 3);
        that.drawImages(src, img.width, img.height, 4);
        that.drawImages(src, img.width, img.height, 5);
        that.drawImages(src, img.width, img.height, 6);
      };
    },
    drawImages(imgSrc, width, height, index) {
      let that = this;
      var img = new Image();
      img.src = imgSrc;
      img.onload = function () {
        var canvas = document.querySelector("#myCanvas");
        var cxt = canvas.getContext("2d");
        if (width < 750 || height < 1334) {
          canvas.width = width;
          canvas.height = height;
        } else {
          // 一般手机的图像素都比较大，这里缩小一下精度，避免循环太久
          let times = Math.floor(width / 750);
          canvas.width = width / times;
          canvas.height = height / times;
        }
        // 在画布上绘制图片，主要是为了读取读取图片的每一个像素的rgb值
        cxt.drawImage(img, 0, 0, canvas.width, canvas.height);
        // 得到每一个像素的rgb值，得到的数组以4个为单位，%4=1的是r值, %4=2是g值, %4=3是b值，%4=像素模糊值(0-255）
        let imageData = cxt.getImageData(0, 0, canvas.width, canvas.height);
        console.log(imageData);
        let imageData_length = imageData.data.length / 4;
        let d = imageData.data;
        let originData = [];
        // 输出老照片
        if (index == 1) {
          for (var i = 0; i < d.length; i += 4) {
            var r = d[i];
            var g = d[i + 1];
            var b = d[i + 2];
            d[i] = r * 0.393 + g * 0.769 + b * 0.189; // red
            d[i + 1] = r * 0.349 + g * 0.686 + b * 0.168; // green
            d[i + 2] = r * 0.272 + g * 0.534 + b * 0.131; // blue
            d[i + 3] = 255; // 像素模糊值
          }
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img1 = imgurl;
        } else if (index == 2) {
          // 黑白
          for (var i = 0; i < d.length; i += 4) {
            let average = d[i] * 0.1 + d[i + 1] * 0.5 + d[i + 2] * 0.9;
            d[i + 0] = average; //红
            d[i + 1] = average; //绿
            d[i + 2] = average; //蓝
          }
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img2 = imgurl;
        } else if (index == 3) {
          //底片
          for (var i = 0; i < d.length; i += 4) {
            d[i] = 255 - d[i];
            d[i + 1] = 255 - d[i + 1];
            d[i + 2] = 255 - d[i + 2];
          }
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img3 = imgurl;
        } else if (index == 4) {
          //浮雕的效果
          for (let i = 0; i < d.length; i += 4) {
            d[i] = d[i] - d[i + 4] + 128;
            d[i + 1] = d[i + 1] - d[i + 5] + 128;
            d[i + 2] = d[i + 2] - d[i + 6] + 128;
            let avg = (d[i] + d[i + 1] + d[i + 2]) / 3;
            d[i] = avg;
            d[i + 1] = avg;
            d[i + 2] = avg;
          }
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img4 = imgurl;
        } else if (index == 5) {
          // 反像
          for (let i = 0; i < d.length; i += 4) {
            let r = d[i];
            let g = d[i + 1];
            let b = d[i + 2];
            let gray = 0.3 * r + 0.59 * g + 0.11 * b; //去色
            originData.push(gray);
            let anti_data = 255 - gray; //取反
            d[i] = anti_data;
            d[i + 1] = anti_data;
            d[i + 2] = anti_data;
          }
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img5 = imgurl;
        } else if (index == 6) {
          // 高斯模糊
          d = that.gaussBlur(imageData, width, height); //高斯模糊
          // 输出图片
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img6 = imgurl;
        }
      };
    },
    // 高斯模糊
    gaussBlur(img, width, height) {
      var pixes = img.data;
      var gaussMatrix = [],
        gaussSum = 0,
        x,
        y,
        r,
        g,
        b,
        a,
        i,
        j,
        k,
        len;

      var radius = 10;
      var sigma = 5;

      a = 1 / (Math.sqrt(2 * Math.PI) * sigma);
      b = -1 / (2 * sigma * sigma);
      //生成高斯矩阵
      for (i = 0, x = -radius; x <= radius; x++, i++) {
        g = a * Math.exp(b * x * x);
        gaussMatrix[i] = g;
        gaussSum += g;
      }
      //归一化, 保证高斯矩阵的值在[0,1]之间
      for (i = 0, len = gaussMatrix.length; i < len; i++) {
        gaussMatrix[i] /= gaussSum;
      }
      //x 方向一维高斯运算
      for (y = 0; y < height; y++) {
        for (x = 0; x < width; x++) {
          r = g = b = a = 0;
          gaussSum = 0;
          for (j = -radius; j <= radius; j++) {
            k = x + j;
            if (k >= 0 && k < width) {
              //确保 k 没超出 x 的范围
              //r,g,b,a 四个一组
              i = (y * width + k) * 4;
              r += pixes[i] * gaussMatrix[j + radius];
              g += pixes[i + 1] * gaussMatrix[j + radius];
              b += pixes[i + 2] * gaussMatrix[j + radius];
              // a += pixes[i + 3] * gaussMatrix[j];
              gaussSum += gaussMatrix[j + radius];
            }
          }
          i = (y * width + x) * 4;
          // 除以 gaussSum 是为了消除处于边缘的像素, 高斯运算不足的问题
          // console.log(gaussSum)
          pixes[i] = r / gaussSum;
          pixes[i + 1] = g / gaussSum;
          pixes[i + 2] = b / gaussSum;
          // pixes[i + 3] = a ;
        }
      }
      //y 方向一维高斯运算
      for (x = 0; x < width; x++) {
        for (y = 0; y < height; y++) {
          r = g = b = a = 0;
          gaussSum = 0;
          for (j = -radius; j <= radius; j++) {
            k = y + j;
            if (k >= 0 && k < height) {
              //确保 k 没超出 y 的范围
              i = (k * width + x) * 4;
              r += pixes[i] * gaussMatrix[j + radius];
              g += pixes[i + 1] * gaussMatrix[j + radius];
              b += pixes[i + 2] * gaussMatrix[j + radius];
              // a += pixes[i + 3] * gaussMatrix[j];
              gaussSum += gaussMatrix[j + radius];
            }
          }
          i = (y * width + x) * 4;
          pixes[i] = r / gaussSum;
          pixes[i + 1] = g / gaussSum;
          pixes[i + 2] = b / gaussSum;
        }
      }
      return img;
    },
  },
};
</script>

<style lang="less" scoped>
.demo {
  width: 100vw;
  height: 100vh;
  overflow: auto;
  .title {
    height: 2rem;
    line-height: 2rem;
    margin-left: 2rem;
  }
  .upload-btn {
    width: 10rem;
    height: 4rem;
    overflow: hidden;
    position: relative;
    background: bisque;
    border: 1px solid #f3f3f3;
    border-radius: 8px;
    line-height: 4rem;
    font-size: 1.5rem;
    text-align: center;
    .file-btn {
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      opacity: 0;
    }
  }
  .index-put {
    float: right;
    height: 3rem;
    line-height: 3rem;
    font-size: 2rem;
  }
  .move-content {
    width: 37.5rem;
    height: 20rem;
    margin-top: 10rem;
    background: #000;
    overflow: hidden;
    display: table-cell;
    vertical-align: middle;
    text-align: center;
    img {
      width: auto;
      height: auto;
      max-width: 100%;
      max-height: 100%;
    }
  }
  .change-box {
    position: relative;
    .item {
      .title {
        height: 2rem;
        line-height: 2rem;
        margin-left: 2rem;
      }
    }
  }
}
</style>
