<template>
  <div class="canvas-container" >
    <div class="canvas-content">
      <canvas
              id="canvas"
              class="canvas-item"
              :width="canvasWidth"
              :height="canvasHeight"

              @mousedown="canvasClick($event)"
              @mouseup="stopdrag($event)"
              @mouseout="stopdrag($event)"
              @mousemove="drag($event)"
              @dblclick="dblik($event)"

              @touchstart="canvasClick($event)"
              @touchmove="drag($event)"
              @touchend="stopdrag($event)"
      />
    </div>
  </div>
</template>


<script>
  export default {
    name: 'Canvas',
    data() {
      return {
        canvasWidth: document.documentElement.clientWidth-10,
        canvasHeight: document.documentElement.clientHeight-22,
        select : -1,
        isDragging : false,
        lastx: 0,
        lasty: 0,
        points : [
          {
            p: [{x: 100, y: 50}, {x: 150, y: 50}, {x: 150, y: 150}, {x: 200, y: 150}, {x: 200, y: 200}, {x: 100, y: 200}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 6,
            xmin : 100,
            xmax : 200,
            ymin : 50,
            ymax : 200
          },
          {
            p: [{x: 100, y: 0}, {x: 100, y: 50}, {x: 300, y: 50}, {x: 300, y: 0}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 4,
            xmin : 100,
            xmax : 300,
            ymin : 0,
            ymax : 50
          },
          {
            p: [{x: 150, y: 200}, {x: 200, y: 200}, {x: 200, y: 250},{x: 250, y: 250}, {x: 250, y: 300}, {x: 100, y: 300},{x: 100, y: 250}, {x: 150, y: 250}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 8,
            xmin : 100,
            xmax : 250,
            ymin : 200,
            ymax : 300
          },
          {
            p: [{x: 150, y: 300}, {x: 200, y: 300}, {x: 200, y: 400},{x: 150, y: 400}, {x: 150, y: 450}, {x: 100, y: 450},{x: 100, y: 350}, {x: 150, y: 350}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 8,
            xmin : 100,
            xmax : 200,
            ymin : 300,
            ymax : 450
          }
        ],
        points2 : [
          {
            p: [{x: 100, y: 50}, {x: 150, y: 50}, {x: 150, y: 150}, {x: 200, y: 150}, {x: 200, y: 200}, {x: 100, y: 200}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 6,
            xmin : 100,
            xmax : 200,
            ymin : 50,
            ymax : 200
          },
          {
            p: [{x: 100, y: 0}, {x: 100, y: 50}, {x: 300, y: 50}, {x: 300, y: 0}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 4,
            xmin : 100,
            xmax : 300,
            ymin : 0,
            ymax : 50
          },
          {
            p: [{x: 150, y: 200}, {x: 200, y: 200}, {x: 200, y: 250},{x: 250, y: 250}, {x: 250, y: 300}, {x: 100, y: 300},{x: 100, y: 250}, {x: 150, y: 250}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 8,
            xmin : 100,
            xmax : 250,
            ymin : 200,
            ymax : 300
          },
          {
            p: [{x: 150, y: 300}, {x: 200, y: 300}, {x: 200, y: 400},{x: 150, y: 400}, {x: 150, y: 450}, {x: 100, y: 450},{x: 100, y: 350}, {x: 150, y: 350}],
            color: "#ffd58b",
            click: false,
            drag: false,
            types: 8,
            xmin : 100,
            xmax : 200,
            ymin : 300,
            ymax : 450
          }
        ]
      }
    },
    watch: {
      //监听长度和宽度 自适应大小
      canvasHeight (val) {
        // 为了避免频繁触发resize函数导致页面卡顿，使用定时器
        if (!this.timer) {
          // 一旦监听到的screenWidth值改变，就将其重新赋给data里的screenWidth
          this.canvasHeight = val
          this.timer = true
          let that = this
          setTimeout(function () {
            // 打印screenWidth变化的值
            console.log(that.canvasHeight)
            that.draw()
            that.timer = false
          }, 400)
        }
      },
      canvasWidth (val) {
        // 为了避免频繁触发resize函数导致页面卡顿，使用定时器
        if (!this.timer) {
          // 一旦监听到的screenWidth值改变，就将其重新赋给data里的screenWidth
          this.canvasWidth = val
          this.timer = true
          let that = this
          setTimeout(function () {
            // 打印screenWidth变化的值
            console.log(that.canvasWidth)
            that.draw()
            that.timer = false
          }, 400)
        }
      }
    },

    mounted() {
      const canvas = document.querySelector('#canvas')
      this.context = canvas.getContext('2d')
      this.draw()

      const that = this
      window.onresize = () => {
        return (() => {
          that.canvasHeight = document.documentElement.clientHeight-22
          that.canvasWidth = document.documentElement.clientWidth-10
        })()
      }
    },
    methods: {
      draw(){
        // 清除画布，准备绘制
        this.context.clearRect(0, 0, canvas.width, canvas.height);
        //遍历数组，以每个点为起点画图
        for (var i = 0; i < this.points.length; i++) {
          this.context.beginPath();
          this.context.moveTo(this.points[i].p[0].x, this.points[i].p[0].y);
          for (var j = 0; j < this.points[i].p.length; j++) {
            this.context.lineTo(this.points[i].p[j].x, this.points[i].p[j].y);
          }
          this.context.lineTo(this.points[i].p[0].x, this.points[i].p[0].y);
          this.context.strokeStyle = "black";
          this.context.lineWidth = "3";
          this.context.fillStyle = this.points[i].color;
          this.context.stroke();
          this.context.fill();
          this.context.closePath();
        }
      },
      canvasClick(e){
        if(e.touches){ //判断是鼠标还是触摸
          console.log('touchdown', e)
          var clickX = e.touches[0].pageX - canvas.offsetLeft;
          var clickY = e.touches[0].pageY - canvas.offsetTop;
        }
        else{
          console.log('mousedown', e)
          var clickX = e.pageX - canvas.offsetLeft;
          var clickY = e.pageY - canvas.offsetTop;
        }


        for (var i = 0; i < this.points.length; i++) {
          var flag = false;
          if (this.points[i].types == 6) {
            flag = this.anglecompult(this.points[i].p, clickX, clickY);
          } else if (this.points[i].types == 4) {
            flag = this.reactcompult(this.points[i].p, clickX, clickY);
          } else if (this.points[i].types == 8 && i == 2) {
            flag = this.thirdcompult(this.points[i].p, clickX, clickY);
          } else if (this.points[i].types == 8 && i == 3) {
            flag = this.fouthcompult(this.points[i].p, clickX, clickY)
          }
          if (flag == true) {
            this.isDragging = true;
            this.points[i].click = true;
            this.points[i].drag = true;
            this.select = i;
            this.lastx = clickX;
            this.lasty = clickY;
            console.log(this.lastx, this.lasty);
          } else {
            this.points[i].click = false;
            this.points[i].drag = false;
          }
        }
      },
      stopdrag(e){
        console.log("stopmove");
        //this.lasty = 0;
        // this.lastx = 0;
        if(this.select != -1) {
          for (var j = 0; j < this.points[this.select].p.length; j++) {  //结束后吧points2的值更新
            this.points2[this.select].p[j].x = this.points[this.select].p[j].x;
            this.points2[this.select].p[j].y = this.points[this.select].p[j].y;
            this.points2[this.select].xmin = this.points[this.select].xmin;
            this.points2[this.select].xmax = this.points[this.select].xmax;
            this.points2[this.select].ymin = this.points[this.select].ymin;
            this.points2[this.select].ymax = this.points[this.select].ymax;
          }
        }
        this.isDragging = false;
      },
      dblik(e){   //旋转90度
        if(e.touches){
          var x = e.touches[0].pageX - canvas.offsetLeft;
          var y = e.touches[0].pageY - canvas.offsetTop;
        }
        else{
          var x = e.pageX - canvas.offsetLeft;
          var y = e.pageY - canvas.offsetTop;
        }

        // 将圆圈移动到鼠标位置
        var changdu = this.points[this.select].p.length;

        var yuanX = this.points[this.select].p[0].x;
        var yuanY = this.points[this.select].p[0].y;

        var yuanX2 = this.points2[this.select].p[0].x;
        var yuanY2 = this.points2[this.select].p[0].y;
        var xianx;
        var xianx2;
        var xiany;
        var xiany2;
        var xiebian;
        for (var j = 1; j < changdu; j++) {
          xianx = (this.points[this.select].p[j].x - yuanX) * (Math.cos((2 * Math.PI / 360) * 90)) - (this.points[this.select].p[j].y - yuanY) * (Math.sin((2 * Math.PI / 360) * 90)) + yuanX;
          xiany = (this.points[this.select].p[j].x - yuanX) * (Math.sin((2 * Math.PI / 360) * 90)) + (this.points[this.select].p[j].y - yuanY) * (Math.cos((2 * Math.PI / 360) * 90)) + yuanY;
          this.points[this.select].p[j].x = xianx;
          this.points[this.select].p[j].y = xiany;

          xianx2 = (this.points2[this.select].p[j].x - yuanX2) * (Math.cos((2 * Math.PI / 360) * 90)) - (this.points2[this.select].p[j].y - yuanY2) * (Math.sin((2 * Math.PI / 360) * 90)) + yuanX2;
          xiany2 = (this.points2[this.select].p[j].x - yuanX2) * (Math.sin((2 * Math.PI / 360) * 90)) + (this.points2[this.select].p[j].y - yuanY2) * (Math.cos((2 * Math.PI / 360) * 90)) + yuanY2;
          this.points2[this.select].p[j].x = xianx2;
          this.points2[this.select].p[j].y = xiany2;
        }
        // 更新画布
        this.draw();

        //}
      },
      drag(e) {
        // 判断圆圈是否开始拖拽
        console.log("move")
        if (this.isDragging == true) {
          // 判断拖拽对象是否存在

          // 取得鼠标位置
          if(e.touches){
            var x = e.touches[0].pageX - canvas.offsetLeft;
            var y = e.touches[0].pageY - canvas.offsetTop;
          }
          else{
            var x = e.pageX - canvas.offsetLeft;
            var y = e.pageY - canvas.offsetTop;
          }
          // console.log(x,y);

          var changdu = this.points[this.select].p.length;

          var xx = x - this.lastx;
          var yy = y - this.lasty;
          // if(this.bordor(xx, yy)) {   //判断是否到四周
          if (this.points2[this.select].xmin + xx > 0 && this.points2[this.select].xmax + xx < this.canvasWidth && this.points2[this.select].ymin + yy > 0 && this.points2[this.select].ymax + yy < this.canvasHeight) {
            for (var j = 0; j < changdu; j++) {//将图形拖动到鼠标相应位置
              this.points[this.select].p[j].x = this.points2[this.select].p[j].x + xx;
              this.points[this.select].p[j].y = this.points2[this.select].p[j].y + yy;
              this.points[this.select].xmin = this.points2[this.select].xmin + xx;
              this.points[this.select].xmax = this.points2[this.select].xmax + xx;
              this.points[this.select].ymin = this.points2[this.select].ymin + yy;
              this.points[this.select].ymax = this.points2[this.select].ymax + yy;
            }
          }
          //  }
          // 更新画布
          this.draw();

        }
      },

      //判断是否在图形内
      anglecompult(anglepostions, x, y){      //判断是否在第一个图形里面 解决方案为切割为两个矩形 然后再用reactcompult判断
        var test1 = [{x:anglepostions[0].x,y:anglepostions[0].y}, {x:anglepostions[1].x,y:anglepostions[1].y}, {x:anglepostions[2].x,y:anglepostions[2].y}, {x:anglepostions[0].x,y:anglepostions[2].y}];
        var test2 = [{x:anglepostions[0].x,y:anglepostions[2].y}, {x:anglepostions[3].x,y:anglepostions[3].y}, {x:anglepostions[4].x,y:anglepostions[4].y}, {x:anglepostions[5].x,y:anglepostions[5].y}];
        if(this.reactcompult(test1, x, y) || this.reactcompult(test2, x, y)){
          return true;
        }
        else {
          return false;
        }
      },
      reactcompult(reactpostions, x, y){   //判断是否在第二个图形里面  判断是否在矩形内 这个判断只能针对非斜45放置的矩形
        var xmin, xmax, ymin, ymax;
        xmin = xmax = reactpostions[0].x;
        ymin = ymax = reactpostions[0].y;
        for (var i = 0; i < reactpostions.length; i++) {
          if (reactpostions[i].x < xmin) {
            xmin = reactpostions[i].x;
          }
          if (reactpostions[i].x > xmax) {
            xmax = reactpostions[i].x;
          }
          if (reactpostions[i].y < ymin) {
            ymin = reactpostions[i].y;
          }
          if (reactpostions[i].y > ymax) {
            ymax = reactpostions[i].y;
          }
        }

        if (xmin < x && xmax > x && ymin < y && ymax > y) {
          return true;
        } else {
          return false;
        }
      },
      thirdcompult(thirdpostions, x, y){
        var test1 = [{x:thirdpostions[0].x,y:thirdpostions[0].y}, {x:thirdpostions[1].x,y:thirdpostions[1].y}, {x:thirdpostions[2].x,y:thirdpostions[2].y}, {x:thirdpostions[7].x,y:thirdpostions[7].y}];
        var test2 = [{x:thirdpostions[3].x,y:thirdpostions[3].y}, {x:thirdpostions[4].x,y:thirdpostions[4].y}, {x:thirdpostions[5].x,y:thirdpostions[5].y}, {x:thirdpostions[6].x,y:thirdpostions[6].y}];
        if(this.reactcompult(test1, x, y) || this.reactcompult(test2, x, y)){
          return true;
        }
        else {
          return false;
        }
      },
      fouthcompult(fouthpostions, x, y){
        var test1 = [{x:fouthpostions[0].x,y:fouthpostions[0].y}, {x:fouthpostions[1].x,y:fouthpostions[1].y}, {x:fouthpostions[2].x,y:fouthpostions[2].y}, {x:fouthpostions[3].x,y:fouthpostions[3].y}];
        var test2 = [{x:fouthpostions[4].x,y:fouthpostions[4].y}, {x:fouthpostions[5].x,y:fouthpostions[5].y}, {x:fouthpostions[6].x,y:fouthpostions[6].y}, {x:fouthpostions[7].x,y:fouthpostions[7].y}];
        if(this.reactcompult(test1, x, y) || this.reactcompult(test2, x, y)){
          return true;
        }
        else {
          return false;
        }
      },

      handleCommand(command) {
        this.$message('click on item ' + command);
      }

    }
  }
</script>

<style>
  .el-dropdown-link {
    cursor: pointer;
    color: #409EFF;
  }
  .el-icon-arrow-down {
    font-size: 12px;
  }
</style>