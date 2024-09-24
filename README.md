# vue2-selectTree

一款无限级树形结构多项选择器，支持多选、全选、回显选项、搜索等多功能选择组件。

# 效果图

<div align="center">
  <img src="https://oss.jixianku.com/images/goods/goods-1727164144416.jpg" width="70%">
</div>
<div align="center">
  <img src="https://oss.jixianku.com/images/goods/goods-1727164242554.jpg" width="70%">
</div>
<div align="center">
  <img src="https://oss.jixianku.com/images/goods/goods-1727164273340.jpg" width="70%">
</div>


# 安装

## Node.js版本
```
node: 14+
```

## UI框架
```
iView： 轻度依赖，可根据项目自行修改Button、Input等组件
```

## 安装
```
yarn install
```

### 运行
```
npm run serve
```

## 使用方法
```Javascript
     <template>
      <selectTreeOrganize
        :checkList="checkList"
        v-if="tree.length > 0"
        :props="prop"
        @sendValue="confirm"
        :isCheck="true"
        :treeNone="tree"
      >
      ></selectTreeOrganize>
     </template>
    <script>
      import selectTreeOrganize from "./components/select-tree-organize";
      import { treeNode } from "./data.js";
      export default {
        name: 'App',
        data() {
          return {
            tree: [],
            checkList: [], //默认选中值
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
```

# P.S.
基于（https://github.com/LSZ579/xiaolu-tree-plugin）-uniapp小程序树形选择版本修改