<template>
    <header>
      <!--data-toggle="modal" data-target="#ModalLogin"-->
      <div class="header-body">
        <div class="header-body-button">
          <label v-if="!Index"><a @click="showLogin()">登录 </a>|<a @click="showRegister()"> 注册</a></label>
          <label v-if="Index"><a >{{userName}} </a> | <a @click="getBaseInfo()"> 个人中心</a> | <a @click="LoginOut()">注销</a></label>
        </div>
       <div class="header-title" @click="BackHome()">云南民族大学中印瑜伽学院瑜伽非学历教育综合信息平台</div>
        <div class="container header-search">
          <div class="row">
            <div class="col-md-4 col-md-offset-4">
              <div class="input-group">
                <input type="text" class="form-control header-search-input" v-model="idCode" placeholder="请输入18位身份证号码">
                <span class="input-group-btn">
                  <button class="btn btn-primary header-search-btn" @click="getCertList(idCode)" type="button"><span class="glyphicon glyphicon-search"></span></button>
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <Modal v-model="modalLogin" title="用户登录" footer-hide>
        <Form ref="LresquesInfo" :model="LresquesInfo" :rules="ruleCustoms" :label-width="80">
          <FormItem label="用户名" prop="userName">
            <Input v-model="LresquesInfo.userName" placeholder="请输入11位手机号码"></Input>
          </FormItem>
          <FormItem label="密码" prop="passwds">
            <Input type="password" v-model="LresquesInfo.loginKey" placeholder="请输入由数字、字母、下划线组合的不少于8位不大于20位的密码"></Input>
          </FormItem>
          <FormItem label="验证码" prop="VCodes" >
            <Input class="ivu-inputs" v-model="LresquesInfo.imageCode" type="text" placeholder="请输入右侧图片验证码"></Input>
            <img :src="imageData[0]" @click="getImageData" height="34">
          </FormItem>
          <FormItem>
            <Button type="primary" @click="SubmitLogin('LresquesInfo')" style="margin: 20px 8px 0 0px; width: 350px;">提交</Button>
          </FormItem>
        </Form>
      </Modal>

      <Modal v-model="modalRegister" title="用户注册" footer-hide>
        <Form ref="RresquesInfo" :model="RresquesInfo" :rules="ruleCustom" :label-width="80">
          <FormItem label="手机号" prop="Phone">
            <Input v-model="RresquesInfo.userName" placeholder="请输入11位手机号码"></Input>
          </FormItem>
          <FormItem label="密码" prop="passwd">
            <Input type="password" v-model="RresquesInfo.loginKey" placeholder="请输入由数字、字母、下划线组合的不少于8位不大于20位的密码"></Input>
          </FormItem>
          <FormItem label="确认密码" prop="passwdCheck">
            <Input type="password" v-model="RresquesInfo.loginKey2" placeholder="请保证两次输入的密码一致"></Input>
          </FormItem>
          <FormItem label="验证码" prop="VCode" >
            <Input class="ivu-inputs" v-model="RresquesInfo.imageCode" type="text" placeholder="请输入右侧图片验证码"></Input>
            <img :src="imageData[0]" @click="getImageData" height="34">
          </FormItem>
          <FormItem label="短信验证码" prop="MCode">
            <Input class="ivu-inputs" v-model="RresquesInfo.messageCode" placeholder="请输入短信验证码"></Input>
            <Button type="success" @click="getRegisterCode()" :disabled="IsSend">{{value}}</Button>
          </FormItem>
          <FormItem>
            <Button type="primary" @click="SubmitRegister('RresquesInfo')" style="margin: 20px 8px 0 0; width: 350px;">提交</Button>
          </FormItem>
        </Form>
      </Modal>

      <!--<LoginModal  :resquesInfo="resquesInfo" :imageData="imageData" :SubmitLogin="SubmitLogin" :getImageData="getImageData" :showRegister="showRegister" :getRegisterCode="getRegisterCode"  :SubmitRegister="SubmitRegister" :value="value"></LoginModal>-->
    </header>
</template>

