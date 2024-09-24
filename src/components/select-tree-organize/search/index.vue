<template>
  <div class="filterBox">
      <Input v-model="inputVal"  @on-keyup="handleFilter" search placeholder="搜索"></Input>
    </div>
</template>

<script>
export default {
  name: 'SelectTree',
  data() {
    return {
      inputVal: "",
    };
  },
  methods: {
    handleFilter() {
      this.$emit("confirm", this.inputVal);
    },
    clears(){
      this.inputVal=""
    }
  },
};
</script>

<style lang="scss" scoped>
.filterBox {
  padding: 20px 20px 10px;
  background-color: #fff;
}
// @import url("../css/icon.css");
</style>
