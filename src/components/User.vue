<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--card卡片区域-->
    <el-card class="box-card">
      <!--搜索框 和 添加用户按钮-->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="querycdt.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="7">
          <el-button type="primary">添加用户</el-button>
        </el-col>
      </el-row>

      <!--表格展示数据列表-->
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="110"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="120"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="160"></el-table-column>
        <el-table-column prop="mg_state" label="状态" width="70">
          <!--插槽填充赶脚
              在此位置要获取“每个”用户的信息，具体是用户的状态信息
              用户的信息已经被子组件(el-table-column)的插槽传递过来了
              <slot row="每个用户的数据对象"></slot>
              因此可以通过如下方式获得当前遍历好的每个用户信息
              <span slot-scope="info">{{info.row}}</span>
              也可以通过如下方式获得当前遍历好的每个用户的具体信息
              <span slot-scope="info">{{info.row.xxxxx}}</span>
          -->
          <!-- <span slot-scope="info">{{info.row.username}}</span> -->
          <el-switch
            v-model="info.row.mg_state"
            slot-scope="info"
            @change="changeState(info.row.id, info.row.mg_state)"
          ></el-switch>
        </el-table-column>
        <el-table-column label="操作" width="230">
          <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini"></el-button>
          <el-tooltip content="分配角色" placement="top" enterable="false">
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
          </el-tooltip>
        </el-table-column>
      </el-table>

      <!--数据分页展示-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querycdt.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="querycdt.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="querycdt.tot"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  // 生命周期函数
  created() {
    this.getUserInfos()
  },
  methods: {
    // 修改用户状态的方法
    // uid：被修改状态用户的id值
    // state: 被修改后的状态信息true/false
    async changeState(uid, state) {
      // console.log(uid, state)    //500 false
      const {data:res} = await this.$http.put('users/'+uid+'/state/'+state)

      if(res.meta.status!==200){return this.$message.error(res.meta.msg)}

      // 提示修改状态成功的信息
      this.$message.success(res.meta.msg)
    },

    // 数据分页相关1
    // 每条记录条数变化的回调处理
    handleSizeChange(arg) {
      // arg: 变化后的记录条数
      // console.log(arg)
      this.querycdt.pagesize = arg
      // 重新根据条件获得数据
      this.getUserInfos()
    },
    // 当前页码变化的回调处理
    handleCurrentChange(arg) {
      // arg: 变化后的当前页码值
      // console.log(arg)
      this.querycdt.pagenum = arg
      // 根据变化后的页码重新获得数据
      this.getUserInfos()
    },
    // 数据分页相关2

    // 获得用于显示的真实用户列表数据
    async getUserInfos() {
      // this.$http.get('users', 条数/页码/关键字)
      // 以下axios发送请求的url格式为：http://主机名：端口/xxx/users?query=&pagenum=1&pagesize=3
      const { data: res } = await this.$http.get('users', {
        params: this.querycdt
      })
      // console.log(res)

      // 判断获取数据是否失败
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 记录总记录条数
      this.querycdt.tot = res.data.total
      // 把获得好的用户数据 赋予 给userInfos成员
      this.userInfos = res.data.users
    }
  },
  data() {
    return {
      // 用户数据
      userInfos: [],

      // 给获取用户数据设置查询条件
      querycdt: {
        // 查询关键字
        query: '',
        // 当前页码
        pagenum: 1,
        // 每页获取记录条数
        pagesize: 3,
        // 总记录条数
        tot: 0
      }

      // table表格展示的demo数据，结构是“数组对象集”
      // tableData: [
      //   {
      //     date: '2016-05-02',
      //     name: '王小虎',
      //     address: '上海市普陀区金沙江路 1518 弄'
      //   },
      //   {
      //     date: '2016-05-04',
      //     name: '王小虎',
      //     address: '上海市普陀区金沙江路 1517 弄'
      //   },
      //   {
      //     date: '2016-05-01',
      //     name: '王小虎',
      //     address: '上海市普陀区金沙江路 1519 弄'
      //   },
      //   {
      //     date: '2016-05-03',
      //     name: '王小虎',
      //     address: '上海市普陀区金沙江路 1516 弄'
      //   }
      // ]
    }
  }
}
</script>

<style lang="less" scoped>
</style>
