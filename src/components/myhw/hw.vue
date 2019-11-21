<template>
  <div>
    <el-container>
      <el-header>嵌漫</el-header>
      <el-container>
        <el-aside width="200px">
          <el-button @click="erase">清空</el-button>
          <br>
          <el-button @click="textEdit">入字</el-button>
          <br>
          <span>操作模式</span><br>
          <el-radio-group v-model="model" size="small">
            <el-radio label="1">划线</el-radio>
          </el-radio-group>
          <br>
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
    function Line(color, no, begin) {
        this.color = color
        this.begin = begin
        //this.end = end
        this.no = no
        this.vLen = function () {
            return Math.abs(this.begin.y - this.end.y)
        }
        this.top = function () {
            return this.begin.y < this.end.y ? this.begin : this.end
        }
        this.bottom = function () {
            return this.begin.y > this.end.y ? this.begin : this.end
        }
    }

    export default {
        name: "hw",
        data() {
            return {
                //文字大小(px)
                textSize: 30,
                //存放所有线段数据
                lineData: {},
                //鼠标按下时,记录位置
                mousedown: {},
                //画线模式: y: 画竖线 all: 无限制
                drawLineMode: 'y',
                // canvas的context对象
                ctx: undefined,
                //线条颜色
                lineColor: 'black',
                //canvas画图缓存对象
                imageData: [],
                //操作模式
                model: '1',
                //对话框
                dialogVisible: false,
                //文本输入内容
                content: '',
                dragging: false,//是否在拖动
            }
        },
        mounted() {
            this.ctx = this.$refs.canvas.getContext('2d');
        },
        methods: {
            //文本输入对话框确认时,找到当前编辑的线条颜色的线条数据集
            //遍历文本每一个字符,计算字高,将字依次打到合适的位置
            textOK() {
                let lineList = this.lineData[this.lineColor]
                console.log(lineList)
                let index = 0//lineList索引
                let count = 0//计数器当另起一列竖线时,count重置为零
                let line = lineList[index]
                this.ctx.font = this.textSize + "px serif";
                for (let i = 0; i < this.content.length; i++) {
                    let y = this.textSize * count + line.top().y
                    this.ctx.fillText(this.content[i], line.top().x, y);
                    count++
                    if (index < lineList.length - 1 && y >= line.bottom().y) {
                        line = lineList[++index]
                        count = 0
                    }
                }
                this.dialogVisible = false
            },
            textEdit() {
                this.content = ''
                this.dialogVisible = true
            },
            erase() {
                this.ctx.clearRect(0, 0, canvas.width, canvas.height);
                this.lineData = {}
                saveDrawingSurface();
            },
            colorChange() {
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

                this.ctx.font = "48px serif";
                this.ctx.strokeText("Hello world", 10, 50);
            },
            //保存当前的canvas上的数据
            saveDrawingSurface() {
                this.imageData = this.ctx.getImageData(0, 0, canvas.width, canvas.height);
            },
            //恢复canvas的数据，主要用来显示最新的线段，擦除原来的线段
            restoreDrawingSurface() {
                this.ctx.putImageData(this.imageData,
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
                e.preventDefault();
                const loc = this.canvasLocate(e)
                this.saveDrawingSurface();
                this.mousedown.x = loc.x;
                this.mousedown.y = loc.y;
                this.dragging = true;

                //记录线条数据
                let lineList = this.lineData[this.lineColor]
                if (!lineList) {
                    lineList = []
                    this.lineData[this.lineColor] = lineList
                }
                lineList.push(new Line(this.lineColor, lineList.length, {
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
                let lastLine = lineList[lineList.length - 1]
                lastLine.end = loc
                //线条集合重排序 由于排字一般从右往左依次排,所以线条也要根据位置从右往左排序
                lineList.sort((a, b) => {
                    return b.top().x - a.top().x
                })
            },
            //编程直线信息
            modeLocate(locate) {
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
  canvas {
    border: #2c3e50 solid;
  }
</style>
