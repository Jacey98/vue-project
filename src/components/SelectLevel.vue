<template>
  <div class="main">
    <div>难 度</div>
    <ul>
      <li
        v-for="(item, index) in level"
        :key="index"
        @click="handleChoseLevel(item.value)"
      >
        {{ item.text }}
      </li>
      <li @click="showForm = !showForm">🌞 自定义</li>
    </ul>
    <auto-form
      v-if="showForm"
      :form-info="formInfo"
      class="form"
      @submit="submit"
    ></auto-form>
  </div>
</template>

<script>
import AutoForm from "@/components/AutoForm.vue";
export default {
  name: "mineSweeping",
  components: { AutoForm },
  data() {
    return {
      // 显示表单
      showForm: false,
      // 难度
      level: [
        {
          text: "⭐ LEVEL1", // 难度描述
          value: [9, 9, 10], // 格子横排数，格子纵排数，雷数
        },
        {
          text: "⭐ LEVEL2",
          value: [16, 9, 20],
        },
        {
          text: "⭐ LEVEL3",
          value: [16, 16, 30],
        },
      ],
      formInfo: [
        {
          check: 1, // 校验方式
        },
        [
          {
            id: "001", // 唯一编号
            name: "col",
            type: "number", // 表单类型
            title: "列数",
            prompt_msg: "输入3-30之间的列数", // placeholder
          },
          {
            id: "002",
            name: "row",
            type: "number",
            title: "行数",
            prompt_msg: "输入3-30之间的行数",
          },
          {
            id: "003",
            name: "mine",
            type: "number",
            title: "雷数",
            prompt_msg: "输入合适的地雷数量",
          },
        ],
      ],
    };
  },
  methods: {
    // 选择难度
    handleChoseLevel(level) {
      this.$emit("chose-level", level);
    },
    submit(val) {
      this.$emit("chose-level", [val["001"], val["002"], val["003"]]);
    },
  },
};
</script>

<style lang="scss" scoped>
.main {
  width: 40%;
  margin: auto;
  margin-top: 100px;
  padding-bottom: 24px;
  background-color: rgb(241, 241, 241);
  border: 5px dashed silver;
  border-radius: 10px;
  * {
    text-align: center;
  }
  div {
    height: 60px;
    line-height: 80px;
    font-size: 18px;
  }
  ul {
    padding: 0;
    li {
      margin: 10px auto;
      cursor: pointer;
      list-style: none;
    }
  }
  .form {
    display: inline;
    line-height: 30px;
    font-size: 16px;
  }
}
</style>