<template>
  <div id="app">
    <canvas
      ref="canvas"
      id="canvas"
      width="500"
      height="700"
      @mousemove="canvas($event)"
    ></canvas>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  mounted() {
    this.startGame();
  },
  data() {
    return {
      playerScore: 0,
      computerScore: 0,
      paddleWidth: 50,
      width: 500,
      height: 700,
      screenWidth: window.screen.width,
      ballX: 250,
      ballY: 350,
      ballRadius: 5,
      speedY: 1,
      speedX: 1,
      trajectoryX: "",
      computerSpeed: "",
      playerMoved: false,
      paddleContact: false,
      paddleBottomX: 225,
      paddleDiff: 25,
      paddleTopX: 225
    };
  },
  computed: {
      canvasPosition: function() { return this.screenWidth / 2 - this.width / 2}
  },
  methods: {
    renderCanvas() {
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");

      const paddleHeight = 10;
      const paddleWidth = 50;

      const ballRadius = 5;
      context.fillStyle = "black";
      context.fillRect(0, 0, 500, 700);
      context.fillStyle = "white";
      context.fillRect(this.paddleBottomX, 680, paddleWidth, paddleHeight);
      context.fillRect(this.paddleTopX, 10, paddleWidth, paddleHeight);
      // dashed line
      context.beginPath();
      context.setLineDash([4]);
      context.moveTo(0, 350);
      context.lineTo(500, 350);
      context.strokeStyle = "white";
      context.stroke();
      // ball
      context.beginPath();
      context.arc(this.ballX, this.ballY, ballRadius, 2 * Math.PI, false);
      context.fillStyle = "white";
      context.fill();

      context.font = "32px Courier New";
      context.fillText(this.playerScore, 20, canvas.height / 2 + 50);
      context.fillText(this.computerScore, 20, canvas.height / 2 - 30);
    },
    animate() {
      this.ballMove();
      // console.log('this.padlebottomx--->', this.paddleBottomX)
      this.renderCanvas();
      this.ballBoundaries();
      this.computerAI();
      window.requestAnimationFrame(this.animate);
    },
    ballMove() {
      // Vertical Speed
      this.ballY += -this.speedY;
      // Horizontal Speed
      if (this.playerMoved && this.paddleContact) {
        this.ballX += this.speedX;
      }
    },

    canvas(e) {
      const canvas = this.$refs.canvas;
       // let playerMoved = true;
      //  console.log('widthds ---->', this.screenWidth,this.width, this.screenWidth / 2 - this.width / 2)
      //  console.log('BEFORE: this.paddleBottomX -----> ', this.canvasPosition,  )
      this.paddleBottomX = e.clientX - this.canvasPosition - this.paddleDiff;
       console.log('AFTER: this.paddleBottomX -----> ', this.paddleBottomX, this.canvasPosition )
     
     if (this.paddleBottomX < this.paddleDiff) {
        this.paddleBottomX = 0;
      }
      if (this.paddleBottomX > this.width - this.paddleWidth) {
        this.paddleBottomX = this.width - this.paddleWidth;
      }

      canvas.style.cursor = "none";
    },

    startGame() {
      this.playerScore = 0;
      this.computerScore = 0;
      this.ballReset();
      console.log("bottomX-->", this.paddleBottomX);

      this.animate();
      this.createCanvas();
      this.canvas({ clientX: 0 });
    },
    ballReset() {
      this.ballX = this.width / 2;
      this.ballY = this.height / 2;
      this.speedY = -3;
      this.paddleContact = false;
    },

    ballBoundaries() {
      // Bounce off Left Wall
      if (this.ballX < 0 && this.speedX < 0) {
        this.speedX = -this.speedX;
      }
      // Bounce off Right Wall
      if (this.ballX > this.width && this.speedX > 0) {
        this.speedX = -this.speedX;
      }
      // Bounce off player paddle (bottom)
      if (this.ballY > this.height - this.paddleDiff) {
        if (
          this.ballX > this.paddleBottomX &&
          this.ballX < this.paddleBottomX + this.paddleWidth
        ) {
          this.paddleContact = true;
          // Add Speed on Hit
          if (this.playerMoved) {
            this.speedY -= 1;
            // Max Speed
            if (this.speedY < -5) {
              this.speedY = -5;
              this.computerSpeed = 6;
            }
          }
          this.speedY = -this.speedY;
          this.trajectoryX =
            this.ballX - (this.paddleBottomX + this.paddleDiff);
          this.speedX = this.trajectoryX * 0.3;
        } else if (this.ballY > this.height) {
          // Reset Ball, add to Computer Score
          this.ballReset();
          this.computerScore++;
        }
      }
      // Bounce off computer paddle (top)
      if (this.ballY < this.paddleDiff) {
        console.log('this.paddleTopX--->', this.paddleTopX)
        if (
          this.ballX > this.paddleTopX &&
          this.ballX < this.paddleTopX + this.paddleWidth
        ) {
          // Add Speed on Hit
          if (this.playerMoved) {
            this.speedY += 1;
            // Max Speed
            if (this.speedY > 5) {
              this.speedY = 5;
            }
          }
          this.speedY = -this.speedY;
        } else if (this.ballY < 0) {
          // Reset Ball, add to Player Score
          this.ballReset();
          this.playerScore++;
        }
      }
    },

    computerAI() {
      if (this.playerMoved) {
        if (this.paddleTopX + this.paddleDiff < this.ballX) {
          this.paddleTopX += this.computerSpeed;
        } else {
          this.paddleTopX -= this.computerSpeed;
        }
      }
    },

    createCanvas() {
      this.renderCanvas();
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
