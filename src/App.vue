<template>
  <div class="p5Canvas"></div>
</template>

<script>
import P5 from 'p5';
import ml5 from 'ml5';

export default {
  data() {
    return {
      p5Canvas: null,
    }
  },
  created() {
    const sketch = p5 => {
      let video;
      let poseNet;
      let noseX = 0;
      let noseY = 0;
      let eyelX = 0;
      let eyelY = 0;
      var width = 1380;
      var height = 840;
      let sun;
      let particles = [];
      let r = 200;
      let canvas;
      p5.setup = () => {
        canvas = p5.createCanvas(width, height);
        video = p5.createCapture(p5.VIDEO);
        video.hide();
        poseNet = ml5.poseNet(video, modelReady);
        poseNet.on('pose', gotPoses);
        for (let i = 0; i < 1000; i++) {
          let mode = p5.random(0,1);
          let len = p5.random(0, 1.0);
          let angle = p5.random(p5.TWO_PI);
          let orbitspeed = p5.random(0.01, 0.1);
          particles.push(new Planet(mode, len, angle, orbitspeed));
        }
      };
      function gotPoses(poses) {
        // console.log(poses);
        if (poses.length > 0) {
          let nX = poses[0].pose.keypoints[10].position.x * width /video.width;
          let nY = poses[0].pose.keypoints[10].position.y * width /video.width;
          let eX = poses[0].pose.keypoints[9].position.x  * width /video.width;
          let eY = poses[0].pose.keypoints[9].position.y  * width /video.width;
          noseX = p5.lerp(noseX, nX, 0.5);
          noseY = p5.lerp(noseY, nY, 0.5);
          eyelX = p5.lerp(eyelX, eX, 0.5);
          eyelY = p5.lerp(eyelY, eY, 0.5);
        }
      }
      function modelReady() {
        console.log('model ready');
      }
      p5.draw = () => {
        p5.translate(width,0);
        p5.scale(-1, 1);
        p5.image(video, 0, 0, width, width * video.height / video.width);
        let d = p5.dist(noseX, noseY, eyelX, eyelY);
        if (d >= 250) {
          d = 250;
        }
        p5.noStroke();
        // fill(61, 229, 255, 100);
        // circle(0,0,200);
        // fill(18, 29, 200, 100);
        // circle(0,0,100);
        p5.push();
        drawGradient(noseX, noseY, d);
        for (let i = 0; i < 1000; i++) {
          particles[i].show(noseX, noseY,d);
          particles[i].orbit();
        }
        p5.frameRate(30);
        p5.pop();
      }
      class Planet {
        constructor(mode, len, angle, orbitspeed) {
          this.mode = mode;
          this.orbitspeed = orbitspeed;
          this.angle = angle;
          this.len = len;
        }
        orbit() {
          this.angle += this.orbitspeed;
        }
        show(x, y, d) {
          p5.fill(p5.random(0, 150), p5.random(100,360), p5.random(270, 360), 360);
          p5.noStroke();
          // console.log(this.a)
          var newx;
          var newy; 
          if(this.mode == 0){
            newx = d*p5.cos(this.angle);
            newy = d*this.len*p5.sin(this.angle);
          } else {
            newx = d*this.len*p5.cos(this.angle);
            newy = d*p5.sin(this.angle);
          }
          p5.translate(newx, newy);
          p5.ellipse(x, y, 5 + 0.02*d);
          p5.translate(-1*newx, -1*newy);
        }
      }
      function drawGradient(x, y, d) {
        let g = 20;
        let b = 150;
        let h = 0;
        for (let i = 1.5*d; i> 0; --i) {
          p5.fill(h, g + h, b + h, 3);
          p5.ellipse(x, y, i, i);
          h = (h + 0.8) % 360;
        }
      }
    }
    this.p5Canvas = new P5(sketch, 'p5Canvas');
  },
  unmounted() {
    this.p5Canvas = null;
  }
}
</script>

<style>
#p5Canvas {
  width: 100vw;
  position: relative;
}

main {
  margin: 0;
  width: 100vw;
}
</style>