<template>
    <div>
        <div v-for="(i, idx) in 5" :key='idx' class="container">
            <div v-for="(j, idx2) in 5" :key='idx2' :class="{ 'waku': xy[i][j] != 6, 'canput': xy[i][j] == 6}">
                <div :class="{ 'red': xy[i][j]==0, 'green': xy[i][j]==1
                ,'white': xy[i][j]==2, 'blue': xy[i][j]==3, 'blank': xy[i][j]==4 || xy[i][j]>=6, 'attack': xy[i][j]==5}" @click="getCard(i, j)">
                  {{ i * 5 + j - 5 }}
                </div>
            </div>
        </div>
        <p>赤:{{ counter[0] }}, 緑:{{ counter[1] }},白:{{ counter[2] }}</p>
        <p>{{ ["赤の選択中", "緑の選択中", "白の選択中", "青の選択中", "ゲーム進行中", "アタックチャンス"][wb] }}</p>
        <div class="member">
            <button class="button_red" @click="ans_red"><img src="../images/date.jpg" height="100" width="120"><br>赤</button>
            <button class="button_green" @click="ans_green"><img src="../images/kuwabara.jpg" height="100" width="120"><br>緑</button>
            <button class="button_white" @click="ans_white"><img src="../images/shimazu.jpg" height="100" width="120"><br>白</button>
            <button class="button_blue" @click="ans_blue"><img src="../images/wotsushi.jpg" height="100" width="120"><br>青</button>
        </div>
        <p>
            <button @click="next">次の問題</button>
            <button @click="attackChance">アタックチャンス</button>
        </p>
        <audio id="audio_fail" src="../assets/audio/fail.wav"></audio>
        <audio id="audio_green" src="../assets/audio/green.wav"></audio>
        <audio id="audio_op" src="../assets/audio/op.wav"></audio>
        <audio id="audio_red" src="../assets/audio/red.wav"></audio>
        <audio id="audio_white" src="../assets/audio/white.wav"></audio>
        <audio id="audio_thinking" src="../assets/audio/thinking.wav"></audio>
    </div>
