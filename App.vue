<template>
  <div class="app-container">
    <div class="header">
      <h1 class="winner">WINNER</h1>
      <p class="winner-text">{{ winner }}</p>
    </div>
    <div class="wheel">
      <canvas ref="canvas" width="500" height="500"></canvas>
      <div class="center-circle" @click="spin">
        <div class="triangle"></div>
      </div>
    </div>
    <div class="historyArea">
      <h2>History</h2>
      <ul>
        <li v-for="(item, index) in history" :key="index">{{ item }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      wheelItems: "BLACK\nWHITE\nBLACK\nWHITE\nBLACK\nWHITE", // default items
      winner:" ", // Initial winner
      currentDeg: 0,
      step: 0,
      itemDegs: {},
      speed: 0,
      maxRotation: 0,
      pause: false,
      items: [],
      history: [],
    };
  },
  computed: {
    canvas() {
      return this.$refs.canvas;
    },
  },
  methods: {
    toRad(deg) {
      return deg * (Math.PI / 180.0);
    },
    randomRange(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    easeOutSine(x) {
      return Math.sin((x * Math.PI) / 2);
    },
    getPercent(input, min, max) {
      return (((input - min) * 100) / (max - min)) / 100;
    },
    createWheel() {
      this.items = this.wheelItems.split("\n");
      this.step = 360 / this.items.length;
      this.draw();
    },
    draw() {
      const ctx = this.canvas.getContext("2d");
      const width = this.canvas.width;
      const height = this.canvas.height;
      const centerX = width / 2;
      const centerY = height / 2;
      const radius = width / 2;

      ctx.clearRect(0, 0, width, height); // Clear the canvas

      ctx.beginPath();
      ctx.arc(centerX, centerY, radius, this.toRad(0), this.toRad(360));
      ctx.fillStyle = `rgb(${33},${33},${33})`;
      ctx.lineTo(centerX, centerY);
      ctx.fill();

      let startDeg = this.currentDeg;
      for (let i = 0; i < this.items.length; i++, startDeg += this.step) {
        let endDeg = startDeg + this.step;

        // Alternate Black and White colors
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius - 2, this.toRad(startDeg), this.toRad(endDeg));
        ctx.fillStyle = i % 2 === 0 ? "black" : "white";
        ctx.lineTo(centerX, centerY);
        ctx.fill();

        // Draw text
        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(this.toRad((startDeg + endDeg) / 2));
        ctx.textAlign = "center";
        ctx.fillStyle = i % 2 === 0 ? "white" : "black";
        ctx.font = 'bold 24px serif';
        ctx.fillText(this.items[i], 130, 10);
        ctx.restore();

        this.itemDegs[this.items[i]] = { startDeg, endDeg };

        // Check winner
        if (startDeg % 360 < 360 && startDeg % 360 > 270 && endDeg % 360 > 0 && endDeg % 360 < 90) {
          this.winner = this.items[i];
        }
      }
    },
    animate() {
      if (this.pause) return;

      this.speed = this.easeOutSine(this.getPercent(this.currentDeg, this.maxRotation, 0)) * 10;
      if (this.speed < 0.01) {
        this.speed = 0;
        this.pause = true;
      }
      this.currentDeg += this.speed;
      this.draw();
      window.requestAnimationFrame(this.animate);
    },
    spin() {
      if (this.speed !== 0) return;

      this.createWheel();
      this.draw();

      // Pick a random item as the target
      const randomItem = this.items[Math.floor(Math.random() * this.items.length)];
      const targetDeg = this.itemDegs[randomItem].endDeg;

      // Set maxRotation for several rotations before landing on the target
      this.maxRotation = 360 * 6 + targetDeg;
      this.currentDeg = 0;
      this.pause = false;

       // Add the winner to history when the spin stops
       this.history.unshift(this.winner); // add to the start of the list
      
      // Start animation
      window.requestAnimationFrame(this.animate);
    },
  },
  mounted() {
    this.createWheel();
  },
};
</script>

<style>
body {
  background-color: #333;
  font-family: Arial, sans-serif;
  color: #fff;
  margin: 0;
  padding: 0;
}

.historyArea {
  position: absolute;
  display: flex;
  justify-content: center;
  flex-direction: column;
  margin-top: -35%;
  right: 20%;
  color: #fff;
  background-color: rgba(0, 0, 0, 0.6);
  padding: 10px;
  border-radius: 5px;
}

.winner-text {
  position: absolute;
  font-size: 24px;
  font-weight: bold;
  right: 48%;
  color: yellow;
  /* Use padding or margin for fine adjustment */
}

.app-container {
  text-align: center;
}

.header {
  padding: 40px;
}
.winner{
  position:absolute;
  text-align: right;
  left: 46%;
  margin-top: -75px;
}

.wheel {
  display: flex;
  justify-content: center;
  position: relative;
  left:50%;
  margin-top: 0px;
  margin-bottom: 50px;
}

.center-circle {
  width: 100px;
  height: 100px;
  border-radius: 60px;
  background-color: #706b70;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}

.triangle {
  width: 0;
  height: 0;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
  border-right: 30px solid white;
  position: absolute;
  top: 50%;
  right: -220%;
  transform: translateY(-50%);
}

textarea {
  background-color: rgba(20, 20, 20, 0.2);
  caret-color: #fff;
  resize: none;
  color: #fff;
}
</style>
