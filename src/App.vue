<template>
  <div id="app">
    <selectTreeOrganize
      :checkList="checkList"
      v-if="tree.length > 0"
      :props="prop"
      @sendValue="confirm"
      :isCheck="true"
      :treeNone="tree"
    >
    ></selectTreeOrganize>
  </div>
</template>

<script>
import selectTreeOrganize from "./components/select-tree-organize";
import { treeNode } from "./data.js";
export default {
  name: 'App',
  data() {
    return {
      tree: [],
      checkList: [],
      prop: {
        label: "name",
        children: "children",
        multiple: true,
        checkStrictly: true,
        hasPath: true,
      },
    };
  },
  components: {
    selectTreeOrganize
  },
  created() {
    this.tree = treeNode; // 树形数据赋值
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
