<template>
  <div class="tree-pannel">
    <div class="header">
      <search v-if="searchIf" ref="sea" @confirm="confirmSearch" />
      <div class="select-tree-pannel">
        <div class="scroll-wrapper" :scroll-left="scrollLeft">
          <div v-for="(item, index) in tree_stack" class="inline-item-model" :key="index">
            <div v-if="index === 0" class="inline-item-li" @click="backTree(item, -1)">
              <div :class="[isActive(index) && !is_search ? 'none' : 'active','nav-item']">
                <p class="name">全部</p>
                <p class="arror">></p>
              </div>
            </div>
            <div
              v-if="index === 0 && is_search"
              @click="backTree(item, -2)"
              :class="[
                index === tree_stack.length - 1 && is_search ? 'none' : 'active',
                'inline-item-li',
              ]"
            >
              <i class="iconfont icon-z043 iconclass"></i> 搜索结果
            </div>
            <div class="inline-item-li" v-if="index !== 0" @click="backTree(item, index)">
              <span :class="[isActive(index) ? 'none' : 'active', 'nav-item']">
                <p class="name">{{ item[props.label] }}</p>
                <p class="arror">></p>
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="main-pannel">
      <div class="main-wrapper" :style="{ height: scrollHeight, 'overflow-y': 'scroll' }">
        <div class="container-list">
          <div
            class="container-li"
            v-for="(item, index) in tree"
            :key="item[keyValue]"
            @click="handleClick(item, index)"
          >
            <div class="content">
              <div class="list-item" v-show="isCheck">
                <!-- 多选 -->
                <div
                  class="checkbox"
                  v-if="props.multiple"
                  @click.stop="handleClick(item, -1)"
                >
                  <span v-if="isSelect(item)">
                    <i
                      v-if="item.bx && newCheckList.length !== 0"
                      class="iconfont txt icon-not-select-all"
                    ></i>
                    <i v-else class="iconfont txt icon-selects"></i>
                  </span>
                  <i v-else-if="item.qx" class="iconfont txt icon-selects"></i>
                  <i
                    v-else-if="item.bx"
                    class="iconfont txt icon-not-select-all"
                  ></i>
                  <i
                    style="color: #b8b8b8"
                    v-else
                    class="iconfont icon-no-select txt"
                  ></i>
                </div>
                <!-- 单选 -->
                <!-- <div
                  class="checkbox"
                  v-else-if="!props.nodes || (props.nodes && item.user)"
                  @click.stop="handleClick(item, -1)"
                >
                  <i v-if="radioSelect(item)" class="txt iconfont icon-selects"></i>
                  <i
                    style="color: #b8b8b8"
                    v-else
                    class="txt iconfont icon-weixuanzhong1"
                  ></i>
                </div> -->
              </div>
              <div class="head" v-if="showHead">
                <div v-if="!item.user && item[props.children].length > 0" class="group"></div>
                <div v-else class="user">
                  {{ item[props.label].slice(-2) }}
                </div>
              </div>
              <div class="label-text">{{ item[props.label] }}</div>
              <div class="right">
                <p class="hav-children" @click="handleChildrenClick(item)" v-if="!item.user && item[props.children].length > 0">下级 ></p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div
        class="main-selected"
        :style="{ height: scrollHeight, 'overflow-y': 'scroll' }"
      >
        <div class="pannel-model">
          <h3 class="select-title">已选：{{ newCheckList.length }}个</h3>
          <ul class="select-ul">
            <li v-for="(item) in newCheckList" :key="item.id" class="select-li">
              <div class="user-msg">
                <div class="head" v-if="showHead">
                  <div v-if="!item.user && item[props.children].length > 0" class="group"></div>
                  <div v-else class="user">
                    {{ item[props.label].slice(-2) }}
                  </div>
                </div>
                <p class="name">{{ item.name }}</p>
              </div>
              <div class="close" @click="removeItem(item)"></div>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div class="btn">
      <i-button class="sureBtn red" type="primary" size="small" @click="backConfirm">确认</i-button>
      <i-button class="sureBtn" type="default" size="small" @click="cleanSelect">清空</i-button>
      <i-button class="sureBtn" type="default" size="small" @click="cancel">取消</i-button>
    </div>
  </div>
