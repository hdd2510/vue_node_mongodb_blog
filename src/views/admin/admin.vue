<template lang="html">
  <div class="admin">
    <section class="top">
      <router-link to="/admin">
        <img class="img" src="../../assets/logo.png" alt="">
        <p class="top-left">欢迎{{userNews.username}}来到博客后台</p>
      </router-link>
     <!--  <p class="top-right" @click="logout">退出</p>
      <router-link to="/"><p class="top-right">返回前台首页</p></router-link> -->

  <!-- 个人中心 -->

    <el-dropdown @command="handleCommand" class="top-right">
      <span class="el-dropdown-link">
        个人中心<i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <el-dropdown-menu slot="dropdown">
        <router-link to="/"><el-dropdown-item command="欢迎来到前台">let's go前台</el-dropdown-item></router-link>
        <el-dropdown-item divided command="用户资料">修改资料</el-dropdown-item>
        <el-dropdown-item divided command="退出后台">退出</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>

    </section>
    <section class="list">
      <div class="list-left">
        <el-menu 
          background-color="#1f2d3d"
          text-color="#fff"
          active-text-color="#ffd04b" default-active="2" class="el-menu-vertical-demo" @open="handleOpen" @close="handleClose">
          <el-submenu index="1">
            <template slot="title"><i class="el-icon-setting"></i>用户管理</template>
            <el-menu-item-group>
              <router-link to="/admin_users"><el-menu-item index="1-1"><i class="el-icon-menu"></i>用户列表</el-menu-item></router-link>
            </el-menu-item-group>
          </el-submenu>
          <el-submenu index="2">
            <template slot="title"><i class="el-icon-setting"></i>分类管理</template>
            <el-menu-item-group>
              <router-link to="/admin_category"><el-menu-item index="2-1"><i class="el-icon-menu"></i>分类列表</el-menu-item></router-link>
              <router-link to="/admin_category_add"><el-menu-item index="2-2"><i class="el-icon-plus"></i>分类增加</el-menu-item></router-link>
             <!--  <router-link to="/admin_category_update"><el-menu-item index="2-3"><i class="el-icon-edit"></i>分类修改</el-menu-item></router-link> -->
            </el-menu-item-group>
          </el-submenu>
          <el-submenu index="3">
            <template slot="title"><i class="el-icon-setting"></i>文章管理</template>
            <el-menu-item-group>
              <router-link to="/admin_article"><el-menu-item index="3-1"><i class="el-icon-menu"></i>文章列表</el-menu-item></router-link>
              <router-link to="/admin_article_add"><el-menu-item index="3-2"><i class="el-icon-plus"></i>文章增加</el-menu-item></router-link>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </div>
      <div class="list-right">
        <router-view :userNews="userNews">
        </router-view>
      </div>
    </section>

    <el-dialog title="本人资料" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.name" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="旧密码" :label-width="formLabelWidth">
          <el-input v-model="form.oldPassword" auto-complete="off" disabled></el-input>
        </el-form-item>

        <el-form-item label="新密码" :label-width="formLabelWidth">
          <el-input v-model="form.newPassword" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="确认密码" :label-width="formLabelWidth">
          <el-input v-model="form.ComfirmPassword" auto-complete="off"></el-input>
        </el-form-item>
        
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="changeInformation">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    data() {
      return {
        dialogFormVisible: false,
        userNews : '',
        form: {
          name: '',
          password : '',
          ComfirmPassword : ''
        },
        formLabelWidth: '120px'
      }
    },
    created(){
      this.$http.get('/users/checkLogin').then((respond) => {
        var that = this;
        if(respond.data.code == 200 ){
          this.userNews = respond.data.message
          console.log(respond.data.message)
        }
      })

    },
    methods: {
      handleCommand(command) {
        if(command=="退出后台"){
          this.$http.get('/users/logout').then((respond) => {
            var that = this;
            if(respond.data.code == 200 ){
              this.userCookie = !this.userCookie;
              this.$message({
                message: '退出后台管理平台，返回前台首页',
                type: 'success'
              });
              setTimeout(function(){
                that.$router.push({path: '/'});
              },1000)
            }
          })
        }else if(command=="用户资料"){
            this.dialogFormVisible = true;
            this.$http.get('/admin/admin_information?_id=' + this.userNews._id).then((respond) => {
            var that = this;
            if(respond.data.code == 200 ){
              console.log(respond.data.data);
              this.form.name = respond.data.data.username;
              this.form.oldPassword = respond.data.data.password;

            }
          })
        }else{
          this.$message({
            message : command,
            type : 'success'
          });
        }

     
      },
      handleOpen(key, keyPath) {
        console.log(key, keyPath);
      },
      handleClose(key, keyPath) {
        console.log(key, keyPath);
      },
      changeInformation(e){
          if(this.form.name == ''){
            this.$message({
              message : "用户名不可以为空",
              type : 'error'
            });
            return;
          }
          if(this.form.newPassword == ''){
            this.$message({
              message : "密码不可以为空",
              type : 'error'
            });
            return;
          }
          if(this.form.newPassword !== this.form.ComfirmPassword){
            this.$message({
              message : "两次输入的密码不一样",
              type : 'error'
            });
            return;
          }
          this.$http.post('/admin/admin_change_information',{
            _id : this.userNews._id,
            username: this.form.name,
            newPassword: this.form.newPassword,
            ComfirmPassword: this.form.ComfirmPassword
          }).then((res => {
            if(res.data.code == 1){
              this.$message({
              message : res.data.message,
              type : 'error'
            }); 
              return;
            }else{
              this.$message({
                message : '修改成功',
                type : 'success'
              });
              this.dialogFormVisible = false
            }
          }))
      }
    }
  }
</script>

<style lang="less" scoped>
  .admin{
    background-color: #f1f1f1;
    .top {
      position: fixed;
      top: 0;
      left: 0;
      height: 80px;
      width: 100%;
      overflow: hidden;
      background-color: #1f2d3d;
      .img{
        float: left;
        height: 60px;
        width: 60px;
        margin: 10px 30px;
      }
      .top-left{
        float: left;
        margin-right: 20px;
        line-height: 80px;
        color: #fff;
        .el-menu {
          border-right: solid 0px #e6e6e6;
        }
      }
      .top-right{
        float: right;
        margin-right: 20px;
        margin-top : 30px;
        line-height: 20px;
        color: #fff;
        cursor: pointer;
      }
      .el-dropdown-link {
        cursor: pointer;
        color: #409EFF;
      }
      .el-icon-arrow-down {
        font-size: 12px;
      }
    }
    .list{
      margin-top: 80px;
      display: flex;
      .list-left{
        position: fixed;
        top: 80px;
        left: 0;
        flex: 0 0 250px;
        width: 250px;
        height: 100%;
        background-color: #324157;
      }
      .list-right{
        flex: 1;
        margin-left: 250px;
        padding: 20px;
        background-color: #fff;
      }
    }
  }
</style>
