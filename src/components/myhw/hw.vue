<template>
  <div>
    <el-container>
      <el-header>嵌漫</el-header>
      <el-container>
        <el-aside width="200px">
          <el-button @click="erase">清空</el-button><br>
          <el-button @click="textEdit">入字</el-button><br>
          <span>操作模式</span><br>
          <el-radio-group v-model="model" size="small">
            <el-radio label="1">划线</el-radio>
          </el-radio-group><br>
          <span>线条颜色</span>
          <el-select v-model="lineColor" placeholder="颜色" @change="colorChange">
            <el-option label="black" value="black"></el-option>
            <el-option label="red" value="red"></el-option>
            <el-option label="green" value="green"></el-option>
            <el-option label="blue" value="blue"></el-option>
            <el-option label="orange" value="orange"></el-option>
          </el-select>
          <div :key="key" v-for="(value, key) in lineData">
            {{key}}:{{value.length}}
          </div>
        </el-aside>
        <el-container>
          <el-main>
            <!--@click="click($event)"-->
            <canvas ref="canvas"
                    id="canvas" width="800" height="600"
                    @mousedown="mouseDown($event)"
                    @mouseup="mouseUp($event)"
                    @mousemove="mouseMove($event)">
            </canvas>
          </el-main>
          <el-footer>Footer</el-footer>
        </el-container>
      </el-container>
    </el-container>
    <div class="hide">
      <el-dialog
        :title="lineColor"
        :visible.sync="dialogVisible"
        width="50%"
        :before-close="handleClose">
        <div style="height: 200px">
          <span>请输入文字</span>
          <el-input v-model="textSize" placeholder="请输入字体大小(px)"></el-input>
          <div>
            <textarea ros="5" cols="50" v-model="content">
          </textarea>
          </div>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="textOK">确 定</el-button>
        </span>
      </el-dialog>
    </div>

  </div>
</template>

