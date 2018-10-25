<template>
<div class="layout">
    <div class="header">
        <div class="logo">
            作业批改
        </div>
        <div class="nav" style="position:fixed;top: 10px;">
            <mu-flat-button v-for="tab in tabs" :key="tab.name" :label="tab.name" class="tab demo-flat-button" :icon="tab.icon" @click="tabfun(tab.fun)" color="#6e86fd" backgroundColor="#FFFFFF" />
            <a href="javascript:void(0);" ref="download" download="picture.png" v-show="false"></a>
        </div>
    </div>
    <div class="content-left">
        <div class="setterSize">
            <span>线条粗细:{{penSize}}</span>
            <mu-slider class="demo-slider" v-model="penSize" :step="1" :max="10" :min="1" color="secondary" track-color="secondary" />
        </div>
        <div class="choose-color">
            <span v-for='(item, index) in colors' :class='{activeSpan: isSpanActive == index}' :style='{backgroundColor: item}' @click='colorSpanClick(index, item)'></span>
        </div>
        <mu-paper class="demo-menu">
            <mu-menu v-for="tool in tools" :key="tool.name">
                <mu-menu-item :title="tool.name" :leftIcon="tool.icon" @click="drawType(tool)" :class="{'selected':tool.ischoose}" />
            </mu-menu>
        </mu-paper>
    </div>
    <div class="content">
        <div class="content-right">
            <div class="body" :style="{width:canvasSize.width + 'px',height: canvasSize.height + 'px' }">
                <canvas id="canvas" ref="canvas" :style="{cursor:curcursor}"></canvas>
                <canvas id="canvas_bak" ref="canvas_bak" :style="{cursor:curcursor}"></canvas>
            </div>
        </div>
    </div>
</div>
</template>

