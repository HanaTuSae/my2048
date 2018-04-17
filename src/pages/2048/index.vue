<template>
  <div class="container">
    <div class="header">
      <div class="left">
        <div class="title">{{title}}</div>
        <button class="button" bindtap="default" @click="start">{{buttonText}}</button>
      </div>
      <div class="right">
        <div class="score-main">
          <div class="score-text">{{scoreText}}</div>
          <div class="score">{{score}}</div>
        </div>
      </div>
    </div>
    <div class="play-game" @touchstart="gameTouchStart" @touchmove="gameTouchMove" @touchend="gameTouchEnd">
      <div class="game" >
        <div class="row" v-for="(row,index) in num" :key="row.index">
          <div class="cell" v-for="(cell,indexs) in row" :key="cell.indexs">
            <div class="cell-on" :class="'cell-on-' + cell">{{cell}}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="game-over" v-show="isGameOver">
      <div class="best-score">历史最高分: {{bestScore}}</div>
      <div class="current-score">本次得分: {{score}}</div>
      <div class="msg">{{msg}}</div>
      <button class="button" bindtap="default" @click="start">{{buttonText}}</button>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      isGameOver: false,
      title: 2048,
      buttonText: '开始游戏',
      scoreText: 'score',
      score: 0,
      bestScore: 0,
      num: [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]],
      startX: 0,
      startY: 0,
      moveEndX: 0,
      moveEndY: 0,
      msg: '游戏结束！'
    }
  },
  created () {
    if (!wx.getStorageSync('bestScore')) {
      wx.setStorageSync('bestScore', this.bestScore)
    }
  },
  computed: {
  },
  mounted () {
  },
  methods: {
    start () {
      if (this.buttonText === '开始游戏') {
        this.num = this.initNum()
        for (let i = 0; i < 2; i++) {
          this.randomNumVal()
        }
        this.buttonText = '重新开始'
      } else if (this.buttonText === '重新开始') {
        this.isGameOver = false
        this.score = 0
        this.num = this.initNum()
        for (let i = 0; i < 2; i++) {
          this.randomNumVal()
        }
      }
    },
    gameOver () {
      this.isGameOver = true
      this.buttonText = '重新开始'
      if (this.score >= 2048) {
        this.msg = '恭喜达到2048！'
      } else if (this.score > this.bestScore) {
        this.msg = '创造新记录！'
        this.bestScore = this.score
      } else {
        this.msg = '游戏结束！'
      }
      wx.setStorageSync('bestScore', this.bestScore)
    },
    // 判断是否游戏结束
    isPlayOver () {
      let cell = this.useCell()
      let arr = [...this.num]
      if (cell.length !== 0) {
        return false
      } else {
        for (let i = 0; i < 4; i++) {
          for (let j = 1; j < 4; j++) {
            if (arr[i][j] === arr[i][j - 1]) {
              return false
            }
          }
        }
        for (let i = 0; i < 4; i++) {
          for (let j = 1; j < 4; j++) {
            if (arr[j][i] === arr[j - 1][i]) {
              return false
            }
          }
        }
        return true
      }
    },
    // 初始化数组
    initNum () {
      let arr = new Array(4)
      for (let i = 0; i < 4; i++) {
        arr[i] = new Array(i)
        for (let j = 0; j < 4; j++) {
          arr[i][j] = 0
        }
      }
      return arr
    },
    // 记录值为0的格子
    useCell () {
      let cell = []
      let arr = [...this.num]
      for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 4; j++) {
          if (arr[i][j] === 0) {
            cell.push({
              x: i,
              y: j
            })
          }
        }
      }
      return cell
    },
    // 随机选择一个格子
    randomSelectCell () {
      let cell = this.useCell()
      if (cell.length) {
        return cell[Math.floor(Math.random() * cell.length)]
      }
    },
    // 随机选择一个格子赋值2或4
    randomNumVal () {
      let cell = this.randomSelectCell()
      let num = Math.random() < 0.8 ? 2 : 4
      this.num[cell.x][cell.y] = num
    },
    gameTouchStart (ev) {
      ev.preventDefault()
      ev.stopPropagation()
      this.startX = ev.mp.changedTouches[0].pageX
      this.startY = ev.mp.changedTouches[0].pageY
    },
    gameTouchMove (ev) {
      ev.preventDefault()
      ev.stopPropagation()
      this.moveEndX = ev.mp.changedTouches[0].pageX
      this.moveEndY = ev.mp.changedTouches[0].pageY
    },
    gameTouchEnd (ev) {
      ev.preventDefault()
      ev.stopPropagation()
      let x = this.moveEndX - this.startX
      let y = this.moveEndY - this.startY
      let X = Math.abs(x)
      let Y = Math.abs(y)
      if (this.isPlayOver()) {
        this.gameOver()
      } else {
        if (this.moveEndX !== 0) {
          if (Math.max(X, Y) > 50) {
            let flag = X > Y ? (x > 0 ? 1 : 3) : (y > 0 ? 2 : 0)// 确定移动方向
            let list = this.moveNum(flag)
            let cell = this.useCell()
            this.num = list
            this.score = this.scoreCount(list)
            // 如果还有空格子则随机选择一个赋值
            if (cell.length && cell.length !== 16) {
              this.randomNumVal()
            }
            if (this.isPlayOver()) {
              this.gameOver()
            }
          }
        }
      }
      this.startX = 0
      this.startY = 0
      this.moveEndX = 0
      this.moveEndY = 0
    },
    // 根据移动方向将计算好的数组还原 0:上 1:右 2:下 3:左
    moveNum (i) {
      let arr = this.moveList(i)
      arr = this.addNum(arr)
      let list = [[], [], [], []]
      for (let m = 0; m < 4; m++) {
        for (let n = 0; n < 4; n++) {
          switch (i) {
            case 0:
              list[m][n] = arr[n][m]
              break
            case 1:
              list[m][n] = arr[m][3 - n]
              break
            case 2:
              list[m][n] = arr[n][3 - m]
              break
            case 3:
              list[m][n] = arr[m][n]
              break
          }
        }
      }
      return list
    },
    // 移动后计算本次游戏得分
    scoreCount (arr) {
      // let score = 0
      // let list = [[], [], [], []]
      // for (let i = 0; i < 4; i++) {
      //   for (let j = 0; j < 4; j++) {
      //     list[i] = arr[i][j]
      //   }
      //   let maxNum = list[i].sort((a, b) => {
      //     return b - a
      //   })[0]
      //   score = score > maxNum ? score : maxNum
      // }
      // return score
      let list = arr.map(Function.apply.bind(Math.max, null)) // 二维数组求最大值返回一个一维数组
      return list.sort((a, b) => {
        return b - a
      })[0]
    },
    // 根据移动方向生成新的数组
    moveList (i) {
      let arr = [...this.num]
      let list = [[], [], [], []]
      for (let m = 0; m < 4; m++) {
        for (let n = 0; n < 4; n++) {
          switch (i) {
            case 0:
              list[m].push(arr[n][m])
              break
            case 1:
              list[m].push(arr[m][3 - n])
              break
            case 2:
              list[m].push(arr[3 - n][m])
              break
            case 3:
              list[m].push(arr[m][n])
              break
          }
        }
      }
      return list
    },
    // 数字靠边
    numSide (arr) {
      let k = 0
      for (let m = 0; m < 4; m++) {
        if (arr[m] !== 0) {
          arr[k++] = arr[m]
        }
      }
      for (let n = k; n < 4; n++) {
        arr[n] = 0
      }
      return arr
    },
    // 相邻相同的数字合并
    addNum (arr) {
      for (let i = 0; i < 4; i++) {
        arr[i] = this.numSide(arr[i])
      }
      for (let m = 0; m < 4; m++) {
        for (let n = 1; n < 4; n++) {
          if (arr[m][n] === arr[m][n - 1] && arr[m][n] !== 0) {
            arr[m][n - 1] += arr[m][n]
            arr[m][n] = 0
          }
        }
      }
      for (let i = 0; i < 4; i++) {
        arr[i] = this.numSide(arr[i])
      }
      return arr
    }
  }
}

