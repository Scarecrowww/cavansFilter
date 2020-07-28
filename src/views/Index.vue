<template>
  <div class="demo">
    <div class="upload-btn">
      <span>上传图片</span>
      <input type="file" class="file-btn" ref="fileBtn" @change="uploadImg" accept="image/*" />
    </div>
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
        <div class="title">素描</div>
        <div class="move-content content1">
          <img :src="img7" class="pic" />
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
    </div>
    <canvas id="myCanvas" style="display: none;"></canvas>
  </div>
</template>

<script>
export default {
  name: "Demo2",
  data() {
    return {
      imageUrl: require("../assets/bg.png"),
      // imageUrl: null,
      // oldImg: "", // 复古
      img1: "", // 1复古
      img2: "", // 2黑白
      img4: "", // 4浮雕
      img5: "", // 5反像
      img6: "", // 6模糊
      img7: "", // 素描
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
        that.drawImages(src, img.width, img.height, 4);
        that.drawImages(src, img.width, img.height, 6);
        that.drawImages(src, img.width, img.height, 7);
      };
    },
    async uploadImg() {
      var that = this;
      let inputFile = await that.$refs.fileBtn.files[0];
      let size = inputFile.size; // 读取文件的大小
      let maxSize = 7 * 1024 * 1024; // 限定最大文件的大小
      if (size > maxSize) {
        alert("不能上传大于7M的图");
      } else {
        let res;
        if (inputFile) {
          // 使用FileReader去读取file对象
          const reader = new FileReader();
          res = reader.readAsDataURL(inputFile);
          reader.onloadend = function () {
            // 把图片转base64
            var strBase64 = reader.result.substring(0);
            that.imageUrl = strBase64;
            let img = new Image();
            img.src = strBase64;
            img.onload = function () {
              // 处理图像
              that.getImg(that.imageUrl);
            };
          };
        } else {
          alert("上传失败");
        }
      }
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
        let imageData_length = imageData.data.length / 4;
        let d = imageData.data;
        let originData = [];
        // 输出老照片
        if (index == 1) {
          that.drawOldImg(d);
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img1 = imgurl;
        } else if (index == 2) {
          // 黑白
          that.discolor(d);
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img2 = imgurl;
        } else if (index == 4) {
          //浮雕的效果
          that.relief(d);
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img4 = imgurl;
        } else if (index == 6) {
          // 高斯模糊
          that.gaussBlurs(d, width, height, 20); //高斯模糊
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img6 = imgurl;
        } else if (index == 7) {
          // 素描
          that.sketch(imageData, 20, canvas, cxt);
          cxt.putImageData(imageData, 0, 0);
          let imgurl = canvas.toDataURL(); //获取图片的DataURL
          that.img7 = imgurl;
        }
      };
    },
    // 老照片
    drawOldImg(d) {
      for (var i = 0; i < d.length; i += 4) {
        var r = d[i];
        var g = d[i + 1];
        var b = d[i + 2];
        d[i] = r * 0.393 + g * 0.769 + b * 0.189; // red
        d[i + 1] = r * 0.349 + g * 0.686 + b * 0.168; // green
        d[i + 2] = r * 0.272 + g * 0.534 + b * 0.131; // blue
        d[i + 3] = 255; // 像素模糊值
      }
      return d;
    },
    // 素描
    sketch(imgData, radius, canvas, ctx) {
      let pixes = imgData.data;
      let width = imgData.width;
      let height = imgData.height;
      let copyPixes = "";
      this.discolor(pixes); //去色
      //复制一份
      ctx.clearRect(0, 0, width, height);
      ctx.putImageData(imgData, 0, 0);
      copyPixes = ctx.getImageData(0, 0, width, height).data;
      // 拷贝数组太慢
      this.invert(copyPixes); //反相
      this.gaussBlurs(copyPixes, width, height, radius); //高斯模糊
      this.dodgeColor(pixes, copyPixes); //颜色减淡
      return pixes;
    },
    // 把图像变成黑白色
    discolor(d) {
      for (var i = 0; i < d.length; i += 4) {
        let average = d[i] * 0.1 + d[i + 1] * 0.5 + d[i + 2] * 0.9;
        d[i + 0] = average; //红
        d[i + 1] = average; //绿
        d[i + 2] = average; //蓝
      }
      return d;
    },
    //浮雕
    relief(d) {
      for (var i = 0, j = 4; i < d.length; i += 4, j += 4) {
        if (j > d.length) {
          j = d.length - 4;
        }
        // 3.把相邻像素的同个通道进行差值运算,再加上中性灰的色值
        let r = Math.abs(d[i] - d[j] + 128),
          g = Math.abs(d[i + 1] - d[j + 1] + 128),
          b = Math.abs(d[i + 2] - d[j + 2] + 128);

        // 4.把结果通道的值进行求和并按权平均作为最终通道的值
        let val = parseInt(r * 0.3 + g * 0.6 + b * 0.1);
        d[i] = val;
        d[i + 1] = val;
        d[i + 2] = val;
      }
      return d;
    },
    // 把图片反相, 即将某个颜色换成它的补色
    invert(pixes) {
      for (var i = 0, len = pixes.length; i < len; i += 4) {
        pixes[i] = 255 - pixes[i]; //r
        pixes[i + 1] = 255 - pixes[i + 1]; //g
        pixes[i + 2] = 255 - pixes[i + 2]; //b
      }
      return pixes;
    },
    // 颜色减淡
    dodgeColor(basePixes, mixPixes) {
      for (var i = 0, len = basePixes.length; i < len; i += 4) {
        basePixes[i] =
          basePixes[i] + (basePixes[i] * mixPixes[i]) / (255 - mixPixes[i]);
        basePixes[i + 1] =
          basePixes[i + 1] +
          (basePixes[i + 1] * mixPixes[i + 1]) / (255 - mixPixes[i + 1]);
        basePixes[i + 2] =
          basePixes[i + 2] +
          (basePixes[i + 2] * mixPixes[i + 2]) / (255 - mixPixes[i + 2]);
      }
      return basePixes;
    },
    // 高斯模糊
    gaussBlurs(pixes, width, height, radius) {
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

      radius = Math.floor(radius) || 3;
      let sigma = radius / 3;

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
              gaussSum += gaussMatrix[j + radius];
            }
          }
          i = (y * width + x) * 4;
          pixes[i] = r / gaussSum;
          pixes[i + 1] = g / gaussSum;
          pixes[i + 2] = b / gaussSum;
        }
      }
      return pixes;
    },
  },
};
</script>

<style lang="less" scoped>
.demo {
  width: 100vw;
  height: 100vh;
  overflow: auto;
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
