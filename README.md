## 起步

```bash
# 克隆项目
git clone https://github.com/PanJiaChen/vue-element-admin.git

# 进入项目目录
cd vue-element-admin

# 安装依赖
npm install

# 启动服务
npm run dev
```

## 发布

```bash
# 构建生产环境
npm run build
```

### 食用方法

```vue
<template>
	<tree-table ref='treeTable' :data="data">
        <el-table-column label="Title">
          <slot name="header"></slot>
          <template slot-scope="scope">
            {{scope.row.name}}
          </template>
        </el-table-column>
      </tree-table>
</template>

<script>
	{
      data() {
        return {
          data:[],
        }
      },
      mounted(){
        this.$refs.treeTable.render(dataJson)
      }
    };
</script>
```