<script>
  function Line(color,no,begin) {
    this.color= color
    this.begin = begin
    //this.end = end
    this.no = no
    this.vLen=function () {
      return Math.abs(this.begin.y-this.end.y)
    }
    this.top=function () {
      return this.begin.y < this.end.y ? this.begin: this.end
    }
    this.bottom=function () {
      return this.begin.y > this.end.y ? this.begin: this.end
    }
  }
  export default {
    name: "hw",
    data() {
      return {
        textSize: 30,
        lineData:{
        },
        mousedown: {
        },
        editInfo: {},
        drawLineMode: 'y',
        ctx: undefined,
        lineColor: 'black',
        imageData: [],
        model: '1',
        dialogVisible: false,
        content: '',
        dragging: false,//是否在拖动
      }
    },
    mounted() {
      this.ctx=this.$refs.canvas.getContext('2d');
    },
    methods: {
      textOK(){
        let lineList = this.lineData[this.lineColor]
        console.log(lineList)
        let index = 0
        let count = 0
        let line  = lineList[index]
        this.ctx.font = this.textSize+"px serif";
        for(let i =0; i<this.content.length;i++){
          let y=this.textSize*count+line.top().y
          this.ctx.fillText(this.content[i], line.top().x, y);
          count++
          if(index<lineList.length-1 && y>=line.bottom().y ){
            line = lineList[++index]
            count = 0
            console.log(line)
          }
        }
        this.dialogVisible = false
      },
      textEdit(){
        this.content = ''
        this.dialogVisible = true
      },
      erase(){
        this.ctx.clearRect(0, 0, canvas.width, canvas.height);
        saveDrawingSurface();
      },
      colorChange(){
        this.ctx.strokeStyle = this.lineColor
      },
      canvasLocate(e) {
        const x = e.clientX - this.$refs.canvas.getBoundingClientRect().left;
        const y = e.clientY - this.$refs.canvas.getBoundingClientRect().top;
        return {
          x, y
        }
      },
      draw() {
        const ctx = this.$refs.canvas.getContext('2d');
        ctx.font = "48px serif";
        ctx.strokeText("Hello world", 10, 50);
      },
      //保存当前的canvas上的数据
      saveDrawingSurface() {
        const ctx = this.$refs.canvas.getContext('2d');
        this.imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      },
      //恢复canvas的数据，主要用来显示最新的线段，擦除原来的线段
      restoreDrawingSurface() {
        const ctx = this.$refs.canvas.getContext('2d');
        ctx.putImageData(this.imageData,
          0, 0, 0, 0, canvas.width, canvas.height
        );
      },
      //更新
      updateRubberband(loc) {
        //此处在《HTML5 canvas核心技术——图形、动画与游戏开发》一书中
        //updateRubberbandRectangle方法是没有注释的，但是我不懂要这个
        //方法有什么作用，注释之后也不影响，话说我也不用话什么矩形哇
        //有知道这个方法在这里是做什么的同学在下方评论一下告知哈
        //this.updateRubberbandRectangle(loc);
        this.drawRubberbandShape(loc);
      },
      //画最新的线条
      drawRubberbandShape(loc) {
        const context = this.$refs.canvas.getContext('2d');
        context.beginPath();
        context.moveTo(this.mousedown.x, this.mousedown.y);
        /*switch (this.drawLineMode) {
          case "y":
            context.lineTo(this.mousedown.x, loc.y);
            break
          case "all":
            context.lineTo(loc.x, loc.y);
            break
        }*/
        context.lineTo(loc.x, loc.y);
        context.stroke();
      },
      //画横线，在y坐标上
      drawHorizontLine(y) {
        const context = this.$refs.canvas.getContext('2d');
        context.beginPath();
        context.moveTo(0, y + 0.5);
        context.lineTo(canvas.width, y + 0.5);
        context.stroke();
      },
      //画竖线
      drawVerticalLine(x) {
        const context = this.$refs.canvas.getContext('2d');
        context.beginPath();
        context.moveTo(x + 0.5, 0);
        context.lineTo(x + 0.5, canvas.height);
        context.stroke();
      },
      drawGuidewires(x, y) {
        const context = this.$refs.canvas.getContext('2d');
        context.save();
        context.strokeStyle = "rgba(0,0,230, 0.4)";
        context.lineWidth = 0.5;
        this.drawHorizontLine(y);
        this.drawVerticalLine(x);
        context.restore();
      },
      handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {
          });
      },
      mouseDown(e) {
        const loc = this.canvasLocate(e)
        e.preventDefault();
        this.saveDrawingSurface();
        this.mousedown.x = loc.x;
        this.mousedown.y = loc.y;
        this.dragging = true;
        let lineList = this.lineData[this.lineColor]
        if(!lineList){
          lineList = []
          this.lineData[this.lineColor] = lineList
        }
        lineList.push(new Line(this.lineColor,lineList.length,{
          x: loc.x,
          y: loc.y
        }))
      },
      mouseUp(e) {
        const loc = this.canvasLocate(e)
        this.modeLocate(loc)
        this.restoreDrawingSurface()
        this.updateRubberband(loc)
        //鼠标抬起，拖动标记设为否
        this.dragging = false
        let lineList = this.lineData[this.lineColor]
        let lastLine = lineList[lineList.length-1]
        lastLine.end = loc
      },
      //编程直线信息
      modeLocate(locate){
        switch (this.drawLineMode) {
          case "y":
            locate.x = this.mousedown.x
            break
        }
      },
      mouseMove(e) {
        //判断当前是否用户在拖动
        if (!this.dragging) {
          return
        }
        e.preventDefault();
        const loc = this.canvasLocate(e)
        this.modeLocate(loc)
        this.restoreDrawingSurface();
        this.updateRubberband(loc);
        /*if(guidewires) {
          //如果选中的加入辅助线
          //这里的辅助线应该只有在鼠标那个地方才出现的
          drawGuidewires(loc.x, loc.y);
        }*/
      }

    }
  }
</script>

<style scoped>
  canvas{
    border: #2c3e50 solid;
  }
</style>
