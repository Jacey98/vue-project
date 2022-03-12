<template>
  <div class="main">
    <!-- 功能区 -->
    <div class="tool-content">
      <span class="btn" @click="changeLevel">更换难度</span>
      <span>雷数：{{ gameInfo[2] - markNum }}</span>
      <span>用时：{{ time }} s</span>
      <span class="btn" @click="reStart">重新开始</span>
    </div>
    <!-- 游戏区 -->
    <div class="game-content">
      <div
        v-for="col in cols"
        :key="Math.random() + col"
        class="game-content-row"
      >
        <span
          v-for="row in rows"
          :key="Math.random() + row"
          class="game-block"
          :class="[
            lattice[(col - 1) * rows + row - 1].isOpen ? 'open' : '',
            over !== 0 &&
            (lattice[(col - 1) * rows + row - 1].isMark ||
              lattice[(col - 1) * rows + row - 1].isMine)
              ? 'open'
              : '',
          ]"
          @click.left="leftClick(lattice[(col - 1) * rows + row - 1])"
          @click.right.prevent="rightClick(lattice[(col - 1) * rows + row - 1])"
          @dblclick="ondblClick(lattice[(col - 1) * rows + row - 1])"
        >
          <span v-if="over !== 0 && lattice[(col - 1) * rows + row - 1].isMine"
            >💣</span
          >
          <span
            v-else-if="over === 0 && lattice[(col - 1) * rows + row - 1].isMark"
            >🚩</span
          >
          <span v-else-if="lattice[(col - 1) * rows + row - 1].mineNum !== 0">{{
            lattice[(col - 1) * rows + row - 1].mineNum
          }}</span>
          <span v-else>&nbsp;</span>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "mineSweeping",
  components: {},
  props: {
    // 是否展示游戏盘
    showGame: {
      required: true,
      type: Boolean,
    },
    // 游戏盘格子数和雷数
    gameInfo: {
      required: true,
      type: Array,
    },
  },
  data() {
    return {
      // 横排格子数
      rows: 0,
      // 纵排格子数
      cols: 0,
      // 格子总数
      latticeNum: 0,
      // 标记总数
      markNum: 0,
      // 点开总数
      openNum: 0,
      // 雷点位置（找出一个少一个）
      minePosition: [],
      // 雷点位置备个份
      minePositionBake: [],
      // 格子属性
      lattice: {
        index: 0, // 格子索引
        mineNum: 0, // 周围雷数
        isMine: false, // 是否是雷
        isOpen: false, // 是否已经被点开
        isMark: false, // 是否被标记
      },
      // 游戏是否结束： 0-未结束 1-点到雷了导致结束  2-雷点被标记完了
      over: 0,
      // 游戏计时
      time: 0,
      // 游戏计时器
      interval: null,
      // 成功局数
      successNum: 0,
      // 失败局数
      failNum: 0,
    };
  },
  computed: {
    timeText() {
      return this.time;
    },
  },
  watch: {
    over(newVal) {
      switch (newVal) {
        case 1:
          this.failNum++;
          clearInterval(this.interval);
          setTimeout(() => {
            alert(`
            很遗憾，您失败了

            成功局数：${this.successNum}
            失败局数：${this.failNum}
            `);
          }, 0);
          break;
        case 2:
          this.successNum++;
          clearInterval(this.interval);
          setTimeout(() => {
            alert(`
            恭喜，扫雷成功！

            成功局数：${this.successNum}
            失败局数：${this.failNum}
            `);
          }, 0);
          break;
      }
    },
  },
  created() {
    this.rows = this.gameInfo[0];
    this.cols = this.gameInfo[1];
    this.latticeNum = this.rows * this.cols;
    this.init();
    document.onselectstart = function () {
      return false;
    };
  },
  mounted() {
    this.setTime();
  },
  beforeDestroy() {
    clearInterval(this.interval);
  },
  methods: {
    init() {
      this.markNum = 0;
      this.openNum = 0;
      this.minePosition = [];
      this.minePositionBake = [];
      this.lattice = {
        index: 0, // 格子索引
        mineNum: 0, // 周围雷数
        isMine: false, // 是否是雷
        isOpen: false, // 是否已经被点开
        isMark: false, // 是否被标记
      };
      this.over = 0;
      this.time = 0;
      // 获取雷点位置
      this.getMinePosition();
      // 给每个格子赋予正确的属性
      this.initLattice();
    },
    // 开始计时
    setTime() {
      clearInterval(this.interval);
      this.interval = setInterval(() => {
        this.time++;
      }, 1000);
    },
    // 随机获取雷点位置
    getMinePosition() {
      // 定义一个数组装不重复的格点
      let mineArr = [];
      // 循环雷数生成不重复的雷点
      for (let n = 0; n < this.gameInfo[2]; n++) {
        const random = Math.floor(Math.random() * this.latticeNum);
        if (mineArr.indexOf(random) === -1) {
          mineArr.push(random);
        } else {
          n--;
        }
      }
      this.minePosition = mineArr;
      this.minePositionBake = [].concat(mineArr);
    },
    // 格子属性初始化
    initLattice() {
      let latticeArr = [];
      for (let n = 0; n < this.latticeNum; n++) {
        let lattice = {
          index: n,
          isOpen: false,
          mineNum: 0,
          isMark: false,
        };
        // n标记是否是雷
        lattice.isMine = this.minePosition.indexOf(n) > -1;
        // 如果不是雷，计算出格子周围8个点的雷数
        if (!lattice.isMine) {
          this.getMineNumAroundLattice(lattice, n);
        }
        latticeArr.push(lattice);
      }
      this.lattice = latticeArr;
    },
    // 获取格子周围的雷数，
    getMineNumAroundLattice(lattice, index) {
      // 先获取格子周围的有效索引
      const latticeIndexArr = this.getLatticeIndex(index);
      // 循环索引，索引值在雷点数组中的，即为雷，当前格子的雷点数加1
      latticeIndexArr.forEach((i) => {
        if (this.minePosition.indexOf(i) > -1) {
          lattice.mineNum++;
        }
      });
    },
    // 获取格子周围的有效索引
    getLatticeIndex(index) {
      // 0做索引不好算，按正常数字来算
      index++;
      // 存索引值的变量
      let latticeIndexArr = [];
      // 当前格子位于第几行
      const latticeRow = Math.ceil(index / this.rows);
      // 当前格子位于第几列（求余为0说明是最右边一列）
      const latticeCol = Math.ceil(index % this.rows) || this.rows;
      // const latticeCol = Math.ceil(index % this.rows) === 0 ? this.rows : Math.ceil(index % this.rows);
      // 第一行没有上一行，不需要计算减1的行值，最后一行没有下一行，不需要计算加1的行值
      for (
        let i = latticeRow === 1 ? 0 : -1;
        i < (latticeRow === this.cols ? 1 : 2);
        i++
      ) {
        // 第一列没有左列，不需要计算减1的列值，最后一列没有右列，不需要计算加1的列值
        for (
          let j = latticeCol === 1 ? 0 : -1;
          j < (latticeCol === this.rows ? 1 : 2);
          j++
        ) {
          // 索引值 = (当前行值 + （上一行【-1】/当前行【0】/下一行【+1】） - 1【1是索引从0开始，所以需要减去】) * 每行格子数 + 当前列值 + （上一列【-1】/当前列【0】/下一列【+1】）
          const latticeIndex =
            (latticeRow + i - 1) * this.rows + (latticeCol + j);
          // 初始值加了1，所以索引要减去1才对
          latticeIndexArr.push(latticeIndex - 1);
        }
      }
      return latticeIndexArr;
    },
    // 左键点击
    leftClick(lattice) {
      if (this.over) {
        return false;
      }
      // 是雷，提前结束战斗
      if (!lattice.isOpen && lattice.isMine) {
        lattice.isOpen = true;
        this.over = 1;
      } else {
        this.openNum++;
        // 是数字
        if (lattice.mineNum) {
          if (!lattice.isOpen && !lattice.isMark) {
            lattice.isOpen = true;
          }
        } else {
          // 是空白
          const latticeIndexArr = this.getLatticeIndex(lattice.index);
          this.showWhiteAround(lattice, latticeIndexArr);
        }
      }
    },
    // 右键确认是雷点
    rightClick(lattice) {
      if (this.over) {
        return false;
      }
      if (!lattice.isOpen) {
        lattice.isMark = true;
        lattice.isOpen = true;
        this.markNum++;
        this.openNum++;
        if (lattice.isMine)
          this.minePosition.splice(this.minePosition.indexOf(lattice.index), 1);
      } else if (lattice.isMark) {
        lattice.isMark = false;
        lattice.isOpen = false;
        this.markNum--;
        this.openNum--;
        if (lattice.isMine) this.minePosition.push(lattice.index);
      }
      if (this.markNum === this.gameInfo[2]) this.judgeIsOver();
    },
    // 当前格子周围的雷点被正确找出时，双击填充周围格子
    ondblClick(lattice) {
      console.log(1);
      // 首先判断当前格子已被点开
      if (lattice.isOpen && !lattice.isMark) {
        // 拿到周围有效格子索引
        const latticeIndexArr = this.getLatticeIndex(lattice.index);

        let markedNum = 0; // 标记的数量
        let mineAllMarked = false; // 是否有没有标记的雷
        let tempLatticeIndexArr = [];
        for (let i = 0; i < latticeIndexArr.length; i++) {
          if (this.lattice[latticeIndexArr[i]].isMark) {
            markedNum++;
          } else if (this.lattice[latticeIndexArr[i]].isMine) {
            mineAllMarked = true;
          } else {
            tempLatticeIndexArr.push(latticeIndexArr[i]);
          }
        }
        if (markedNum === lattice.mineNum) {
          if (mineAllMarked) this.over = 1;
          else {
            tempLatticeIndexArr.forEach((item) => {
              this.lattice[item - 1].isOpen = true;
            });
          }
        }
      }
    },
    // 判断游戏是否结束
    judgeIsOver() {
      this.over = this.minePosition.length === 0 ? 2 : 0;
    },
    // 判断错误的标记
    judgeWrongMark() {
      let wrongMark = 0;
      this.minePositionBake.forEach((item) => {
        if (!this.lattice[item].isMark) {
          wrongMark++;
        }
      });
      return wrongMark;
    },
    // 展示周围的空白标记，直至边缘（格子边缘或者数字）
    showWhiteAround(lattice, latticeIndexArr) {
      // 避免有重复的数据停不下来，去个重
      latticeIndexArr = [...new Set(latticeIndexArr)];
      for (let i = 0; i < latticeIndexArr.length; i++) {
        const item = latticeIndexArr[i];
        // 计算一个少一个，减少循环
        latticeIndexArr.splice(i, 1);
        i--;
        if (this.lattice[item].isOpen) {
          continue;
        }
        this.lattice[item].isOpen = true;
        if (!this.lattice[item].mineNum) {
          const arr = this.getLatticeIndex(this.lattice[item].index);
          this.showWhiteAround(this.lattice[item], latticeIndexArr.concat(arr));
        }
      }
    },
    // 重开一局
    reStart() {
      this.init();
      this.setTime();
    },
    // 改变难度
    changeLevel() {
      this.$emit("update:showGame", false);
    },
  },
};
</script>

<style lang="scss" scoped>
.main {
  width: 80%;
  margin: auto;
  margin-top: 100px;
  background-color: rgb(241, 241, 241);
  border: 5px dashed silver;
  border-radius: 10px;
  .tool-content {
    margin: 20px 5%;
    height: 30px;
    line-height: 30px;
    display: flex;
    justify-content: space-between;
    .btn {
      padding: 1px 10px;
      background-color: rgb(209, 204, 204);
      border-radius: 2px;
      text-align: center;
      cursor: pointer;
    }
  }
  .game-content {
    margin: 10px;
    overflow: auto;
    max-height: 70vh;
    .game-content-row {
      margin: 0 auto;
      width: fit-content;
      white-space: nowrap;
      .game-block {
        display: inline-block;
        margin: 1px;
        background-color: #d8afafcc;
        width: 30px;
        height: 30px;
        line-height: 30px;
        font-size: 14px;
        color: rgb(85, 69, 69);
        text-align: center;
        cursor: pointer;
        & > span {
          visibility: hidden;
        }
        &.open {
          background-color: #f7d8d79f;
          & > span {
            visibility: visible;
          }
        }
      }
    }
  }
}
</style>