<script>
export default {
    mounted() {
        this.initCanvas()
        this.addkeyBoardlistener()
        this.drawType(this.tools[0])
    },
    data() {
        return {
            isSpanActive: 0,
            canvasSize: {
                width: window.innerWidth * 0.8,
                height: window.innerHeight
            },
            canvas: this.$refs.canvas,
            context: null,
            canvas_bak: this.$refs.canvas_bak,
            context_bak: null,
            imageUrl: 'src/assets/img/15.jpg',
            color: {
                hex: '#FF0000'
            },
            colors: ['#FF0000', '#000000', '#0000FF'],
            penSize: 2,
            canDraw: false,
            curcursor: 'auto',
            cancelList: [],
            tools: [{
                name: '铅笔(CTRL+P)',
                icon: 'mode_edit',
                fun: 'pencil',
                ischoose: false,
            }, {
                name: '直线(CTRL+L)',
                icon: 'remove',
                fun: 'line',
                ischoose: false
            }, {
                name: '矩形(CTRL+R)',
                icon: 'crop_square',
                fun: 'square',
                ischoose: false
            }, {
                name: '正确(CTRL+O)',
                icon: 'check',
                fun: 'right',
                ischoose: false
            }, {
                name: '错误(CTRL+W)',
                icon: 'close',
                fun: 'wrong',
                ischoose: false
            }, {
                name: '星星(CTRL+B)',
                icon: 'star_border',
                fun: 'star',
                ischoose: false
            }, {
                name: '文字(CTRL+T)',
                icon: 'title',
                fun: 'text',
                ischoose: false
            }
                // , {
                //   name: '橡皮',
                //   icon: 'border_style',
                //   fun: 'rubber',
                //   ischoose: false
                // }
            ],
            tabs: [
                {
                    name: '清除(ctrl+c)',
                    icon: 'clear',
                    fun: 'clear'
                },
                {
                    name: '保存(ctrl+s)',
                    icon: 'system_update_alt',
                    fun: 'save'
                },
                {
                    name: '上一步(ctrl+z)',
                    icon: 'reply',
                    fun: 'cancel'
                }
            ]
        }
    },
    methods: {
        initCanvas() {
            let imgs = new Image();
            imgs.src = this.imageUrl;
            this.canvas = document.getElementById("canvas")
            this.canvas_bak = document.getElementById("canvas_bak")
            this.context = this.canvas.getContext('2d')
            this.context_bak = this.canvas_bak.getContext('2d')
            this.canvas.width = this.canvasSize.width
            this.canvas_bak.width = this.canvasSize.width
            imgs.onload = () => {
                let sWidth = imgs.width
                let sHeight = imgs.height
                imgs.width = this.canvas.width
                this.canvas.height = this.canvas.width * sHeight / sWidth
                this.canvas_bak.height = this.canvas_bak.width * sHeight / sWidth
                imgs.height = this.canvas.height
                this.canvasSize.height = this.canvas.height
                this.context.drawImage(imgs, 0, 0, this.canvas.width, this.canvas.height)
                this.context.drawImage(imgs, 0, 0, this.canvas_bak.width, this.canvas_bak.height)
            }
        },
        colorSpanClick(index, item) {
            this.isSpanActive = index;
            this.color.hex = item;
        },
        drawType(pen) {
            switch (pen.fun) {
                case 'pencil':
                    this.curcursor =
                        "url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAQAAABKfvVzAAAAZ0lEQVR4AdXOrQ2AMBRF4bMc/zOUOSrYoYI5cQQwpAieQDW3qQBO7Xebxx8bWAk5/CASmRHzRHtB+d0Bkw0W5ZiT0SYbFcl6u/2eeJHbxIHOhWO6Er6/y9syXpMul5PLefAGKZ1/rwtTimwbWLpiCgAAAABJRU5ErkJggg==') 3 21,  auto"
                        break
                case 'rubber':
                    this.curcursor = 'pointer'
                    break
                case 'square':
                case 'line':
                    this.curcursor = 'crosshair'
                    break
                case 'right':
                case 'wrong':
                case 'star':
                    this.curcursor = 'copy'
                    break
                case 'text':
                    this.curcursor = 'text'
                    break
                default:
                    this.curcursor = 'auto'
                    break;
            }
            this.draw_graph(pen.fun)
            this.chooseImg(pen)
        },
        //选择功能按钮 修改样式
        chooseImg(obj) {
            for (let i = 0; i < this.tools.length; i++) {
                this.tools[i].ischoose = false
            }
            obj.ischoose = true
        },
        tabfun(fun) {
            if (fun === 'clear') {
                this.clearContext('1')
            } else if (fun === 'save') {
                this.downloadImage()
            } else if (fun === 'cancel') {
                this.cancel()
            }
        },
        timesDraw(graphType) {
            let image = new Image()
            if (graphType != 'rubber') {
                image.src = this.canvas_bak.toDataURL()
                image.onload = () => {
                    this.context.drawImage(image, 0, 0, image.width, image.height, 0, 0, this.canvasSize.width, this.canvasSize.height)
                    this.clearContext()
                }
            }
            this.saveImageToAry()
        },
        drawRight(ctx, x, y) {
            ctx.beginPath()
            this.clearContext()
            ctx.lineJoin = 'round'
            ctx.lineCap = 'round'
            ctx.moveTo(x - 20, y - 20)
            ctx.lineTo(x - 5, y - 5)
            ctx.lineTo(x + 30, y - 40)
            ctx.stroke()
        },
        drawWrong(ctx, x, y) {
            ctx.beginPath()
            this.clearContext()
            ctx.lineJoin = 'round'
            ctx.lineCap = 'round'
            ctx.moveTo(x - 20, y - 20)
            ctx.lineTo(x, y)
            ctx.moveTo(x - 20, y)
            ctx.lineTo(x, y - 20)
            ctx.stroke()
        },
        drawStar(ctx, x, y) {
            ctx.beginPath()
            this.clearContext()
            for (let i = 0; i <= 5; i++) {
                let nx = x - 18 * Math.sin(Math.PI / 180 * 360 / 5 * i * 2)
                let ny = y - 18 * Math.cos(Math.PI / 180 * 360 / 5 * i * 2)
                ctx.lineTo(nx, ny)
            }
            ctx.fillStyle = this.color.hex;
            ctx.fill();
        },
        drawText(ctx, x, y) {
            let text = prompt('请输入批改文案(文案为单行，若要输入多行，请多次输入)');
            if (text) {
                ctx.font = "20px Roboto,Lato,sans-serif";
                ctx.fillStyle = this.color.hex;
                ctx.fillText(text, x, y + 6);
            } else {
                ctx.stroke()
                this.clearContext()
                return 0;
            }
        },
        draw_graph(graphType) {
            this.canvas_bak.style.zIndex = 1
            //先画在蒙版上 再复制到画布上
            this.canDraw = false
            let startX, startY, endX, endY
            let mousedown = (e) => {
                this.context.strokeStyle = this.color.hex
                this.context_bak.strokeStyle = this.color.hex
                this.context_bak.lineWidth = this.penSize
                e = e || window.event
                var rect = this.canvas.getBoundingClientRect();
                startX = e.pageX - rect.left
                startY = e.pageY - rect.top

                this.context_bak.moveTo(startX, startY)
                this.canDraw = true
                if (graphType == 'pencil') {
                    this.context_bak.beginPath()
                } else if (graphType == 'rubber') {
                    this.context.clearRect(startX - this.penSize * 10, startY - this.penSize * 10, this.penSize * 20, this.penSize * 20);
                }
            }
            let mouseup = (e) => {
                e = e || window.event
                var rect = this.canvas.getBoundingClientRect();
                endX = e.pageX - rect.left
                endY = e.pageY - rect.top
                this.canDraw = false
                if (graphType === 'right' || graphType === 'wrong' || graphType === 'text' || graphType === 'star') {
                    console.log('goon');
                    if (graphType === 'right') {
                        this.drawRight(this.context_bak, endX, endY)
                    } else if (graphType === 'wrong') {
                        this.drawWrong(this.context_bak, endX, endY)
                    } else if (graphType === 'star') {
                        this.drawStar(this.context_bak, endX, endY)
                    } else if (graphType === 'text') {
                        this.drawText(this.context_bak, endX, endY)
                    }
                } else {
                    if (Math.abs(endX - startX) < 3 && Math.abs(endY - startY) < 3) {
                        this.context_bak.stroke()
                        this.clearContext()
                        return 0;
                    }
                }
                //鼠标离开 把蒙版canvas的图片生成到canvas中
                this.timesDraw(graphType)
            }
            // 鼠标移动
            let mousemove = (e) => {
                e = e || window.event
                var rect = this.canvas.getBoundingClientRect();
                let x = e.pageX - rect.left
                let y = e.pageY - rect.top
                //方块  4条直线搞定
                if (graphType == 'square') {
                    if (this.canDraw) {
                        this.context_bak.beginPath()
                        this.clearContext()
                        this.context_bak.lineJoin = 'round'
                        this.context_bak.moveTo(startX, startY)
                        this.context_bak.lineTo(x, startY)
                        this.context_bak.lineTo(x, y)
                        this.context_bak.lineTo(startX, y)
                        this.context_bak.lineTo(startX, startY)
                        this.context_bak.stroke()
                    }
                    //直线
                } else if (graphType == 'line') {
                    if (this.canDraw) {
                        this.context_bak.beginPath()
                        this.clearContext()
                        this.context_bak.lineCap = 'round'
                        this.context_bak.moveTo(startX, startY)
                        this.context_bak.lineTo(x, y)
                        this.context_bak.stroke()
                    }
                    //画笔
                } else if (graphType == 'pencil') {
                    if (this.canDraw) {
                        this.context_bak.lineCap = 'round'
                        this.context_bak.lineJoin = 'round'
                        this.context_bak.lineTo(x, y)
                        this.context_bak.stroke()
                    }
                    //橡皮擦 不管有没有在画都出现小方块 按下鼠标 开始清空区域（会清除背景 的图片  需要另找解决方案）
                } else if (graphType == 'rubber') {
                    this.context_bak.lineWidth = 1
                    this.clearContext()
                    this.context_bak.beginPath()
                    this.context_bak.strokeStyle = '#000000'
                    this.context_bak.moveTo(x - this.penSize * 10, y - this.penSize * 10)
                    this.context_bak.lineTo(x + this.penSize * 10, y - this.penSize * 10)
                    this.context_bak.lineTo(x + this.penSize * 10, y + this.penSize * 10)
                    this.context_bak.lineTo(x - this.penSize * 10, y + this.penSize * 10)
                    this.context_bak.lineTo(x - this.penSize * 10, y - this.penSize * 10)
                    this.context_bak.stroke()
                    if (this.canDraw) {
                        this.context.clearRect(x - this.penSize * 10, y - this.penSize * 10, this.penSize * 20, this.penSize * 20)
                    }
                }
            }
            //鼠标离开区域以外 不显示橡皮擦
            let mouseout = () => {
                if (graphType === 'rubber') {
                    this.clearContext();
                }
            }
            this.canvas_bak.onmousedown = () => {
                mousedown()
            }
            this.canvas_bak.onmousemove = () => {
                mousemove()
            }
            this.canvas_bak.onmouseup = () => {
                mouseup()
            }
            this.canvas_bak.onmouseout = () => {
                mouseout()
            }
        },
        clearContext(type) {
            if (!type) {
                this.context_bak.clearRect(0, 0, this.canvasSize.width, this.canvasSize.height)
            } else {
                this.context.clearRect(0, 0, this.canvasSize.width, this.canvasSize.height)
                this.context_bak.clearRect(0, 0, this.canvasSize.width, this.canvasSize.height)
                this.initCanvas()
            }
        },
        downloadImage() {
            this.$refs.download.href = this.canvas.toDataURL()
            this.$refs.download.click()
        },
        cancel() {
            console.log(this.cancelList);
            if (this.cancelList.length !== 0) {
                this.context.putImageData(this.cancelList.pop(), 0, 0);
            }
            console.log(this.cancelList);
        },
        //保存历史 用于撤销
        saveImageToAry() {
            console.log('saveImageToAry');
            console.log(this.cancelList);
            let width = this.context.canvas.width;
            let height = this.context.canvas.height;
            this.cancelList.push(this.context.getImageData(0, 0, width, height));
            console.log(this.cancelList);
        },
        addkeyBoardlistener() {
            document.onkeydown = (event) => {
                let e = event || window.event || arguments.callee.caller.arguments[0];
                if (e.ctrlKey) {
                    switch (e.keyCode) {
                        case 90:
                            this.cancel()
                            break
                        case 67:
                            this.clearContext('1')
                            break
                        case 83:
                            this.downloadImage()
                            break
                        case 80:
                            this.drawType(this.tools[0])
                            break
                        case 76:
                            this.drawType(this.tools[1])
                            break
                        case 82:
                            this.drawType(this.tools[2])
                            break
                        case 79:
                            this.drawType(this.tools[3])
                            break
                        case 87:
                            this.drawType(this.tools[4])
                            break
                        case 66:
                            this.drawType(this.tools[5])
                            break
                        case 84:
                            this.drawType(this.tools[6])
                            break
                    }
                }
            }
        }
    }
}
</script>
<style scoped>
* {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
canvas {
    position: absolute;
    z-index: 0;
    left: 0;
    right: 0;
    width: 100%;
    height: 100%;
}
.layout {
    background-color: rgb(236, 236, 236);
    /* height: 100%; */
}
.header {
    background-color: #6e86fd;
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    z-index: 999;
}
.logo {
    font-size: 24px;
    color: white;
    display: inline-block;
    padding: 10px 20px;
}
.logo:hover {
    transform: rotateY(666turn);
    transition-delay: 1s;
    transition-property: all;
    transition-duration: 59s;
    transition-timing-function: cubic-bezier(.34, 0, .84, 1);
}
.nav {
    display: inline-block;
    width: calc(100% - 150px);
    margin: 0 auto;
}
.content {
    overflow: hidden;
    /* height: calc(100% - 56px); */
    margin-top: 56px;
}
.content-left {
    min-width: 2rem;
    width: 15%;
    float: left;
    height: 100%;
    background-color: white;
    position: fixed;
    left: 0;
    top: 56px;
}
.content-right {
    width: 85%;
    display: inline-block;
    margin-left: 15%;
    padding: 10px 20px;
    background-color: rgba(0, 0, 0, 0)
}
.breadcrumb {
    margin: 10px 0;
}
.body {
    position: relative;
    background-color: white;
    border-radius: 5px;
    height: 100%;
    margin: 0 auto;
}
.setterSize {
    padding: 5%;
    font-size: 0.5rem;
}
.choose-color {
    display: flex;
    justify-content: space-around;
    margin-bottom: 16px;
}
.choose-color span {
    display: inline-block;
    width: 30px;
    height: 30px;
    border-radius: 2px;
    cursor: pointer;
    transition: all .2s;
}
.choose-color .activeSpan {
    border-radius: 15px;
}
.demo-menu {
    display: inline-block;
    margin: 0px auto;
    width: 100%;
}
.mu-paper,
.mu-menu,
.mu-menu-item-wrapper {
    overflow: hidden;
    width: 100%;
}
.selected {
    background: rgba(0, 0, 0, 0.35) !important;
}
.tab {
    margin-right: 5px
}
</style>
