<template>
    <div>
        <p>赤:{{ count_red }}, 緑:{{ count_green }},白:{{ count_white }}}</p>
        <p>{{ ["赤", "緑", "白"][wb] }}の手番</p>
        <div v-for="(i, idx) in 5" :key='idx' class="container">
            <div v-for="(j, idx2) in 5" :key='idx2' :class="{ 'waku': xy[i][j] != 6, 'canput': xy[i][j] == 6}">
                <div :class="{ 'red': xy[i][j]==0, 'green': xy[i][j]==1
                ,'white': xy[i][j]==2, 'blue': xy[i][j]==3, 'blank': xy[i][j]>=4}" @click="getCard(i, j)">
                  {{ i * j }}
                </div>
            </div>
        </div>
        <p>
            <button @click="pass">pass</button>
        </p>
    </div>
</template>

<script>
//var vm = new Vue({
export default {
  data() {
    return {
      wb: 1,
      xy: [
        [5, 5, 5, 5, 5, 5, 5],
        [5, 4, 4, 4, 4, 4, 5],
        [5, 4, 4, 4, 4, 4, 5],
        [5, 4, 4, 0, 4, 4, 5],
        [5, 4, 4, 4, 4, 6, 5],
        [5, 4, 4, 4, 4, 4, 5],
        [5, 5, 5, 5, 5, 5, 5],
      ],
    }
  },

  computed: {
    count_red() {
      return (this.xy.join('').match(/0/g) || []).length;
    },
    count_green() {
      return (this.xy.join('').match(/1/g) || []).length;
    },
    count_white() {
      return (this.xy.join('').match(/2/g) || []).length;
    },
    count_blue() {
      return (this.xy.join('').match(/3/g) || []).length;
    },
  },

  methods: {
    chekPutType() {
      for (var x = 1; x <= 5; x++) {
        for (var y = 1; y <= 5; y++) {
          if(this.xy[x][y] != 4)continue;

          for (var dx = -1; dx <= 1; dx++) {
            for (var dy = -1; dy <= 1; dy++) {
              if (dx == 0 && dy == 0) continue;
              var k = 1;
              while (this.xy[x + k * dx][y + k * dy] <= 1) {
                if (this.xy[x + k * dx][y + k * dy] == this.wb) {
                  if(k > 2) {
                    return "normal";
                  }
                  break;
                }
                k++;
             }
            }
          }

          for (var dx = -1; dx <= 1; dx++) {
            for (var dy = -1; dy <= 1; dy++) {
              if (dx == 0 && dy == 0) continue;
              var k = 1;

              while (this.xy[x + k * dx][y + k * dy] == 1 - this.wb) {
                k++;
              }
              k++;
              if (this.xy[x + k * dx][y + k * dy] == 2) {
                return "after"
              }
            }
          }
        }
      }
      return "no"
    },
    getCard(x, y) {
      if (this.xy[x][y] != 2) return;
      var flag = false;
      var type = chekPutType();

      alert("app" + type);
      if (type == "normal") {
        for (var dx = -1; dx <= 1; dx++) {
          for (var dy = -1; dy <= 1; dy++) {
            if (dx == 0 && dy == 0) continue;
            var k = 1;
            while (this.xy[x + k * dx][y + k * dy] <= 1) {
              if (this.xy[x + k * dx][y + k * dy] == this.wb) {
                k = 1;
                while (this.xy[x + k * dx][y + k * dy] == 1 - this.wb) {
                  flag = true;
                  this.xy[x + k * dx][y + k * dy] = this.wb;
                  k++;
                }
                break;
              }
              k++;
            }
          }
        }
      } else if (type == "after") {
        //自分以外の色が1つ以上ありその先に自分の色があれば置いてよい
        for (var dx = -1; dx <= 1; dx++) {
          for (var dy = -1; dy <= 1; dy++) {
            if (dx == 0 && dy == 0) continue;
            var k = 1;
            if (this.xy[x + k * dx][y + k * dy] == 1 - this.wb) {
              k++;
              while (this.xy[x + k * dx][y + k * dy] <= 2) {
                if (this.xy[x + k * dx][y + k * dy] == 2) {
                  flag = true;
                  break;
                }
                k++;
              }
              if(flag == true) {
                break;
              }
            }
          }
        }
      } else if(type == "next") {
         //自分以外の色が隣接していればよい
         //flag = true;
      }

      if(flag == false) {
        alert("cannot get card.");
        return;
      } else {
        this.xy[x][y] = this.wb;
        this.wb = 1 - this.wb;
        this.xy.push();
      }
    },
    pass() {
      this.wb = this.wb + 1;
      if(this.wb > 2)this.wb=0
    },
  },
}
//});

</script>

<style>
p {
  text-align: center;
}

div {
  min-width: 150px;
  min-height: 100px;
}

.container {
  display: flex;
  justify-content: center;
  text-align: center;
  font-size: 500%; 
}

.waku {
  border: 1px solid black;
  background-color: gray;
  display: table;
  width: 100%;
}

.canput {
  border: 1px solid yellow;
  background-color: gray;
  display: table;
  width: 100%;
}

.blank {
  background-color: gray;
  display: table-cell;
  vertical-align: middle;
}

.red {
  border: 35px;
  background-color: red;
  display: table-cell;
  vertical-align: middle;
}

.green {
  border: 35px;
  background-color: green;
  display: table-cell;
  vertical-align: middle;
}

.white {
  border: 35px;
  background-color: white;
  display: table-cell;
  vertical-align: middle;
}

.blue {
  border: 35px;
  background-color: blue;
  display: table-cell;
  vertical-align: middle;
}
</style>
