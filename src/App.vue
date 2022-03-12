<template>
  <mine-sweeping
    v-if="showGame"
    :game-info="gameInfo"
    :show-game.sync="showGame"
  ></mine-sweeping>
  <select-level v-else @chose-level="choseLevel"></select-level>
</template>

<script>
import SelectLevel from "@/components/SelectLevel.vue";
import MineSweeping from "@/components/MineSweeping.vue";
export default {
  name: "app",
  components: {
    SelectLevel,
    MineSweeping,
  },
  data() {
    return {
      // 默认展示游戏界面
      showGame: true,
      // 初始行列数及雷数
      gameInfo: [9, 9, 10],
    };
  },
  methods: {
    // 选择难度
    choseLevel(level) {
      this.gameInfo = level;
      this.showGame = true;
    },
  },
};
</script>	