</template>

<script>
import Search from "./search";
export default {
  name: 'tree-component',
  data() {
    return {
      is_search: false,
      tree: Object.freeze(this.treeNone),
      newNum: 0,
      oldNum: 0,
      catchTreeNone: [...this.treeNone],
      tree_stack: [1],
      searchResult: [],
      newCheckList: [],
      scrollLeft: 999,
      nodePathArray: [],
    };
  },
  components: {
    Search,
  },
  props: {
    // 树形数据源
    treeNone: {
      type: Array,
      default: () => {
        return [];
      },
    },
    //是否开启选中
    isCheck: {
      type: Boolean,
      default: () => {
        return false;
      },
    },
    // 弹出模式时需传的列表滚动高度
    scrollHeight: {
      type: String,
      default: "500px",
    },
    // 选中的值的 必传[{[keyValue]:value}]
    checkList: {
      type: Array,
      default: () => [],
    },
    searchIf: {
      type: Boolean,
      default: () => true,
    },
    // 默认key
    keyValue: {
      type: String,
      default: "id",
    },
    props: {
      type: Object,
      default: () => {
        return {
          label: "name",
          children: "children",
          multiple: false,
          checkStrictly: false, //不关联
          hasPath: false, //返回值是否都带路径
        };
      },
    },
    //是否需要头像
    showHead: {
      type: Boolean,
      default: () => true
    }
  },
  watch: {
    treeNone(newValue, oldValue){
      console.log(newValue,oldValue,'00009')
      this.tree = Object.freeze(newValue)
    },
    checkList(newValue, oldValue) {
      console.log('myProp 发生变化:', oldValue, '=>', newValue);
      console.log(this.treeNone,'treeee')
      const baseCheckList = this.findNodesByIds(this.treeNone, newValue)
      console.log(baseCheckList,'baseCheckList')
      this.newCheckList = baseCheckList;
      // this.Init()
    }
  },
  computed: {
    isSelect() {
      return (item) => {
        const checkList = this.newCheckList;
        if (checkList.length == 0) {
          this.props.checkStrictly ? ((item.bx = 0), (item.qx = 0)) : "";
          return false;
        }
        const select = checkList.some((e) => item[this.keyValue] == e[this.keyValue]);
        return select && !item.qx;
      };
    },
    radioSelect() {
      const list = this.newCheckList;
      return (item) => {
        return list.length > 0 && item[this.keyValue] == list[0][this.keyValue];
      };
    },
    isActive() {
      return (index) => {
        return index === this.tree_stack.length - 1;
      };
    },
  },
  created() {
    if(this.checkList){
      const baseCheckList = this.findNodesByIds(this.treeNone, this.checkList)
      console.log(baseCheckList,'baseCheckList')
      this.newCheckList = baseCheckList;
    }
    this.Init();
  },
  methods: {
    Init() {
      console.log(this.newCheckList,'sssss')
      if (this.newCheckList.length !== 0) {
        let { tree_stack, props, catchTreeNone, newCheckList } = this,
          index = 0,
          flag = props.multiple && props.checkStrictly;
        flag && (index = newCheckList.length - 1);
        this.getNodeRoute(catchTreeNone, newCheckList[index][this.keyValue]);
        let arr = this.nodePathArray.reverse();
        if (arr.length == 0) return;
        this.tree_stack = tree_stack.concat(arr);
        this.tree = this.tree_stack[this.tree_stack.length - 1][this.props.children];
        flag && this.checkAllChoose();
      }
    },
    removeItem(item) {
      this.checkboxChange(item, -1, item.bx, item.qx);
      this.checkAllChoose()
    },
    // 点击项目处理
    handleClick(item, index) {
      let children = item[this.props.children];
      if (index > -1 && children && children.length > 0) {
        this.checkboxChange(item, index, item.bx, item.qx);
      } else if (this.props.multiple) {
        this.checkboxChange(item, index, item.bx, item.qx);
      } else {
        this.checkbox(item, index);
      }
    },
    handleChildrenClick(item) {
      this.toChildren(item);
    },
    findNodesByIds(tree, targetIds) {
      const foundNodes = []; // 存储找到的节点

      function findNodeById(tree, targetId) {
        for (const node of tree) {
          if (node.id === targetId) {
            foundNodes.push(node); // 找到匹配的节点，加入结果数组
            return; // 找到后无需继续查找当前分支
          }
          // 如果有子节点，递归遍历子节点
          if (node.children && node.children.length > 0) {
            findNodeById(node.children, targetId);
          }
        }
      }

      // 对每个 targetId 进行递归查找
      targetIds.forEach((target) => findNodeById(tree, target.id));

      return foundNodes; // 返回所有找到的节点
    },

    // 使用方法
    // const targetIds = [{ id: "1-1-1" }, { id: "1-2-1" }];
    // const foundNodes = findNodesByIds(treeNode, targetIds);
    // (tree为目标树，targetId为目标节点id)
    getNodeRoutePath(tree, targetId, nodePathArray = []) {
      for (let index = 0; index < tree.length; index++) {
        if (tree[index][this.props.children]) {
          let endRecursiveLoop = this.getNodeRoutePath(
            tree[index][this.props.children],
            targetId,
            nodePathArray
          );
          if (endRecursiveLoop) {
            nodePathArray.push(tree[index]);
            return true;
          }
        }
        if (tree[index][this.keyValue] === targetId) {
          return true;
        }
      }
    },
    // 获取路径
    getPath(id) {
      if (!this.props.hasPath) return "";
      const { tree_stack, props } = this;
      if (this.is_search) {
        // 搜索情况下，无法获取完整的路径，如果需要获取，则只能通过完整的树去查找
        let path = [];
        this.getNodeRoutePath(this.catchTreeNone, id, path);
        console.log(path);
        return path.reverse();
      }
      const path = [...tree_stack].map((e) => {
        const item = Object.assign({}, e);
        delete item[props.child];
        return item;
      });
      return path.slice(1, path.length) || [];
    },
    //多选
    async checkboxChange(item, index, bx, qx) {
      let that = this;
      const { props } = that;
      if (!props.multiple) return;
      let findIdex = that.newCheckList.findIndex(
        (e) => item[this.keyValue] == e[this.keyValue]
      );
      const path = this.getPath(item[this.keyValue]);
      if (findIdex > -1) {
        //反选
        if (props.checkStrictly) {
          //关联子级
          if (item.user) {
            //用户
            that.newCheckList.splice(findIdex, 1);
          } else {
            //非用户，取消所有下一级
            that.getIdBydelete(item[this.props.children]);
          }
        } else {
          that.newCheckList.splice(findIdex, 1);
        }
        // if (props.checkStrictly && !item.user) { //关联子级 非用户，取消所有下一级
        // 	item.qx = item.bx = 0
        // 	that.getIdBydelete(item[this.props.children])
        // }
      } else {
        //选中
        if (!item.user && props.checkStrictly) {
          //选中下一级
          if (qx || bx) {
            //取消下级
            await that.getIdBydelete(item[this.props.children]);
            item.qx = item.bx = 0;
            that.newCheckList.splice(findIdex, 1);
          } else {
            item.qx = 1;
            item.bx = 0;
            const { id, name, user } = item;
            const newObj = {
              id,
              name,
              user,
            };
            const pathList =
              this.tree_stack.length === 1 ? [newObj, ...path] : [...path, newObj];
            await that.chooseChild(item[this.props.children], pathList);
            // that.newCheckList.push({
            // 	...item,
            // 	paths
            // });
          }
          this.$forceUpdate();
          return;
        }
        that.newCheckList.push({
          ...item,
          path,
        });
      }
    },
    // 取消下一级的选中
    getIdBydelete(arr) {
      arr.forEach((e) => {
        if (e.user) {
          for (var i = 0; i < this.newCheckList.length; i++) {
            if (e[this.keyValue] == this.newCheckList[i][this.keyValue]) {
              this.newCheckList.splice(i, 1);
              break;
            }
          }
        } else {
          this.getIdBydelete(e[this.props.children]);
        }
      });
    },

    // 关联下一级,选中
    chooseChild(arr, path) {
      let that = this;
      const oldPath = [...path];
      for (var i = 0, len = arr.length; i < len; i++) {
        let item = arr[i];
        if (item.user) {
          that.newCheckList.push({
            ...item,
            path: oldPath,
          });
        } else {
          const newItem = {
            ...item,
          };
          delete newItem[that.props.children];
          const newPath = [...oldPath, newItem];
          that.chooseChild(item[this.props.children], newPath);
        }
      }
    },

    // (tree为目标树，targetId为目标节点id)
    getNodeRoute(tree, targetId) {
      for (let index = 0; index < tree.length; index++) {
        if (tree[index][this.props.children]) {
          let endRecursiveLoop = this.getNodeRoute(
            tree[index][this.props.children],
            targetId
          );
          if (endRecursiveLoop) {
            this.nodePathArray.push(tree[index]);
            return true;
          }
        }
        if (tree[index][this.keyValue] === targetId) {
          return true;
        }
      }
    },

    //单选
    checkbox(itemName) {
      const path = this.getPath(itemName[this.keyValue]);
      this.$set(this, "newCheckList", [
        {
          ...itemName,
          path,
        },
      ]);
    },
    //到下一级
    toChildren(item) {
      if (item.user) return;
      var that = this;
      // uni.showLoading({
      //   title: "加载中",
      // });
      let children = that.props.children;
      if (
        !item.user &&
        item[children].length > 0 &&
        !(that.tree_stack[0][this.keyValue] == item[this.keyValue])
      ) {
        that.tree = item[children];
        that.tree_stack.push(item);
      }
      this.$nextTick(() => {
        // uni.hideLoading();
        this.scrollLeft += 200;
      });
      if (this.props.checkStrictly) this.checkAllChoose();
    },
    search(data, keyword) {
      let that = this;
      let { label, children } = that.props;
      for (var i = 0, len = data.length; i < len; i++) {
        if (data[i][label].indexOf(keyword) >= 0) {
          that.searchResult.push(data[i]);
        }
        if (!data[i].user && data[i][children].length > 0) {
          that.search(data[i][children], keyword);
        }
      }
    },
    checkAllChoose() {
      this.tree.forEach((e, i) => {
        if (!e.user) {
          e.qx = e.bx = false;
          this.computAllNumber(e.children);
          // console.log(this.newNum,this.oldNum)
          if (this.newNum != 0 && this.oldNum != 0) {
            e.qx = this.newNum == this.oldNum;
            e.bx = !e.qx;
          }
          if (this.newNum != 0 && this.oldNum == 0) {
            this.$set(this.tree[i], "bx", false);
            this.$set(this.tree[i], "qx", false);
          }
          this.$forceUpdate();
          this.newNum = this.oldNum = 0;
        }
      });
    },
    computAllNumber(arr) {
      for (let j = 0; j < arr.length; j++) {
        var e = arr[j];
        this.checkSum(e[this.keyValue]);
        if (e.user) {
          this.newNum++;
        } else {
          this.computAllNumber(e.children);
        }
      }
    },
    checkSum(id) {
      for (let i = 0; i < this.newCheckList.length; i++) {
        if (id == this.newCheckList[i][this.keyValue]) {
          this.oldNum++;
          break;
        }
      }
    },
    //返回其它层
    backTree(item, index) {
      let that = this,
        tree_stack = that.tree_stack,
        max = 300;
      if (index === -1) {
        that.tree = that.catchTreeNone;
        that.tree_stack.splice(1, max);
        that.is_search = false;
        that.$refs.sea?.clears();
      } else if (index === -2) {
        //搜索
        that.tree = that.searchResult;
        that.tree_stack.splice(1, max);
      } else {
        if (tree_stack.length - index > 2) {
          that.tree_stack.splice(index + 1, max);
        } else if (index !== tree_stack.length - 1) {
          that.tree_stack.splice(tree_stack.length - 1, 1);
        }
        that.tree = item[that.props.children];
      }
      if (this.props.checkStrictly) this.checkAllChoose();
      this.$forceUpdate();
    },
    backConfirm() {
      this.$emit("sendValue", this.newCheckList, "back");
    },
    cleanSelect() {
      this.newCheckList = []
    },
    cancel() {
      this.$emit("cancel");
    },
    confirmSearch(val) {
      this.searchResult = [];
      this.search(this.catchTreeNone, val);
      this.is_search = true;
      this.tree_stack.splice(1, 1000);
      // uni.showLoading({
      //   title: "正在查找",
      // });
      setTimeout(() => {
        this.tree = this.searchResult;
        if (this.props.checkStrictly) this.checkAllChoose();
        // uni.hideLoading();
      }, 300);
    },
  },
};
</script>

