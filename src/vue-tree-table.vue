<template>
  <el-table
    :data="data"
    :row-style="showTr"
    v-bind="$attrs"
    @selection-change="handleSelectionChange"
    @row-click="handleRowClick"
    @header-click="handleHeaderClick"
    size="mini"
    ref="table"
  >
    <el-table-column v-if="selection" type="selection" :width="selectionWidth">
    </el-table-column>
    <slot name="left" />
    <el-table-column
      align="left"
      :label="nodeFieldLabel || nodeField || 'ID'"
      min-width="95"
    >
      <template slot-scope="scope">
        <span
          :style="{
            paddingLeft: judegeLevel(scope.row) * levelPadding + 'px',
            cursor: showExpanIcon(scope.row) ? 'pointer' : ''
          }"
          @click.stop="toggleExpand(scope.$index)"
          >{{ scope.row[nodeField] || scope.row.id }}</span
        >
        <el-button
          class="expand-button"
          type="text"
          size="mini"
          v-if="showExpanIcon(scope.row)"
          @click.stop="toggleExpand(scope.$index)"
        >
          <i
            :class="scope.row._isExpanded ? collapseIcon : expandIcon"
            class="f-fwb"
          ></i>
        </el-button>
      </template>
    </el-table-column>
    <slot />
  </el-table>
</template>
<script>
// 兼容性处理
// import { version } from "element-ui";
const semverGte = require('semver/functions/gte')
const semverLte = require('semver/functions/lte')
let version = ''
if (typeof window !== 'undefined' && window.ELEMENT) {
  version  = window.ELEMENT.version
}
/* process.env.NODE_ENV !== "production" &&
  semverGte(version,"2.7.0") &&
  semverLte(version , "2.12.0")
  console.warn(
    "element-ui版本在2.7.0-2.12.0中,tablebody.vue 指令 v-show 会覆盖 row-style 中 display,因此用z-index进行隐藏.若要更好的渲染效率,请把element-ui升级到2.13.0及以上."
  ); */
const trHidden =
  semverGte(version,"2.7.0") && semverLte(version , "2.12.0")
    ? {
        position: "absolute",
        "z-index": -999
      }
    : { display: "none" };
export default {
  props: {
    /* data: {
      type: Array,
      default: () => []
    }, */
    nodeField: {
      type: String,
      default: ""
    },
    nodeFieldLabel: {
      type: String,
      default: ""
    },
    defaultExpandAll: {
      type: Boolean,
      default: false
    },
    defaultChildren: {
      type: String,
      default: "children"
    },
    levelPadding: {
      type: Number,
      default: 20
    },
    expandAnimation: {
      type: Object,
      default: () => {
        return {
          animation: "treeTableShow 1s"
        };
      }
    },
    expandIcon: {
      type: String,
      default: "el-icon-plus"
    },
    collapseIcon: {
      type: String,
      default: "el-icon-minus"
    },
    selection: {
      type: Boolean,
      default: false
    },
    selectionWidth: {
      type: Number,
      default: 55
    }
  },
  data() {
    return {
      listLoading: false,
      data: []
    };
  },
  created() {
  },
  methods: {
    render(dataSource){
      this.data = this.formatData(dataSource);
    },
    formatData(data, parentId) {
      let tempData = [];
      const defaultExpandAll = this.defaultExpandAll;
      data.forEach((item, index) => {
        if (parentId !== undefined) {
          this.$set(item, "_isExpanded", defaultExpandAll);
          this.$set(item, "_isShow", defaultExpandAll);
          this.$set(item, "_parentId", parentId + "");
          this.$set(item, "_id", parentId + "-" + index);
        } else {
          this.$set(item, "_isExpanded", defaultExpandAll);
          this.$set(item, "_isShow", true);
          this.$set(item, "_id", index + "");
        }
        if (
          item[this.defaultChildren] &&
          item[this.defaultChildren].length > 0
        ) {
          tempData.push(item);
          tempData = tempData.concat(
            this.formatData(
              item[this.defaultChildren],
              parentId ? parentId + "-" + index : index + ""
            )
          );
        } else {
          tempData.push(item);
        }
      });
      return tempData;
    },
    showTr: function({ row, index }) {
      const show = row._parentId === undefined || row._isShow;
      return show ? this.expandAnimation : trHidden;
    },
    toggleExpand: function(trIndex) {
      const record = this.data[trIndex];
      record._isExpanded = !record._isExpanded;
      const arr = this.data.filter(i => {
        return i._parentId === record._id;
      });
      arr.forEach(i => {
        i._isShow = !i._isShow;
      });
      // beta代码
      record._isExpanded
        ? this.showExpandedChildren(arr)
        : this.hiddenChildren(arr);
      this.$nextTick(() => {
        this.$refs.table.doLayout();
      });
    },
    // beta
    hiddenChildren: function(childrens) {
      childrens.map(children => {
        children._isShow = false;
        // children._isExpanded = false;
        children.operator ? "" : (children._isExpanded = false);
        children.operator ? this.hiddenChildren(children.operator) : "";
      });
    },
    // beta
    showExpandedChildren: function(childrens, parent) {
      childrens.map(children => {
        if (parent && parent.operator && parent._isExpanded && parent._isShow) {
          children._isShow = true;
          childrens._isExpanded = true;
        }
        children.operator
          ? this.showExpandedChildren(children.operator, children)
          : "";
      });
    },
    showExpanIcon(record) {
      if (
        record[this.defaultChildren] &&
        record[this.defaultChildren].length > 0
      ) {
        return true;
      }
      return false;
    },
    judegeLevel(row) {
      return (row._id + "").split("-").length - 1;
    },
    handleSelectionChange(val) {
      this.$emit("selection-change", val);
    },
    handleRowClick(val, column, event) {
      this.$emit("row-click", val);
    },
    handleHeaderClick(val) {
      this.$emit("header-click", val);
    }
  }
};
</script>

<style>
@keyframes treeTableShow {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>

<style scoped>
/* .expand-button {
  margin-left: -20px;
} */
</style>