</script>
<style>
.header,.play-game {
  width: 100%;
}
.header,.left,.right {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}
.left {
  flex-direction: column;
}
.title {
  font-size: 48px;
  text-align: center;
  font-weight: bold;
  color: #333;
}
.button {
  width: 130px;
  color: #fafafa;
  background-color: #bbada0;
  font-size: 20px;
  text-align: center;
  font-weight: bold;
  margin-top: 20px;
}
.score-main {
  width: 120px;
  height: 120px;
  background-color: #bbada0;
  opacity: 0.7;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
.score-text,.score {
  font-size: 30px;
  font-weight: bold;
  text-align: center;
}
.score-text {
  color: #333;
}
.score {
  color: #fafafa;
}
.play-game {
  flex: 2;
  display: flex;
  align-items: center;
  justify-content: center;
}
.game {
  width: 93vw;
  height: 93vw;
  font-size: 20px;
  background-color: #bbada0;
  color: #fff;
  position: absolute;
  padding-top: 5vw;
  padding-left: 1vw;
  border-radius: 6px;
  text-align: center;
  box-sizing: border-box;
  user-select: none;/* css控制文字不能被选中 */
  touch-callout: none;/* 禁用系统默认菜单 */
  touch-action: none;/* 当触控事件发生在元素上时，不进行任何操作 */
}
.game-over {
  width: 100%;
  height: 100%;
  z-index: 10;
  box-sizing: border-box;
  position: absolute;
  left: 0;
  top:0;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  background-color: rgba(255, 255, 255, 0.8);;
}
.best-score,.current-score {
  font-size: 30px;
  margin-bottom: 10px;
}
.msg {
  font-weight: bold;
  font-size: 30px;
}
.cell {
  width: 19vw;
  height: 19vw;
  margin-left: 3vw;
  margin-bottom: 3vw;
  float: left;
  border-radius: 5px;
  box-sizing: border-box;
  background: rgba(238, 228, 218, 0.35);
  overflow: hidden;
}
.cell-on {
  width: 19vw;
  height: 19vw;
  line-height: 19vw;
  position: absolute;
  font-size: 35px;
  font-weight: bold;
  text-align: center;
  border-radius: 5px;
}
.cell-on-0 {
  /*visibility: hidden;*/
  display: none;
}
.cell-on-2 {
  background: #eee4da;
}
.cell-on-4 {
  background: #ede0c8;
}
.cell-on-8 {
  color: #f9f6f2;
  background: #f2b179;
}
.cell-on-16 {
  color: #f9f6f2;
  background: #f59563;
}
.cell-on-32 {
  color: #f9f6f2;
  background: #f67c5f;
}
.cell-on-64 {
  color: #f9f6f2;
  background: #f65e3b;
}
.cell-on-128 {
  color: #f9f6f2;
  background: #edcf72;
  font-size: 30px;
}
.cell-on-256 {
  color: #f9f6f2;
  font-size: 30px;
  background: #edcc61;
}
.cell-on-512 {
  color: #f9f6f2;
  font-size: 30px;
  background: #edc850;
}
.cell-on-1024 {
  color: #f9f6f2;
  font-size: 25px;
  background: #edc53f;
}
.cell-on-2048 {
  color: #f9f6f2;
  font-size: 25px;
  background: #edc22e;
}

</style>