<script>
  import LoginModal from '../components/LoginModal';
  import Vue from 'vue'

    export default {
      name: "Header",
      data: function () {
        //--- 用户名验证 ---
        const validateuserName = (rule, value, callback) => {
          if (!this.checkPhone(this.LresquesInfo.userName)) {
            callback(new Error('请输入正确的11位手机号'));
          } else {
            callback();
          }
        };
        // 密码验证
        const validatepasswds = (rule, value, callback) => {
          if (!this.checkPhone(this.LresquesInfo.userName)) {
            callback(new Error('请输入正确的11位手机号'));
          } else {
            callback();
          }
        };
        // 验证码验证
        const validateVCodes = (rule, value, callback) => {
          if (!this.checkPhone(this.LresquesInfo.userName)) {
            callback(new Error('请输入正确验证码'));
          } else {
            callback();
          }
        };

        //--- 手机号验证 ---
        const validatePhone = (rule, value, callback) => {
          if (!this.checkPhone(this.RresquesInfo.userName)) {
            callback(new Error('请输入正确的11位手机号'));
          } else {
            callback();
          }
        };

        // 密码判断
        const validatePass = (rule, value, callback) => {
          if (!this.checkPassword(this.RresquesInfo.loginKey)) {
            callback(new Error('请输入由数字、字母、下划线组合的不少于8位不大于20位的密码'));
          } else {
            if (this.RresquesInfo.loginKey !== this.RresquesInfo.loginKey2) {
              // 对第二个密码框单独验证
              this.$refs.RresquesInfo.validateField('passwdCheck');
            } else {
              callback();
            }
          }
        };

        // 确认密码判断
        const validatePassCheck = (rule, value, callback) => {
          if (this.RresquesInfo.loginKey !== this.RresquesInfo.loginKey2) {
            callback(new Error('请保证两次输入的密码一致'));
          } else {
            callback();
          }
        };

        // 图片验证码
        const validateVCode = (rule, value, callback) => {
          if (!this.checkVCode(this.RresquesInfo.imageCode)) {
            return callback(new Error('请输入正确的4位验证码'));
          } else {
            callback();
          }
        };

        // 短信验证码
        const validateMCode = (rule, value, callback) => {
          if (!this.checkMCode(this.RresquesInfo.messageCode)) {
            return callback(new Error('请输入正确的4位短信验证码'));
          } else {
            callback();
          }
        };

        return{
          modalLogin: false,
          modalRegister: false,
          loading: true,

          ruleCustoms: {
            userName: [
              { validator: validateuserName, trigger: 'blur' }
            ],
            passwds: [
              { validator: validatepasswds, trigger: 'blur' }
            ],
            VCodes: [
              { validator: validateVCodes, trigger: 'blur' }
            ],
          },

          ruleCustom: {
            Phone: [
              { validator: validatePhone, trigger: 'blur' }
            ],
            passwd: [
              { validator: validatePass, trigger: 'blur' }
            ],
            passwdCheck: [
              { validator: validatePassCheck, trigger: 'blur' }
            ],
            VCode: [
              { validator: validateVCode, trigger: 'blur' }
            ],
            MCode: [
              { validator: validateMCode, trigger: 'blur' }
            ]
          },

          Index: false,
          IsSend: false,
          time: 0,

          userName: '',
          idCode: '',
          value: '获取验证码',
          imageData: ['0', '1'],

          // 注册参数
          LresquesInfo: {
            userName: '',
            loginKey: '',
            imageCode: '',
            imageKey: '',
          },

          // 注册参数
          RresquesInfo: {
            userName: '',
            loginKey: '',
            loginKey2: '',
            imageCode: '',
            imageKey: '',
            messageCode: '',
          },
        }
      },



      methods:{

        handleReset (name) {
          this.$refs[name].resetFields();
        },
        showLogin(){
          this.modalLogin = true;
          this.getImageData();
        },
        showRegister(){
          this.modalRegister = true;
          this.getImageData();
        },
        validateField(e){
          console.log(e)
        },

        // 跳转个人中心
        getBaseInfo: function () {
          var that = this;
          that.$router.push({
            name: 'PersonCenter',
          })
        },

        BackHome: function () {
          var that = this;
          that.$router.push({
            name: 'Home',
          })
        },

        // 查询证书
        getCertList: function (e) {
          // console.log(e);
          var that = this;
          that.$router.push({
            name: 'CertList',
            idCode: e,
            params: {
            },
          })
        },

        // 获取图片验证码
        getImageData: function () {
          const that = this;
          var ResquestInfo = new URLSearchParams();
          that.VuegetResquest(that.GLOBALS.LOGIN_GETIMAGECODE,ResquestInfo,function(res){
            Vue.set(that.imageData, 0 , res.data.imageData);
            that.LresquesInfo.imageKey = res.data.imageKey;
            that.RresquesInfo.imageKey = res.data.imageKey;
          },function (res) { this.$Message.warning('请输入正确的手机号和验证码');});
        },

        // 获取注册短信验证码
        getRegisterCode: function () {
          const that = this;
          var ResquestInfo = new URLSearchParams();
          ResquestInfo.append("phone",that.RresquesInfo.userName);
          ResquestInfo.append("imageCode", that.RresquesInfo.imageCode );
          ResquestInfo.append("imageKey",that.RresquesInfo.imageKey);
          that.VuegetResquest(that.GLOBALS.LOGIN_USERREGISTERSENDMESSAGE,ResquestInfo,function(res){
            that.$Message.success('已发送短信验证码');

            that.run();
            that.IsSend = true;
          },function (res) {that.$Message.warning('请输入正确的手机号和验证码'); });
        },
        run: function() {
          this.time = 60;
          this.timer();
        },
        stop: function(){
          this.time = 0;
          this.IsSend = false;
        },
        timer: function() {
          if (this.time > 0) {
            this.time--;
            this.value = "已发送(" + this.time + ')';
            setTimeout(this.timer, 1000);
          }
        },

        // 登录
        SubmitLogin: function (e) {
          const that = this;
          var ResquestInfo = new URLSearchParams();
          ResquestInfo.append("userName",that.LresquesInfo.userName);
          ResquestInfo.append("loginKey",that.LresquesInfo.loginKey);
          ResquestInfo.append("imageCode",e);
          ResquestInfo.append("imageKey",that.LresquesInfo.imageKey);
          console.log(ResquestInfo);
          that.VuegetResquest(that.GLOBALS.LOGIN_USERLOGIN,ResquestInfo,function(res){
            console.log(res);
            if (res.code === 10000) {
              console.log(res);
              window.location.reload();
              window.localStorage.setItem('Token', res.data.token);
              window.localStorage.setItem('userName', res.data.userName);
              $('#ModalLogin').modal('hide');
              that.$router.push({
                name: 'BaseInfo',
              })
            }
          },function (res) {
            that.$Message.error(res.message);
            // alert(res.message);
            that.getImageData();
            // console.log(res.message)
          });
        },

        // 注册
        SubmitRegister: function (name) {
          this.$refs[name].validate((valid) => {
            if (valid) {

              const that = this;
              var ResquestInfo = new URLSearchParams();
              ResquestInfo.append("phone", that.RresquesInfo.userName);
              ResquestInfo.append("loginKey", that.RresquesInfo.loginKey);
              ResquestInfo.append("phoneCode", e);

              that.VuegetResquest(that.GLOBALS.LOGIN_USERREGISTER, ResquestInfo, function (res) {
                console.log(res);
                that.$Message.success(注册成功);
                if (res.code === 10000) {
                  this.modalRegister = true;
                }
              }, function (res) {
                that.$Message.warning(res.message);
              },function (res) {that.$Message.warning(res.message);});
              // that.value = '获取验证码'
            } else {
              // this.$Message.error('Fail!');
            }
          })

        },

        // 注销
        LoginOut: function () {
          window.localStorage.setItem('Token', -1);
          window.localStorage.setItem('userName', '未登录');
          this.Index = false;
          this.userName = '未登录';
          this.BackHome();
        }
      },

      components: {
        LoginModal,
      },

      created: function () {
        // console.log(this.resquesInfo.userName);
        let userName = window.localStorage.getItem('userName');
        if(userName == null || userName === '未登录') {
          this.Index = false;
          this.userName = '未登录'
        }else {
          this.Index = true;
          this.userName = window.localStorage.getItem('userName')
        }
      }
    }

