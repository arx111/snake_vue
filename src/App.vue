<template>
    <div id="app">
        <canvas @keydown="keyDown" tabindex='1' ref="canvas"></canvas>

        <div> Movement: WASD</div>
        <div> Length: {{ this.snake_.length }}</div>
        <div> Size: {{this.max_width}}, {{this.max_height}}</div>

        <div>WIDTH
            <button @click="max_width += 10">+</button>
            <button @click="max_width -= 10">-</button>
        </div>
        <div>HEIGHT
            <button @click="max_height += 10">+</button>
            <button @click="max_height -= 10">-</button>
        </div>
        <div>
            rows: {{rows}}
            <button @click="rows += 1">+</button>
            <button @click="rows -= 1">-</button>
        </div>
        <div>
            cols: {{cols}}
            <button @click="cols += 1">+</button>
            <button @click="cols -= 1">-</button>
        </div>
        <div>
            <button @click="clear()">Clear</button>
            <button @click="rescale()">Rescale</button>
        </div>

    </div>
</template>

<script>

    export default {
        name: 'App',
        data() {
            return {
                paused: true,
                reset: false,
                max_width: 500,
                max_height: 500,
                food: {x: 0, y: 0},
                direction: [0, -1],
                rows: 25,
                cols: 25,
                framesElapsed: 0,
                stop: false,
                then: 0,
                fps: 5,
                snakeHead: {
                    pos: {r: 5, c: 5}
                },
                snake: [],
                snake_: {
                    w: 1,
                    h: 1,
                    color: "#00FF00",
                    history: [],
                },
                snake_color: "#00FF00"
            }
        },
        computed: {
            gridsize_w() {
              return Math.floor(this.max_width  / this.cols)
            },
            gridsize_h() {
              return Math.floor(this.max_height  / this.rows)
            },
            width() {
                if ( this.gridsize * this.cols > this.max_width) {
                    return this.max_width
                }
                return this.gridsize * this.cols
            },
            height() {
                if (this.gridsize * this.rows > this.max_height) {
                    return this.max_height
                }
                return this.gridsize * this.rows
            },
            fpsInterval() {
                return 1000 / this.fps
            }
        },
        methods: {
            keyDown(e) {
                // Handling input
                switch (e.code) {
                    case "KeyW":
                        console.log("up")
                        if (this.direction[1] === 0) {
                            this.direction = [0, -1]
                        }
                        break;
                    case "KeyD":
                        console.log("right")
                        if (this.direction[0] === 0) {
                            this.direction = [1, 0]
                        }
                        break;
                    case "KeyA":
                        console.log("left")
                        if (this.direction[0] === 0) {
                            this.direction = [-1, 0]
                        }
                        break;
                    case "KeyS":
                        console.log("down")
                        if (this.direction[1] === 0) {
                            this.direction = [0, 1]
                        }
                        break;
                    case "KeyC":
                        this.paused = !this.paused
                        console.log("paused: ", this.paused)
                        break;
                    case "KeyR":
                        this.reset = true
                        console.log("Resetting: ", this.paused)
                        break;
                }
            },
            getRandomInt(max) {
                return Math.floor(Math.random() * Math.floor(max));
            },
            drawFood() {
                this.fillSquare(this.food.x,
                    this.food.y, "#FF0000")
            },
            newFood() {
                this.food.x = this.getRandomInt(this.cols)
                this.food.y = this.getRandomInt(this.rows)
                console.log("New food: ", this.food.x, this.food.y)
            },

            moveSnake() {
                let ln = this.snake_.history.length
                if (ln === 0) {
                    return
                }

                for (let i=0; i < ln; i++) {
                    if (i + 1 === ln) {
                        continue
                    }
                    console.log("Move:  ", this.snake_.history[i], this.snake_.history[i+1])
                    this.snake_.history[i] = this.snake_.history[i + 1]
                }
                this.snake_.history[ln - 1].pos.r += this.direction[0];
                this.snake_.history[ln - 1].pos.c += this.direction[1];

            },
            growSnake() {
                this.snake_.history.unshift(this.snake_.history[0])

            },
            drawSnake() {

                this.snake_.history.forEach((v, i) => {
                    console.log(i, " SNAKE: ", v.pos.r , v.pos.c)
                    this.fillSquare(v.pos.r , v.pos.c, this.snake_.color)
                })
            },
            clear() {
                this.$ctx.clearRect(
                    0,
                    0,
                    this.max_width,
                    this.max_height,
                )
            },
            consumed() {
                let ln = this.snake_.history.length
                return this.snake_.history[ln - 1].pos.r === this.food.x && this.snake_.history[ln- 1].pos.c === this.food.y;
            },
            drawGrid() {
                // horizontal lines
                for (let r = 0; r < this.rows; r++) {
                    let y = r * this.gridsize_h
                    this.$ctx.beginPath();
                    this.$ctx.moveTo(0, y);
                    this.$ctx.lineTo(this.max_width, y);
                    this.$ctx.lineWidth = 1;
                    this.$ctx.stroke();
                }
                // vertical lines
                for (let c = 0; c < this.cols; c++) {
                    // console.log(c)
                    this.$ctx.beginPath();
                    let x = c * this.gridsize_w
                    this.$ctx.moveTo(x, 0);
                    this.$ctx.lineTo(x, this.max_height);
                    this.$ctx.stroke();
                }
                this.fillSquare(9, 5, "black")
            },
            fillSquare(r, c, color) {
                this.$ctx.fillStyle=color
                this.$ctx.fillRect(
                    r * this.gridsize_w,
                    c * this.gridsize_h,
                    this.gridsize_w,
                    this.gridsize_h,
                )
            },
            setCanvasWidth() {
                this.$canvas.width = this.max_width
                console.log("width set to: ", this.$canvas.width)
            },
            setCanvasHeight() {
                this.$canvas.height = this.max_height
                console.log("height set to: ", this.$canvas.height)
            },
            rescale(){
                this.setCanvasWidth()
                    this.setCanvasHeight()
            },
            gameLoop() {
                window.requestAnimationFrame(this.gameLoop)
                if (this.reset) {
                    console.log("Resetting")
                    this.reset = false
                }
                if (this.paused) {
                    // Draw grid with rectangles when paused
                    this.clear()
                    this.drawGrid()
                    return
                }
                let now = Date.now()
                let elapsed = now - this.then;
                if (elapsed > this.fpsInterval) {
                    this.framesElapsed += 1
                    // console.log(`### Count. Elapsed: ${elapsed} ms | Frame ${this.framesElapsed}`)
                    this.clear()

                    this.drawSnake()
                    this.moveSnake()
                    if (this.consumed()) {
                        this.growSnake()
                        this.newFood()
                    }
                    console.log("Food: ", this.food.x, this.food.y )
                    this.drawFood()
                    this.then = now - (elapsed % this.fpsInterval);

                } else {
                    // console.log("No Count")
                }
            }
        },
        mounted() {
            this.$canvas = this.$refs["canvas"]
            this.setCanvasWidth()
            this.setCanvasHeight()


            this.$ctx = this.$canvas.getContext('2d')

            console.log(this.rows * this.gridsize_w)
            console.log(this.cols * this.gridsize_h)

            this.snake_.history.push(this.snakeHead)
            console.log(this.snake_.history)
            this.newFood()
            this.gameLoop()


        }
    }

</script>

<style>
    canvas {
        width: 500px;
        height: 500px;
        border-style: solid;
        border-width: 1px;
    }
</style>
