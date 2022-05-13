<template>
  <div class="main">
    <div class="input">
      <input class="file" type="file" ref="file" />
      <div class="button" @click="start">実行</div>
      <canvas class="canvas" ref="canvas" width="128" height="128"></canvas>
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
      isLoading: false,
      isOutput: false,
      output: "",
      image: "",
    };
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d");
  },
  methods: {
    start() {
      let vm = this;
      this.isLoading = true;
      this.isOutput = false;
      let reader = new FileReader();
      reader.onload = function (e) {
        let image = new Image();
        image.src = e.target.result;
        image.onload = function () {
          vm.context.drawImage(image, 0, 0, 128, 128);
          vm.image = vm.canvas.toDataURL("image/jpeg");
          vm.api();
        };
      };
      reader.readAsDataURL(this.$refs.file.files[0]);
    },
    api() {
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
