<template>
  <div>
    <div v-for="(item, index) in formInfo[1]" :key="item.id">
      {{ item.title }} :
      <input
        :type="item.type"
        :name="item.name"
        :placeholder="item.prompt_msg"
        style="width: 136px"
        autocomplete="off"
        @input="changeHandle($event, item.id)"
      />
    </div>
    <div v-if="showMsg" class="error">{{ verify.msg }}</div>
    <input type="button" value="确定" class="btn" @click="submit" />
  </div>
</template>

<script>
export default {
  name: "autoForm",
  components: {},
  props: {
    formInfo: {
      required: true,
      type: Array,
    },
  },
  data() {
    return {
      val: {},
      showMsg: false,
      verify: {},
    };
  },
  methods: {
    changeHandle(e, id) {
      this.val[id] = Number(e.target.value);
    },
    submit() {
      if (this.formInfo[0].check === 1) this.veri();
      if (!this.verify.flag) {
        this.showMsg = true;
        setTimeout(() => (this.showMsg = false), 500);
        return;
      }
      this.$emit("submit", this.val);
    },
    veri() {
      if (!this.val["001"] || this.val["001"] < 3 || this.val["001"] > 30) {
        this.verify.flag = false;
        this.verify.msg = "请输入正确的列数";
        return;
      }
      if (!this.val["002"] || this.val["002"] < 3 || this.val["002"] > 30) {
        this.verify.flag = false;
        this.verify.msg = "请输入正确的行数";
        return;
      }
      if (
        !this.val["003"] ||
        this.val["003"] < 0 ||
        this.val["003"] > this.val["001"] * this.val["002"]
      ) {
        this.verify.flag = false;
        this.verify.msg = "请输入正确的雷数";
        return;
      }
      this.verify.flag = true;
      this.verify.msg = "ok";
    },
  },
};
</script>

<style lang="scss" scoped>
.btn {
  display: block;
  margin: 0 auto;
  margin-top: 10px;
}
.error {
  font-size: 16px;
  line-height: 24px;
  color: red;
}
</style>