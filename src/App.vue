<template>
  <div class="main">
    <div class="input">
      <input class="file" type="file" ref="file" @change="start" />
      <canvas
        class="canvas"
        ref="canvasEdit"
        width="640"
        height="480"
        @mousedown="mouseDown"
        @mousemove="mouseMove"
        @mouseup="mouseUp"
      ></canvas>
      <br />
      <canvas
        class="canvas canvas-preview"
        ref="canvas"
        width="128"
        height="128"
      ></canvas>
      <div class="button" @click="api">実行</div>
    </div>
    <div class="loading" v-if="isLoading">
      <img src="/assets/img/loading.gif" />
    </div>
    <div class="result" v-html="output" v-if="isOutput"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isBaseImage: false,
      isLoading: false,
      isOutput: false,
      output: "",
      baseImage: new Image(),
      image: "",
      isMouseDown: false,
      sx: 0,
      sy: 0,
      ex: 0,
      ey: 0,
    };
  },
  mounted() {
    const vm = this;
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d");
    this.canvasEdit = this.$refs.canvasEdit;
    this.contextEdit = this.canvasEdit.getContext("2d");
    this.baseImage.src = "/assets/img/noimage.png";
    this.baseImage.onload = function () {
      vm.contextEdit.drawImage(vm.baseImage, 0, 0, 640, 480);
    };
  },
  methods: {
    start() {
      let vm = this;
      this.isOutput = false;
      let reader = new FileReader();
      reader.onload = function (e) {
        vm.baseImage.src = e.target.result;
        vm.baseImage.onload = function () {
          vm.contextEdit.drawImage(vm.baseImage, 0, 0, 640, 480);
          vm.isBaseImage = true;
        };
      };
      reader.readAsDataURL(this.$refs.file.files[0]);
    },
    api() {
      if (this.image != "") {
        this.isLoading = true;
        this.axios
          .post("/api", {
            image: this.image,
          })
          .then((response) => {
            let temp = "";
            let arr = Object.keys(response.data).map(function (key) {
              return [key, response.data[key]];
            });
            for (let i = 0; i < arr.length; i++) {
              temp +=
                "<div class='box'><div class='text'>" +
                arr[i][0] +
                "</div><div class='bar'><div class='inside' style='width: " +
                parseInt(arr[i][1] * 100) +
                "%'></div><div class='value'>" +
                parseInt(arr[i][1] * 100) +
                " %</div></div></div>";
            }
            this.isLoading = false;
            this.isOutput = true;
            this.output = temp;
          })
          .catch((e) => {
            console.log(e);
          });
      }
    },
    mouseDown(e) {
      if (this.isBaseImage == true) {
        const rect = e.target.getBoundingClientRect();
        this.sx = e.clientX - rect.left;
        this.sy = e.clientY - rect.top;
        this.isMouseDown = true;
      }
    },
    mouseMove(e) {
      if (this.isBaseImage == true && this.isMouseDown == true) {
        const rect = e.target.getBoundingClientRect();
        this.ex = e.clientX - rect.left;
        this.ey = e.clientY - rect.top;
        this.contextEdit.drawImage(this.baseImage, 0, 0, 640, 480);
        this.contextEdit.strokeStyle = "red";
        this.contextEdit.strokeWidth = 5;
        this.contextEdit.strokeRect(
          this.sx,
          this.sy,
          this.ex - this.sx,
          this.ey - this.sy
        );
      }
    },
    mouseUp() {
      this.isMouseDown = false;
      this.context.drawImage(
        this.canvasEdit,
        this.sx,
        this.sy,
        this.ex - this.sx,
        this.ey - this.sy,
        0,
        0,
        128,
        128
      );
      this.image = this.canvas.toDataURL("image/jpeg");
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

.main {
  width: 800px;
  margin: auto;
  margin-top: 20px;
  margin-bottom: 30px;
}

.input {
  padding: 10px 30px;
}

.input .file {
  padding: 10px 0;
}

.input .button {
  width: 100px;
  padding: 10px 0;
  color: #fff;
  background-color: #1976d2;
  font-family: "sans-serif";
  font-size: 16px;
  border-radius: 5px;
  text-align: center;
}

.input .button:hover {
  background-color: #1565c0;
  cursor: pointer;
}

.input .canvas {
  padding: 10px 0;
}

.input .canvas-preview {
  margin-top: -24px;
}

.loading {
  margin: 15px 30px;
}

.result .box {
  padding: 10px 30px;
  box-sizing: border-box;
  overflow: hidden;
}

.result .box .text {
  width: 40%;
  margin-top: 3px;
  float: left;
  font-family: "RobotoRegular";
}

.result .box .bar {
  width: 60%;
  height: 10px;
  position: relative;
  float: right;
  background-color: #9e9e9e;
  border-radius: 8px;
}

.result .box .bar .inside {
  height: 10px;
  position: absolute;
  left: 0;
  top: 0;
  background-color: #f44336;
  border-radius: 5px;
}

.result .box .bar .value {
  position: absolute;
  left: 0;
  top: 12px;
  font-family: "RobotoRegular";
  font-size: 12px;
  color: #9e9e9e;
}
</style>
