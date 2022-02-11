<template>
  <div class="pintu_area">
    <div v-for="(part, index) in partList" :key="'part' + index" class="pintu_part"></div>
    <div v-for="(img, index) in imgList" :key="index" :style="{'backgroundPosition': `${img.x * 100}px ${img.y * 100}px`,
            'top': Math.random() * 500 + 150 + 'px',
            'left': Math.random() * 600 + 800 + 'px',
            'transform': `rotate(${img.rotate}turn)`,
            'backgroundImage': `url(${this.image})`}" class="part_img"
         @mousedown="moveImg($event,index)" @contextmenu.prevent="rotateImg($event, index)">
    </div>
    <img class="thumb" :src="image">
    <input type="file" accept=".jpg,.png" @change="chooseImg($event)" title="点击更换图片">
  </div>

</template>


<script>
  export default {
    name: 'Canvas',
    data() {
      return {
        partList: [], // 卡槽
        imgList: [], // 拼图块
        rotateStatus: true, // 拼图旋转状态
        image: 'https://img0.baidu.com/it/u=3429479819,3767560213&fm=253&fmt=auto&app=138&f=JPEG?w=753&h=500' // 拼图图片路径
      }
    },
    mounted() {
      this.initGame()
    },
    methods: {
      initGame() { // 游戏初始化
        this.imgList = []
        this.partList = []
        for (let i = 0; i < 48; i++) { // 循环遍历生成拼图数组与对应卡槽数组
          this.imgList.push({ // 拼图数组
            x: 8 - i % 8, // 第几列
            y: 6 - parseInt((i / 8)), // 第几行
            //backgroundimage : `url(${this.image})`,
            rotate: parseInt(Math.random() * 4) * 0.25 // 初始旋转角度，单位为turn
          })
          this.partList.push({ // 卡槽数组
            x: 8 - i % 8, // 第几列
            y: 6 - parseInt((i / 8)), //第几行
            fill: false, // 是否包含一个拼图块
            check: false // 是否放入正确的拼图
          })
        }
        this.imgList = this.imgList.sort(() => Math.random() - 0.5) // 打乱拼图数组顺序（可以不要）
      },
      moveImg(e, index) { // 移动拼图
        const _this = this
        if (!this.partList[index].check && e.button < 1) { // 判断当前卡片未放置到正确位置切是左键点击
          const el = e.target
          el.style.transition = "none"
          el.style.zIndex = 99
          const sX = e.clientX - el.offsetLeft
          const sY = e.clientY - el.offsetTop
          const elLeft = parseInt(el.style.left) / 100
          const elTop = parseInt(el.style.top) / 100
          const elPart = _this.partList.find((elem) => elem.x == (8 - elLeft) && elem.y == (6 - elTop))
          if (elPart) { // 此处是判断将拼图从错误的卡槽里移除时，清除掉卡槽的填充状态
            const partIndex = _this.partList.indexOf(elPart)
            _this.partList[partIndex].fill = false
          }
          document.onmousemove = (e) => { // 拼图随鼠标移动
            const eX = e.clientX - sX
            const eY = e.clientY - sY
            el.style.left = eX + 'px'
            el.style.top = eY + 'px'
          }
          document.onmouseup = (e) => { // 移动结束时的操作
            document.onmousemove = null
            el.style.transition = "all 1s"
            el.style.zIndex = 1
            const left = parseInt(el.style.left) / 100
            const top = parseInt(el.style.top) / 100
            if (left < 8 && top < 6) { // 判断拼图移到了卡槽区域
              const toLeft = left < 7.5 ? Math.round(left) : 7
              const toTop = top < 5.5 ? Math.round(top) : 5
              const part = _this.partList.find((elem) => elem.x == (8 - toLeft) && elem.y == (6 - toTop))
              if (!part.fill) { // 如果卡槽是空的，将拼图移入离它最近的卡槽里并检查是否是正确的放入正确的卡槽里
                const partIndex = _this.partList.indexOf(part)
                _this.partList[partIndex].fill = true
                el.style.left = toLeft * 100 + 'px'
                el.style.top = toTop * 100 + 'px'
                _this.checkImg(_this, el, index)
              }
            }
          }
        }
      },
      rotateImg(e, index) { // 旋转当前拼图
        if (this.rotateStatus && !this.partList[index].check) { // 拼图可以旋转并且并未正确放置
          const el = e.target
          this.rotateStatus = false
          el.style.transition = "all 1s"
          let angle = this.getAngle(el)
          if (angle < 0) { // 计算出的角度为270度时会返回-0.25，将其转为0.75以实现正确的旋转
            angle = 0.75
          }
          el.style.transform = `rotate(${angle + 0.25}turn)`
          const _this = this
          setTimeout(() => {
            if (angle + 0.25 == 1) { // 如果旋转了360度将其重置为0度，不然再次旋转会变成逆时针旋转，然后检查该拼图是否正确的放置在正确的卡槽里
              el.style.transition = "none"
              el.style.transform = `rotate(0turn)`
              this.checkImg(this, el, index)
            }
            _this.rotateStatus = true
          }, 1000);
        }
      },
      getAngle(el) { // 判断当前元素旋转角度，此段方法是搜出来的，对搜索的结果进行了修改，获取的值以turn为角度单位
        const st = window.getComputedStyle(el, null)
        const tr = st.getPropertyValue("-webkit-transform") ||
                st.getPropertyValue("-moz-transform") ||
                st.getPropertyValue("-ms-transform") ||
                st.getPropertyValue("-o-transform") ||
                st.getPropertyValue("transform") ||
                "FAIL"
        const values = tr.split('(')[1].split(')')[0].split(',')
        const a = values[0]
        const b = values[1]
        return Math.round(Math.atan2(b, a) * (180 / Math.PI)) / 360
      },
      checkImg(_this, el, index) { // 检查图片位置是否正确
        const left = parseInt(el.style.left) / 100
        const top = parseInt(el.style.top) / 100
        if (left < 8 && top < 6) { // 判断拼图移到了卡槽区域
          const toLeft = left < 7.5 ? Math.round(left) : 7
          const toTop = top < 5.5 ? Math.round(top) : 5
          const img = _this.imgList[index]
          if (img.x == (8 - toLeft) && img.y == (6 - toTop) && _this.getAngle(el) == 0) { // 拼图的x，y与当前所在卡槽的位置对应上且拼图角度是正确的
            _this.partList[index].check = true
            // _this.$refs.ding.play() // 游戏音效
            el.style.animation = 'checked 2s'
            el.style.zIndex = 0
          }
          if (_this.partList.every((elem) => elem.check)) {
            // _this.$refs.jubilate.play() // 游戏音效
            alert("已完成拼图")
          }
        }
      },
      chooseImg(e) { // 点击上传图片更新图片
        const file = event.target.files[0]
        const reader = new FileReader()
        const _this = this
        reader.readAsDataURL(file)
        reader.onload = function () { // 此处因this指向问题采用function声明函数，并未使用箭头函数
          _this.partList = []
          for (let i = 0; i < 48; i++) { // 循环遍历生成拼图数组与对应卡槽数组，重置卡槽，不重置的话，之前正确放置的拼图块会出现不可移动的问题
            _this.partList.push({ // 卡槽数组
              x: 8 - i % 8,
              y: 6 - parseInt((i / 8)),
              fill: false,
              check: false
            })
          }
          _this.image = this.result
        }
      },
      randomStyle(img) { // 规律生成拼图块，随机生成位置与旋转角度，更新图片
        console.log(this.image)
        return (img) => {
          return {
            'backgroundPosition': `${img.x * 100}px ${img.y * 100}px`,
            'top': Math.random() * 500 + 150 + 'px',
            'left': Math.random() * 600 + 800 + 'px',
            'transform': `rotate(${img.rotate}turn)`,
            'backgroundImage': `url(${this.image})`
          }
        }
      }

    }
  }
</script>

<style>
  .pintu_area {
    display: flex;
    flex-wrap: wrap;
    width: 800px;
    height: 600px;
    background-color: rgba(0, 0, 0, 0.4);
  }

  .pintu_part {
    width: 100px;
    height: 100px;
    border: 1px solid #fff;
  }

  .part_img {
    width: 100px;
    height: 100px;
    background-size: 800px 600px;
    position: absolute;
  }

  .thumb,
  input {
    width: 200px;
    height: 150px;
    position: absolute;
    top: 0;
    right: 0;
  }

  input {
    opacity: 0;
  }

  @keyframes checked {
    50% {
      box-shadow: 0 0 20px #ffff00;
    }

    to {
      box-shadow: none;
    }
  }
</style>
