<template>
  <div class="message">
    <el-row class="message-msg">
      <el-col :span="24" style="color: black;">
        <h1>个人中心</h1>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="20" :offset="2" style="margin-top:10px;">
        <el-card class="box-card">
          <el-form ref="form" :rules="rules" :label-position="labelPosition" :model="userInfo" label-width="100px" style="width: 90%;">
            <el-row>
              <el-col :span="12">
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="账号：" prop="username">
                      <el-input :disabled="disabled" v-model="userInfo.username"/>
                    </el-form-item>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="姓名：" prop="name">
                      <el-input v-model="userInfo.name" placeholder="请输入姓名"/>
                    </el-form-item>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="性别" prop="sex">
                      <el-radio-group v-model="userInfo.sex">
                        <el-radio :label="0">男</el-radio>
                        <el-radio :label="1">女</el-radio>
                      </el-radio-group>
                    </el-form-item>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="出生日期" prop="born">
                      <el-date-picker v-model="userInfo.born" type="date" format="yyyy 年 MM 月 dd 日" value-format="yyyy-MM-dd" placeholder="出生日期"/>
                    </el-form-item>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="电话号码" prop="phone">
                      <el-input v-model="userInfo.phone" placeholder="电话号码"/>
                    </el-form-item>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-form-item label="邮箱" prop="email">
                      <el-input v-model="userInfo.email" placeholder="邮箱"/>
                    </el-form-item>
                  </el-col>
                </el-row>
              </el-col>
              <el-col :span="10" :offset="2">
                <el-row>
                  <el-col :span="12" :offset="6" style="text-align: center">
                    <el-upload
                      :show-file-list="false"
                      :on-success="handleAvatarSuccess"
                      :before-upload="beforeAvatarUpload"
                      :action="sysConfig.uploadUrl"
                      :headers="headers"
                      :data="params"
                      class="avatar-uploader">
                      <img v-if="userInfo.avatarUrl" :src="userInfo.avatarUrl" class="avatar">
                      <i v-else class="el-icon-plus avatar-uploader-icon"/>
                    </el-upload>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="12" :offset="6" style="text-align: center">
                    <h4>头像</h4>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <el-row>
              <el-col :span="8" :offset="8">
                <el-form-item>
                  <el-button type="primary" @click="update">保存</el-button>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import { updateObjInfo } from '@/api/admin/user'
import OHeader from '../common/header'
import OFooter from '../common/footer'
import { getToken } from '@/utils/auth'
import { mapState } from 'vuex'
import { delAttachment } from '@/api/admin/attachment'
import { getAttachmentPreviewUrl, isNotEmpty, notifySuccess, notifyFail } from '@/utils/util'
import store from '@/store'

export default {
  data () {
    return {
      labelPosition: 'right',
      disabled: true,
      rules: {
        name: [{ required: true, message: '请输入账号', trigger: 'change' }],
        username: [{ required: true, message: '请输入姓名', trigger: 'change' }]
      },
      headers: {
        Authorization: 'Bearer ' + getToken()
      },
      params: {
        busiType: '1'
      }
    }
  },
  components: {
    OHeader,
    OFooter
  },
  created () {
    this.userInfo.sex = parseInt(this.userInfo.sex)
  },
  computed: {
    ...mapState({
      userInfo: state => state.user.userInfo,
      sysConfig: state => state.sysConfig.sysConfig
    })
  },
  methods: {
    update () {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          updateObjInfo(this.userInfo).then(response => {
            if (response.data.data) {
              notifySuccess(this, '修改成功')
            } else {
              notifyFail(this, response.data.msg)
            }
          }).catch(() => {
            notifyFail(this, '修改失败')
          })
        }
      })
    },
    handleAvatarSuccess (res, file) {
      if (!isNotEmpty(res.data) || !isNotEmpty(res.data.fastFileId)) {
        notifyFail(this, '头像上传失败')
        return
      }
      this.userInfo.avatar = res.data.fastFileId
      this.userInfo.avatarUrl = getAttachmentPreviewUrl(this.sysConfig, this.userInfo.avatar)
      this.$refs['form'].validate((valid) => {
        if (valid) {
          if (isNotEmpty(this.userInfo.avatarId)) {
            // 删除旧头像
            delAttachment(this.userInfo.avatarId).then(() => {
              this.userInfo.avatarId = res.data.id
              // 更新头像信息
              updateObjInfo(this.userInfo).then(response => {
                if (isNotEmpty(response.data) && response.data.code === 200) {
                  notifySuccess(this, '头像上传成功')
                  // 更新localStorage
                  store.commit('SET_USER_INFO', this.userInfo)
                }
              }).catch((error) => {
                console.log(error)
                notifyFail(this, '头像上传失败')
              })
            }).catch((error) => {
              console.log(error)
              notifyFail(this, '头像上传失败')
            })
          } else {
            this.userInfo.avatarId = res.data.id
            // 更新头像信息
            updateObjInfo(this.userInfo).then(response => {
              if (isNotEmpty(response.data) && response.data.code === 200) {
                notifySuccess(this, '头像上传成功')
                // 更新localStorage
                store.commit('SET_USER_INFO', this.userInfo)
              }
            }).catch((error) => {
              console.log(error)
              notifyFail(this, '头像上传失败')
            })
          }
        }
      })
    },
    beforeAvatarUpload (file) {
      const isJPG = file.type === 'image/jpeg' || file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isJPG) {
        this.$message.error('上传头像图片只能是 jpg/png 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" rel="stylesheet/scss" scoped>
  .message{
    margin-bottom: 20px;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
</style>
