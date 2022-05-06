<template>
  <div class="result" v-html="output"></div>
</template>

<script>
export default {
  data() {
    return {
      output: "",
    };
  },
  mounted() {
    this.axios
      .get("/api")
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
        this.output = temp;
      })
      .catch((e) => {
        console.log(e);
      });
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

.result {
  width: 800px;
  margin: auto;
  margin-top: 20px;
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
