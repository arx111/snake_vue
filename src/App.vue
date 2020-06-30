<template>
    <div id="app">
        <canvas @keydown="keyDown" tabindex='1' ref="canvas"></canvas>

        <button @click="goLeft"> Left</button>
        <button @click="goUp"> Up</button>
        <button @click="goDown"> Down</button>
        <button @click="goRight"> Right</button>
    </div>
</template>

<script>

    export default {
        name: 'App',
        data() {
            return {
                food: {x: 0, y: 0},
                direction: [0, -1],
                gridsize: 10,
                rows: 20,
                cols: 20,
                framesElapsed: 0,
                stop: false,
                then: 0,
                fps: 5,
                canvas_rect: {
                    bottom: 0,
                    height: 0,
                    left: 0,
                    right: 0,
                    top: 0,
                    width: 0,
                    x: 0,
                    y: 0,
                },
                snakeBody: {
                    rect: {w: 10, h: 10},
                    pos: {r: 5, c: 5}
                },
                snake: []
            }
        },
        computed: {
            width() {
                return this.gridsize * this.cols
            },
            height() {
                return this.gridsize * this.rows
            },
            fpsInterval() {
                return 1000 / this.fps
            }
        },
        methods: {
            keyDown(e) {
                switch (e.code) {
                    case "KeyW":
                        this.goUp()
                        break;
                    case "KeyD":
                        this.goRight()
                        break;
                    case "KeyA":
                        this.goLeft()
                        break;
                    case "KeyS":
                        this.goDown()
                        break;
                }
            },
            getRandomInt(max) {
                return Math.floor(Math.random() * Math.floor(max));
            },
            drawFood() {
                this.$ctx.fillStyle = "#FF0000";
                this.$ctx.fillRect(
                    this.food.x * this.gridsize,
                    this.food.y * this.gridsize,
                    this.gridsize,
                    this.gridsize,
                )
            },
            grow() {
                // let current_pos = this.snak
                // this.snake.push(
                //
                // )
            },
            newFood() {
                this.food.x = this.getRandomInt(this.rows)
                this.food.y = this.getRandomInt(this.cols)
            },
            goLeft() {
                console.log("goLeft")
                if (this.direction[0] === 0) {
                    this.direction = [-1, 0]
                }
            },
            goRight() {
                console.log("goRight")
                if (this.direction[0] === 0) {
                    this.direction = [1, 0]
                }
            },
            goUp() {
                console.log("goUp")
                if (this.direction[1] === 0) {
                    this.direction = [0, -1]
                }
            },
            goDown() {
                console.log("goDown")
                if (this.direction[1] === 0) {
                    this.direction = [0, 1]
                }
            },
            moveSnake() {
                if (this.snake.length === 0) {
                    return
                }
                this.snake[this.snake.length - 1].pos.r += this.direction[0];
                this.snake[this.snake.length - 1].pos.c += this.direction[1];
            },
            drawSnake() {
                this.$ctx.fillStyle = "#00FF00";
                this.snake.forEach((v) => {
                    this.$ctx.fillRect(
                        v.pos.r * this.gridsize,
                        v.pos.c * this.gridsize,
                        this.gridsize,
                        this.gridsize,
                    )

                })
            },
            clear() {
                this.$ctx.clearRect(
                    0,
                    0,
                    this.width,
                    this.height,
                )
            },
            consumed() {
                return this.snake[this.snake.length - 1].pos.r === this.food.x && this.snake[this.snake.length - 1].pos.c === this.food.y;
            },
            gameLoop() {
                window.requestAnimationFrame(this.gameLoop)
                let now = Date.now()
                let elapsed = now - this.then;
                if (elapsed > this.fpsInterval) {
                    this.framesElapsed += 1
                    // console.log(`### Count. Elapsed: ${elapsed} ms | Frame ${this.framesElapsed}`)
                    this.clear()

                    this.drawSnake()
                    this.moveSnake()


                    if (this.consumed()) {

                        this.newFood()
                    }
                    this.drawFood()
                    this.then = now - (elapsed % this.fpsInterval);

                } else {
                    // console.log("No Count")
                }

            }

        },
        mounted() {
            this.$canvas = this.$refs["canvas"]

            this.$ctx = this.$canvas.getContext('2d')
            this.canvas_rect = this.$canvas.getBoundingClientRect()

            this.$canvas.width = this.cols * this.gridsize
            this.$canvas.height = this.rows * this.gridsize
            this.snake.push(this.snakeBody)
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
