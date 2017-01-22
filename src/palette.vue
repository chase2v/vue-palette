<template lang="html">
  <div class="c-palette">
    <div
      class="c-palette-preview"
      :style="color"
      @click.stop.prevent="onClickPreview"
      @keydown.prevent=""
      contentEditable="true"
      spellcheck="false"
      ref="preview">
      {{colorString}}
    </div>
    <div class="c-palette-main">
      <div class="cvs-gray"
        @mousedown.stop="onMousedownGray"
        @mousemove.stop="onMousemoveGray"
        @mouseup.stop="onMouseupGray"
        @mouseleave.stop="onMouseleaveGray">
        <div class="c-palette-slider" ref="sliderGray" v-show="isShowSliderGray"></div>
        <canvas ref="cvsMap"></canvas>
      </div>
      <div
        class="cvs-primary"
        @mousedown.stop="onMousedownPrimary"
        @mousemove.stop="onMousemovePrimary"
        @mouseup.stop="onMouseupPrimary"
        @mouseleave.stop="onMouseleavePrimary">
        <div class="c-palette-slider" ref="sliderPrimary"></div>
        <canvas ref="cvs"></canvas>
      </div>
    </div>
    <!-- <ul class="c-palete-controller">
      <li>RGB</li>
      <li>RGBA</li>
      <li>16进制</li>
    </ul> -->
  </div>
</template>

