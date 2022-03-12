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
    <auto-form v-if="showForm" :form-info="formInfo"></auto-form>
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
      formInfo: [{
        name: 'height',
        type: "integer", // select_item下拉列表、string单行文本、multiple多行文本、integer单行数字
        title: "高度",
        prompt_msg: '请输入1-99之间的数字',
        selectObj: null,
        check: 1 // 校验方式的编号
      },{
        name: 'height',
        type: "integer", 
        title: "宽度",
        prompt_msg: '请输入1-99之间的数字',
        selectObj: null,
        check: 1 
      }，{
        name: 'height',
        type: "integer", 
        title: "雷数",
        prompt_msg: '请输入1-99之间的数字',
        selectObj: null,
        check: 1
      }],
    }
  },
  methods: {
    // 选择难度
    handleChoseLevel(level) {
      this.$emit("chose-level", level);
    },
  },
};
</script>

<style lang="scss" scoped>
.main {
  width: 40%;
  margin: auto;
  margin-top: 100px;
  background-color: rgb(241, 241, 241);
  border: 5px dashed silver;
  border-radius: 10px;
  div {
    height: 60px;
    line-height: 80px;
    text-align: center;
    font-size: 18px;
  }
  ul {
    padding: 0;
    margin-bottom: 30px;
    li {
      text-align: center;
      margin: 10px auto;
      cursor: pointer;
      list-style: none;
    }
  }
}
</style>