</template>
<script>
export default {
  data() {
    return {
      wb: 4,
      xy: [
        [7, 7, 7, 7, 7, 7, 7],
        [7, 4, 4, 4, 4, 4, 7],
        [7, 4, 4, 4, 4, 4, 7],
        [7, 4, 4, 4, 4, 4, 7],
        [7, 4, 4, 4, 4, 4, 7],
        [7, 4, 4, 4, 4, 4, 7],
        [7, 7, 7, 7, 7, 7, 7],
      ],
      putlist: [],
      counter: [],
      otetsuki: [false, false, false ,false],
      question: 1,
      status: "start",//start, normal, answer, put, final
    }
  },

  computed: {
    panel_size: () => 5,
    user_count: () => 4,
    red_panel: () => 0,
    green_panel: () => 1,
    white_panel: () => 2,
    blue_panel: () => 3,
    blank_panel: () => 4,
    attack_panel: () => 5,
    canput_panel: () => 6,
    out_of_panel: () => 7,
    names: () => ['赤', '緑', '白', '青'],
  },

  methods: {
    getCanPutList(color) {
      this.putlist.length = 0;
      var secondlist = [];
      var thirdlist = [];
      var remainList = [];


      if (this.status == "start") {
        this.putlist.push({x : 3, y : 3});
      } else {
        for (var x = 1; x <= this.panel_size; x++) {
          for (var y = 1; y <= this.panel_size; y++) {
            if(this.xy[x][y] != this.blank_panel && this.xy[x][y] != this.attack_panel)continue;
            var push_flag = false

            // rule
            for (var dx = -1; dx <= 1; dx++) {
              for (var dy = -1; dy <= 1; dy++) {
                if (dx == 0 && dy == 0) continue;
                var k = 1;
                if (this.xy[x + k * dx][y + k * dy] >= this.user_count) {
                  continue;
                } else if (this.xy[x + k * dx][y + k * dy] == color) {
                  if(!thirdlist.some(b => b.x === x && b.y === y)) {
                    thirdlist.push({x : x, y : y});
                  }
                  push_flag = true
                  continue;
                }

                k++;
                while (this.xy[x + k * dx][y + k * dy] <= this.attack_panel) {
                  if (this.xy[x + k * dx][y + k * dy] == color) {
                    if(!this.putlist.some(b => b.x === x && b.y === y)) {
                      this.putlist.push({x : x, y : y});
                    }
                    push_flag = true
                    break;
                  } else if (this.xy[x + k * dx][y + k * dy] == this.blank_panel
                            || this.xy[x + k * dx][y + k * dy] == this.attack_panel ) {
                    if(!secondlist.some(b => b.x === x && b.y === y)) {
                      secondlist.push({x : x, y : y});
                    }
                    push_flag = true
                    break;
                  }
                  k++;
                }
              }
            }

            if(!push_flag)remainList.push({x : x, y : y});
          }
        }

        if (this.putlist.length > 0) {
          console.log('putlist')
          this.putlist.forEach(function (value, index) {
              console.log(index + ' : (' + value.x + ',' + value.y + ')');
          });
          return this.putlist;
        } else if (secondlist.length > 0 ) {
          console.log('secondlist')
          secondlist.forEach(function (value, index) {
              console.log(index + ' : (' + value.x + ',' + value.y + ')');
          });
          this.putlist = secondlist;
        } else if (thirdlist.length > 0) {
          console.log('thirdlist')          
          thirdlist.forEach(function (value, index) {
              console.log(index + ' : (' + value.x + ',' + value.y + ')');
          });
          this.putlist = thirdlist;
        } else if (remainList.length > 0) {
          console.log('remainList')
          remainList.forEach(function (value, index) {
              console.log(index + ' : (' + value.x + ',' + value.y + ')');
          });
          this.putlist = remainList;
        }
      }

      return this.putlist;
    },
    getCard(x, y) {
      var put_flag = false
      if(this.status == "put") {
        if(this.putlist.some(b => b.x === x && b.y === y)) {
          this.xy[x][y] = this.wb;
          this.calcReverse(x,y);
          put_flag=true;
          this.wb = this.blank_panel;
          this.status = "normal"
        } else {
          alert("そこには置けないよ！");
        }
      } else if(this.status == "attack") {
        this.wb = this.attack_panel;
        if(this.xy[x][y] < this.user_count) {
          this.xy[x][y] = this.attack_panel;
          put_flag=true;
          this.status = "normal"
        }
        this.wb = this.blank_panel;
      }

      if(put_flag) {
        this.clearPanel();
        this.checkFinish();
        this.next();
      }
    },
    next() {
        for(var i = 0; i < this.otetsuki.length; i++ ) {
          this.otetsuki[i] = false
        }
    },
    attackChance() {
      this.clearPanel();
      this.status = "attack";
    },
    checkFinish() {
      var sum = this.counter.reduce((a,x) => a+=x,0);
      console.log("counter :" + sum);
      if(sum >= 25) {
        this.status = "final";
        alert("ゲーム終了！");
      }
    },
    calcReverse(x, y) {
      for (var dx = -1; dx <= 1; dx++) {
        for (var dy = -1; dy <= 1; dy++) {
          var k = 1;
          if (dx == 0 && dy == 0
            || this.xy[x + k * dx][y + k * dy] >= this.user_count
            || this.xy[x + k * dx][y + k * dy] == this.wb) {
            continue;
          }

          k++;
          while (this.xy[x + k * dx][y + k * dy] < this.user_count) {
            if (this.xy[x + k * dx][y + k * dy] == this.wb) {
              for(var i = 1; i < k ;i++) {
                this.xy[x + i * dx][y + i * dy] = this.wb;
              }
              break;
            }
            k++;
          }
        }
      }
    },
    clearPanel() {
      for(var i = 0; i < this.user_count; i++) {
        this.counter[i] = 0;
      }

      for (var x = 1; x <= this.panel_size; x++) {
        for (var y = 1; y <= this.panel_size; y++) {
          if(this.xy[x][y] == this.canput_panel){
            this.xy[x][y] = this.blank_panel;
          } else if(this.xy[x][y] < this.user_count){
            this.counter[this.xy[x][y]]++;
          }
        }
      }
    },
    ans_red() {
      this.answer(this.red_panel)
    },
    ans_green() {
//      document.getElementById().play();
      this.answer(this.green_panel)
    },
    ans_white() {
      this.answer(this.white_panel)
    },
    ans_blue() {
      this.answer(this.blue_panel)     
    },
    answer(color){
      if(this.status != "start") {
        this.status = "answer";
      }
      this.clearPanel();

      if(!this.otetsuki[color]) {
        if( confirm(this.names[color] + "の回答") ) {
          this.wb = color;
          var list = this.getCanPutList(color);
          for (var x = 1; x <= this.panel_size; x++) {
            for (var y = 1; y <= this.panel_size; y++) {
              if(list.some(b => b.x === x && b.y === y)
                && this.xy[x][y] != this.attack_panel) {
                this.xy[x][y] = this.canput_panel;
              }
            }
          }
          this.status = "put";
        } else {
          //sound failed
          alert(this.names[color] + "は1回休みになるよ")
          this.otetsuki[color] = true;
          this.status = "normal";
        }
      } else {
          alert(this.names[color] + "は1回休み中です")
          this.status = "normal";
      }
    },
  },
}

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

.attack {
  border: 35px;
  background-color: yellow;
  display: table-cell;
  vertical-align: middle;
}

.member {
  list-style: none;
  overflow: hidden;
}

.member button a {
  text-decoration: none;
  color: #fff;
  font-weight: bold;
}

.button_red {
  width: 160px;
  text-align: center;
  background-color: red;
  display: inline-block;
}

.button_white {
  width: 160px;
  text-align: center;
  background-color: white;
  display: inline-block;
}

.button_green {
  width: 160px;
  text-align: center;
  background-color: green;
  display: inline-block;
}

.button_blue {
  width: 160px;
  text-align: center;
  background-color: blue;
  display: inline-block;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave {
  opacity: 0
}
</style>