<script>
export default {
  data () {
    return {
      color: '',
      colorString: '',
      isShowSliderGray: false,
      sliderPrimary: 0,
      sliderGray: 0
    }
  },
  methods: {
    onClickPreview (e) {
      document.execCommand('selectAll')
    },
    onMousedownPrimary (e) {
      this.sliderPrimary = 1
      this.pickColorPrimary(1, e.offsetY)
      this.$refs.sliderPrimary.style.transform = 'translateY(' + e.offsetY + 'px)'
    },
    onMousemovePrimary (e) {
      if (this.sliderPrimary) {
        console.log(e.offsetY)
        this.pickColorPrimary(1, e.offsetY)

        this.$refs.sliderPrimary.style.transform = 'translateY(' + e.offsetY + 'px)'
      }
    },
    onMouseupPrimary (e) {
      this.sliderPrimary = 0
    },
    onMouseleavePrimary (e) {
      this.sliderPrimary = 0
    },
    onMousedownGray (e) {
      this.sliderGray = 1
      this.isShowSliderGray = true

      this.pickColorGray(e.offsetX, e.offsetY)
      this.$refs.sliderGray.style.transform = 'translate(-' + (200 - e.offsetX) + 'px, ' + e.offsetY + 'px)'
    },
    onMousemoveGray (e) {
      if (e.target.tagName !== 'DIV') {
        if (this.sliderGray) {
          // console.log(e.offsetY)
          // console.log(e.offsetX)
          this.pickColorGray(e.offsetX, e.offsetY)

          this.$refs.sliderGray.style.transform = 'translate(-' + (200 - e.offsetX) + 'px, ' + e.offsetY + 'px)'
        }
      }
    },
    onMouseupGray (e) {
      this.sliderGray = 0
    },
    onMouseleaveGray (e) {
      this.sliderGray = 0
    },
    pickColorGray (x, y) {
      let ctx = this.$refs.cvsMap.getContext('2d')
      let imageData = ctx.getImageData(x * 256 / 200, y * 256 / 200, 1, 1)
      this.color = `background-color: rgba(${imageData.data[0]}, ${imageData.data[1]}, ${imageData.data[2]}, ${imageData.data[3]})`
      this.colorString = `rgba(${imageData.data[0]}, ${imageData.data[1]}, ${imageData.data[2]}, ${imageData.data[3]})`
    },
    pickColorPrimary (x, y) {
      let imageData = this.ctx.getImageData(x * 50 / 20, y * 256 * 6 / 200, 1, 1)
      this.color = `background-color: rgba(${imageData.data[0]}, ${imageData.data[1]}, ${imageData.data[2]}, ${imageData.data[3]})`
      this.colorString = `rgba(${imageData.data[0]}, ${imageData.data[1]}, ${imageData.data[2]}, ${imageData.data[3]})`
      this.createImage([
        {
          r: 255,
          g: 255,
          b: 255,
          a: 255
        },
        {
          r: imageData.data[0],
          g: imageData.data[1],
          b: imageData.data[2],
          a: imageData.data[3]
        },
        {
          r: 0,
          g: 0,
          b: 0,
          a: 255
        },
        {
          r: 0,
          g: 0,
          b: 0,
          a: 255
        }
      ], 256, 256, this.$refs.cvsMap)
    },
    /**
     * 创建 ImageData
     *
     * @param colorArray {Array} [leftTopColor, rightTopColor, rightBottomColor, leftBottomColor]
     * @param width {Number} 图片宽度
     * @param height {Number} 图片高度
     * @param cvs {HTMLCanvasElement} canvas's domElm
     */
    createImage (colorArray, width, height, cvs) {
      cvs.width = width
      cvs.height = height

      let ctx = cvs.getContext('2d')
      let imageData = ctx.createImageData(width, height)
      let arr = this.createImageDataArray(colorArray, width, height)

      for (let i = 0; i < width * height * 4; i++) {
        imageData.data[i] = arr[i]
      }
      ctx.putImageData(imageData, 0, 0)
    },
    /**
     * 创建 ImageData 的 data 数组
     *
     * @param colorArray {Array} [leftTopColor, rightTopColor, rightBottomColor, leftBottomColor]
     * @param width {Number} 图片宽度
     * @param height {Number} 图片高度
     */
    createImageDataArray (colorArray, width, height) {
      let arr = []
      let gapHead = { // 计算首列像素差距
        r: colorArray[0].r - colorArray[3].r,
        g: colorArray[0].g - colorArray[3].g,
        b: colorArray[0].b - colorArray[3].b,
        a: colorArray[0].a - colorArray[3].a
      }
      let gapTail = { // 计算尾列像素差距
        r: colorArray[1].r - colorArray[2].r,
        g: colorArray[1].g - colorArray[2].g,
        b: colorArray[1].b - colorArray[2].b,
        a: colorArray[1].a - colorArray[2].a
      }
      for (let i = 0; i < height; i++) {
        let colorStart = { // 计算每行的起始像素
          r: colorArray[0].r - Math.ceil((gapHead.r / (height - 1)) * i),
          g: colorArray[0].g - Math.ceil((gapHead.g / (height - 1)) * i),
          b: colorArray[0].b - Math.ceil((gapHead.b / (height - 1)) * i),
          a: colorArray[0].a - Math.ceil((gapHead.a / (height - 1)) * i)
        }
        let colorEnd = { // 计算每行的结束像素
          r: colorArray[1].r - Math.ceil((gapTail.r / (height - 1)) * i),
          g: colorArray[1].g - Math.ceil((gapTail.g / (height - 1)) * i),
          b: colorArray[1].b - Math.ceil((gapTail.b / (height - 1)) * i),
          a: colorArray[1].a - Math.ceil((gapTail.a / (height - 1)) * i)
        }
        for (let j = 0; j < width; j++) {
          let color = { // 计算每一像素点
            r: colorStart.r - Math.ceil(((colorStart.r - colorEnd.r) / (width - 1)) * j),
            g: colorStart.g - Math.ceil(((colorStart.g - colorEnd.g) / (width - 1)) * j),
            b: colorStart.b - Math.ceil(((colorStart.b - colorEnd.b) / (width - 1)) * j),
            a: colorStart.a - Math.ceil(((colorStart.a - colorEnd.a) / (width - 1)) * j)
          }
          for (let k in color) {
            arr.push(color[k])
          }
        }
      }

      return arr
    }
  },
  mounted () {
    this.$refs.cvs.width = 50
    this.$refs.cvs.height = 256 * 6
    let ctx = this.$refs.cvs.getContext('2d')
    this.ctx = ctx

    let imageData = ctx.createImageData(50, 256 * 6)
    createDataPrimary(50)
    console.log(imageData)
    ctx.putImageData(imageData, 0, 0)

    this.pickColorPrimary(0, 0)

    // 生成 imageData 的数据集
    // w: 图片的宽度
    function createDataPrimary (w) {
      let color = {
        r: 255,
        g: 0,
        b: 0,
        a: 255
      }
      let j = 0

      _loop(1, 'g')
      _loop(0, 'r')
      _loop(1, 'b')
      _loop(0, 'g')
      _loop(1, 'r')
      _loop(0, 'b')

      function _loop (type, c) {
        if (type) {
          __increase()
        } else {
          __decrease()
        }

        function __increase () {
          for (; color[c] <= 255; color[c]++) {
            __same()
          }
          color[c] = 255
        }

        function __decrease () {
          for (; color[c] >= 0; color[c]--) {
            __same()
          }
          color[c] = 0
        }

        function __push (v) {
          imageData.data[j] = v
          j++
        }

        function __same () {
          for (let i = w; i > 0; i--) {
            __push(color.r)
            __push(color.g)
            __push(color.b)
            __push(color.a)
          }
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.c-palette {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);

  width: 250px;
  height: auto;

  // border: 1px solid #ccc;
  background-color: rgb(235, 235, 235);

  .c-palette-preview {
    width: 100%;
    height: 50px;
    line-height: 50px;

    cursor: pointer;

    color: #fff;
    text-decoration: none;
  }

  .c-palette-main {
    box-sizing: border-box;
    width: 100%;
    height: 220px;
    padding: 10px;
  }

  .c-palete-controller {
    display: flex;
    flex-flow: row nowrap;
    width: 100%;
    height: 36px;
    line-height: 36px;

    background-color: #ccc;
    color: #fff;
    font-size: 12px;
    list-style: none;

    li {
      flex: 1 1 auto;
      border-right: 1px solid rgb(235, 235, 235);
      cursor: pointer;

      &:last-child {
        border-right: none;
      }
    }
  }

  .cvs-primary {
    position: relative;
    margin: 0 0 0 10px;

    display: inline-block;
    width: 20px;
    height: 200px;

    canvas {
      width: 100%;
      height: 100%;
    }

    .c-palette-slider {
      position: absolute;
      top: -4px;
      left: -6px;

      border-left: 6px solid #000;
      border-top: 4px solid transparent;
      border-bottom: 4px solid transparent;
    }
  }
  .cvs-gray {
    position: relative;
    float: left;

    display: inline-block;
    width: 200px;
    height: 200px;

    canvas {
      width: 100%;
      height: 100%;
      cursor: crosshair;
    }

    .c-palette-slider {
      position: absolute;
      right: -5px;
      top: -5px;

      width: 10px;
      height: 10px;
      border-radius: 50%;

      cursor: crosshair;
      background-color: #000;

      &:before {
        position: absolute;
        left: 3px;
        top: 3px;

        content: '';

        width: 4px;
        height: 4px;
        border-radius: 50%;

        background-color: #ccc;
      }
    }
  }
}
</style>