</script>

<style scoped>
  .ivu-inputs {
    width: 220px;
    margin-right: 60px;
  }

  .form {
    padding: 20px;
  }
  .form input {
    padding-left: 20px;
  }
  .form-group {
    line-height: 2.5em;
  }
  .close {
    position: fixed;
    right: 0;
    margin-right: 26px;
    font-size: 40px;
    padding: 0;
  }
  .btn{
    line-height: 2em;
  }
  .header-body {
    user-select: none;
    background-image: url("../assets/img/bg.jpg");
    background-repeat:no-repeat;
    background-size:100% 100%;
    -moz-background-size:100% 100%;
    position:relative;
    height: 298px;
    padding-top: 50px;
  }
  .header-body-button {
    line-height: 50px;
    right: 40px;
    position: absolute;
    float: right;
    color: #fff;
    font-size: 18px;
    margin: -40px 0 0 0 ;
  }
  .header-body-button a {
    color: #fff;
    font-weight: 400;
    text-decoration:none
    }
  .header-body-button a:hover{
    color: #1296db;
  }
  .header-title {
    text-align: center;
    font-size: 36px;
    color: #ffffff;
  }
  .header-search {
    padding-top: 50px;
  }
  .header-search-input {
    height: 40px;
    padding: 5px 10px;
    border-radius: 0;
  }
  .header-search-btn {
    height: 40px;
    border-radius: 0;
  }
  .control-label{
    padding: 0px;
  }
</style>
