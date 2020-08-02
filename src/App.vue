<template>
    <div id="app">
        <canvas @keydown="keyDown" tabindex='1' ref="canvas"></canvas>

        <div> Movement: WASD</div>
        <div> Length: {{ this.snake.length }}</div>
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
                max_width: 500,
                max_height: 500,
                accept_movement: true,
                food: {r: 0, c: 0},
                dx: 0,
                dy: 1,
                rows: 100,
                cols: 100,
                framesElapsed: 0,
                stop: false,
                then: 0,
                fps: 15,
                snake: {
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
                return Math.floor(this.max_width / this.cols)
            },
            gridsize_h() {
                return Math.floor(this.max_height / this.rows)
            },

            fpsInterval() {
                return 1000 / this.fps
            }
        },
        methods: {
            setDirection(wasd) {
                console.log("Setting Direction: ", wasd)
                if (wasd === "W") {
                    this.dy = -1
                    this.dx = 0
                }
                if (wasd === "A") {
                    this.dy = 0
                    this.dx = -1
                }
                if (wasd === "S") {
                    this.dy = 1
                    this.dx = 0
                }
                if (wasd === "D") {
                    this.dx = 1
                    this.dy = 0
                }
            },
            keyDown(e) {
                // Handling input
                switch (e.code) {
                    case "KeyO":
                        console.log("Sending Message to Server")
                        this.$ws.send("KeyO")
                        break;
                    case "KeyW":
                        if (this.dy === 0 && this.accept_movement) {
                            this.$ws.send("KeyW")
                        }
                        // this.accept_movement = false
                        break;
                    case "KeyD":
                        if (this.dx === 0 && this.accept_movement) {
                            this.$ws.send("KeyD")
                        }
                        // this.accept_movement = false
                        break;
                    case "KeyA":
                        if (this.dx === 0 && this.accept_movement) {
                            this.$ws.send("KeyA")
                        }
                        // this.accept_movement = false
                        break;
                    case "KeyS":
                        if (this.dy === 0 && this.accept_movement) {
                            this.$ws.send("KeyS")
                        }
                        // this.accept_movement = false
                        break;
                    case "KeyC":
                        this.paused = !this.paused
                        console.log("paused: ", this.paused)
                        break;
                    case "KeyR":
                        console.log("## Resetting")
                        this.reset("Manual Reset")
                        break;
                }
            },
            getRandomInt(max) {
                return Math.floor(Math.random() * Math.floor(max));
            },
            drawFood() {
                this.fillSquare(this.food.r,
                    this.food.c, "#FF0000")
            },
            newFood() {

                this.food.r = this.getRandomInt(this.cols)
                this.food.c = this.getRandomInt(this.rows)

                // This is very inefficient..
                this.snake.history.forEach( v => {
                    if (v.r === this.food.r && v.c === this.food.c ) {
                         this.newFood()
                    }
                })
            },
            randLocation() {
                return {r: this.getRandomInt(this.cols), c: this.getRandomInt(this.rows)}
            },
            newSnake() {
                this.snake.history = []
                this.snake.history.push(this.randLocation())
            },
            moveSnake() {
                let ln = this.snake.history.length
                if (this.snake.length === 0) {
                    return
                }
                let head = this.snake.history[ln - 1]
                this.snake.history.push(head)
                this.snake.history.shift()
                // add to last element
                this.snake.history[ln - 1] = {r: head.r + this.dx, c: head.c + this.dy};

            },
            growSnake() {
                let tail = this.snake.history[0]
                this.snake.history.unshift(tail)
            },
            drawSnake() {
                this.snake.history.forEach((v) => {
                    this.fillSquare(v.r, v.c, this.snake.color)
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
                let ln = this.snake.history.length
                return this.snake.history[ln - 1].r === this.food.r && this.snake.history[ln - 1].c === this.food.c;
            },
            pauseScreen() {
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
                this.$ctx.fillStyle = "black"
                this.$ctx.font = '50px serif';
                this.$ctx.fillText("Press C to start",
                    Math.floor(50),
                    Math.floor(this.max_height / 2)
                )
            },
            fillSquare(r, c, color) {
                this.$ctx.fillStyle = color
                this.$ctx.fillRect(
                    r * this.gridsize_w,
                    c * this.gridsize_h,
                    this.gridsize_w,
                    this.gridsize_h,
                )
                this.$ctx.strokeStyle = "black"
                this.$ctx.strokeRect(
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
            rescale() {
                this.setCanvasWidth()
                this.setCanvasHeight()
            },
            reset(msg) {
                console.log(msg)
                this.newSnake()
                this.paused = true
            },
            checkWallTouch() {
                let head = this.snake.history[this.snake.history.length - 1]
                if (head.r < 0 || head.r >= this.cols) {
                    this.reset("Wall Touch")

                } else if (head.c < 0 || head.c >= this.rows)
                    this.reset("Wall Touch")

            },
            checkBodyTouch() {
                let head = this.snake.history[this.snake.history.length - 1]
                if (this.snake.history.length < 4) {
                    return
                }
                for (let i = 0; i<this.snake.history.length - 1; i++ ) {
                    if (this.snake.history[i].r === head.r && this.snake.history[i].c === head.c) {
                        this.reset("Body Touch")
                    }
                }


            },
            gameLoop() {
                window.requestAnimationFrame(this.gameLoop)
                let now = Date.now()
                let elapsed = now - this.then;
                if (this.paused) {
                    // Draw grid with rectangles when paused
                    this.clear()
                    this.pauseScreen()
                    return
                }
                if (elapsed <= this.fpsInterval) {
                    return
                }
                this.framesElapsed += 1
                // console.log(`### Count. Elapsed: ${elapsed} ms | Frame ${this.framesElapsed}`)
                this.clear()
                this.accept_movement = true

                this.moveSnake()
                this.drawSnake()
                this.checkWallTouch()
                this.checkBodyTouch()

                if (this.consumed()) {
                    // this.dx += 1
                    // this.dy += 1
                    this.growSnake()
                    this.growSnake()
                    this.growSnake()
                    this.newFood()
                }
                this.drawFood()
                this.then = now - (elapsed % this.fpsInterval);

            }
        },
        mounted() {
            this.$ws = new WebSocket("ws://192.168.1.33:9000/ws", "protocolOne");
            this.$ws.onclose = () => {
                console.log("WS closed")
            }
            this.$ws.onopen = () => {
                console.log("WS opened")
            }
            this.$ws.onmessage = (m) => {
                console.log("WS message received: ")
                console.log(m.data)
                this.setDirection(m.data[m.data.length -1])
            }
            this.$ws.onerror = (err) => {
                console.log("WS Error")
                console.log(err)
            }
            console.log(this.$ws)
            this.$canvas = this.$refs["canvas"]
            this.setCanvasWidth()
            this.setCanvasHeight()
            this.$ctx = this.$canvas.getContext('2d')

            this.newFood()
            this.newSnake()
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