<style lang="scss" scoped>
.tree-pannel {
  position: relative;
  padding-bottom: 70px;
}
.select-tree-pannel {
  margin: 0px 0 0;
  padding: 10px 20px;
}
.flex_between_center {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.checkbox {
  position: relative;
  margin-left: 10px;
  margin-right: 0px;
  .color {
    color: #00aaff;
    background-color: #00aaff;
  }
  .txt {
    font-size: 18px;
    width: 100%;
    height: 100%;
    display: flex;
  }
}
.checkBorder {
  border: 1px solid #ecdee4;
}
.header {
  width: 100%;
  background-color: #fff;
  z-index: 9999;
  .title {
    height: 35px;
    padding: 0 20px;
    line-height: 35px;
    font-size: 18px;
    background-color: #f5f5f5;
    color: #606064;
  }
}
.iconclass {
  display: inline-block;
  margin: 0 12px;
  color: #d0d4db;
  font-size: 28px;
}
.container-list {
  // overflow-y: scroll;
  overflow-x: hidden;
  margin-top: 10px;
  .container-li {
    background-color: #fff;
    border-bottom: 1px solid #f4f4f4;
    // padding-left: 10px;
    transition: all .5s;
    padding: 3px 0px 3px 6px;
    &:hover{
      background: #cd170a10;
    }
    .content {
      display: flex;
      align-items: center;
      height: 40px;
      width: 100%;
      cursor: pointer;
      // padding: 15px 0;
      position: relative;
      font-size: 20px;
      
      .label-text {
        margin-left: 8px;
        font-size: 14px;
        color: #5b5757;
        // width: 500px;
        word-break: break-all;
      }
      .right {
        position: absolute;
        right: 10px;
        color: #babdc3;
        font-size: 20px;
        .hav-children{
          font-size: 12px;
          color: #CD170A;
          padding: 3px 8px;
          transition: .5s all;
          border-radius: 5px;
          &:hover{
            background: #cd170ac0;
            color: #fff;
          }
        }
      }
    }
  }
}
.active {
  color: #CD170A !important;
  cursor: pointer;
}

.none {
  color: #666666;
}
// .icon-selects {
//   color: #0095f2 !important;
//   font-size: 18px !important;
//   border: none;
// }
.icons {
  color: #0095f2 !important;
  font-size: 18px !important;
}
.inline-item-model {
  display: inline-block;
  margin-right: 4px;
  &:nth-last-child(1){
    .nav-item{
      .arror{
        display: none;
      }
    }
  }
  .inline-item-li {
    display: inline-block;
    margin-right: 2px;
    font-size: 12px;
    .iconclass-li {
      display: inline-block;
      margin: 0 5px;
      color: #d0d4db;
      font-size: 12px;
    }
    .nav-item{
      display: flex;
      align-items: center;
      line-height: 1;
      .name{
        // font-size: 14px;
        line-height: 14px;
        padding-top: 2px;
      }
      .arror{
        font-size: 14px;
        line-height: 14px;
        padding-left: 3px;
        // padding: -10px 0 0 2px;
      }
    }
  }
}

.content-item {
  display: flex;
  position: relative;
  align-items: center;
}

.box_sizing {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

.btn {
  position: fixed;
  bottom: 0;
  padding: 10px 10px;
  background-color: #fff;
  width: 100%;
  text-align: right;
  .sureBtn {
    margin-right: 10px;
    &:hover{
        color: #CD170A !important;
        border: 1px solid #CD170A !important;
        opacity: 0.8 !important;
        // border: none !important;
      }
    &.red{
      background-color: #CD170A !important;
      border: 1px solid #CD170A !important;
      color: #fff !important;
      &:hover{
        color: #fff !important;
        opacity: 0.8 !important;
        // border: none !important;
      }
    }
    
  }
}
.main-pannel {
  display: flex;
  .main-wrapper {
    width: 50%;
    // overflow-y: auto
  }
  .main-selected {
    width: 50%;
    padding: 10px;
    .pannel-model {
      background: #fff;
      padding: 10px;
      .select-title {
        font-size: 14px;
      }
      .select-ul {
        margin-top: 10px;
        padding: 0px 0px;
        .select-li {
          padding: 8px 10px 8px 0px;
          display: flex;
          align-items: center;
          justify-content: space-between;
          border-bottom: 1px solid #ededed;
          transition: all .5s;
          &:hover{
            background: #cd170a10;
          }
          &:nth-last-child(1) {
            border-bottom: none;
          }
          .user-msg{
            display: flex;
            align-items: center;
            justify-content: flex-start;
            .name {
              font-size: 14px;
              margin-left: 8px;
            }
          }
          
          .close {
            position: relative;
            width: 13px;
            height: 13px;
            &::before,
            &::after {
              position: absolute;
              content: " ";
              background-color: #333;
              left: 6px;
              width: 1px;
              height: 13px;
            }
            &::before {
              transform: rotate(45deg);
            }
            &::after {
              transform: rotate(-45deg);
            }
            &:hover{
              &::after,&::before{
                background-color: #CD170A;
              }
            }
          }
        }
      }
    }
  }
}
.head{
  width: 30px;
  height: 30px;
  margin-left: 8px;
  .user{
    width: 100%;
    height: 100%;
    font-size: 10px;
    border-radius: 50%;
    background: #0095f2;
    text-align: center;
    color: #fff;
    line-height: 30px;
  }
  .group{
    width: 100%;
    height: 100%;
    background: url('~@/assets/images/icon/icon-group.png') no-repeat left;
    background-size: 100% 100%;
  }
}
.scroll-wrapper {
  width: 100%;
}
.inline-item {
  display: inline-block;
  margin-right: 10px;
  font-size: 12px;
}

.iconclass {
  font-size: 12px;
}
.checkbox {
  width: 18px;
  height: 18px;
}
.iconfont {
  &.icon-no-select {
    background: url("~@/assets/images/icon/icon-multiple-no-select.png") no-repeat center;
    background-size: 100% 100%;
  }
  &.icon-selects {
    background: url("~@/assets/images/icon/icon-multiple-select.png") no-repeat center;
    background-size: 100% 100%;
  }
  &.icon-not-select-all{
    background: url("~@/assets/images/icon/icon-multiple-not-all.png") no-repeat center;
    background-size: 100% 100%;
  }
}
</style>
