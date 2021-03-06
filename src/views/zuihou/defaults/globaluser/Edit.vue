<template>
  <el-dialog :close-on-click-modal="false" :close-on-press-escape="false" :title="title" :visible.sync="isVisible" :width="width" top="50px">
    <el-form ref="form" :model="globalUser" :rules="rules" label-position="right" label-width="130px">
      <el-form-item :label="$t(&quot;table.globalUser.tenantCode&quot;)" prop="tenantCode">
        <el-select v-model="globalUser.tenantCode" :disabled="type === &quot;edit&quot;" placeholder="企业">
          <el-option key="1" label="超管" value="admin" />
          <el-option v-for="item in tenantList" :key="item.code" :label="item.name" :value="item.code" />
        </el-select>
      </el-form-item>
      <el-form-item :label="$t('table.globalUser.account')" prop="account">
        <el-input v-model="globalUser.account" :disabled="type === &quot;edit&quot;" />
      </el-form-item>
      <el-form-item v-show="type === 'add'" :label="$t('table.globalUser.password')" prop="password">
        <el-input v-model="globalUser.password" maxlength="64" type="password" />
      </el-form-item>
      <el-form-item v-show="type === 'add'" :label="$t(&quot;table.globalUser.confirmPassword&quot;)" prop="confirmPassword">
        <el-input v-model="globalUser.confirmPassword" maxlength="64" type="password" />
      </el-form-item>
      <el-form-item :label="$t(&quot;table.globalUser.name&quot;)" prop="name">
        <el-input v-model="globalUser.name" maxlength="50" />
      </el-form-item>
      <el-form-item :label="$t(&quot;table.globalUser.mobile&quot;)" prop="mobile">
        <el-input v-model="globalUser.mobile" maxlength="20" />
      </el-form-item>
      <el-form-item :label="$t(&quot;table.globalUser.email&quot;)" prop="email">
        <el-input v-model="globalUser.email" maxlength="255" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button plain type="warning" @click="isVisible = false">{{ $t('common.cancel') }}</el-button>
      <el-button plain type="primary" @click="submitForm">{{ $t('common.confirm') }}</el-button>
    </div>
  </el-dialog>
</template>
<script>
import globalUserApi from '@/api/GlobalUser.js'
import tenantApi from '@/api/Tenant.js'

export default {
  name: 'GlobalUserEdit',
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      tenantList: [],
      type: 'add',
      globalUser: this.initGlobalUser(),
      screenWidth: 0,
      width: this.initWidth(),
      rules: {
        account: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 4, max: 30, message: this.$t('rules.range4to10'), trigger: 'blur' },
          {
            validator: (rule, value, callback) => {
              if (!this.globalUser.id && this.globalUser.tenantCode) {
                const checkData = {
                  tenantCode: this.globalUser.tenantCode,
                  account: value
                }
                globalUserApi.check(checkData)
                  .then((response) => {
                    const res = response.data
                    if (res.data) {
                      callback(this.$t('rules.accountExist'))
                    } else {
                      callback()
                    }
                  })
              } else {
                callback()
              }
            }, trigger: 'blur'
          }
        ],
        tenantCode: [{ required: true, message: this.$t('rules.require'), trigger: 'blur' }]
      }
    }
  },
  computed: {
    isVisible: {
      get () {
        return this.dialogVisible
      },
      set () {
        this.close()
        this.reset()
      }
    }
  },
  mounted () {
    this.initTenantList()
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  methods: {
    initTenantList () {
      tenantApi.list()
        .then((response) => {
          const res = response.data
          this.tenantList = res.data
        })
    },
    initGlobalUser () {
      return {
        id: '',
        tenantCode: 'admin',
        name: '',
        account: '',
        mobile: '',
        email: '',
        password: '',
        confirmPassword: ''
      }
    },
    initWidth () {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '45%'
      } else {
        return '800px'
      }
    },
    setGlobalUser (val) {
      this.globalUser = { ...val }
    },
    close () {
      this.$emit('close')
    },
    submitForm () {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.type === 'add') {
            this.save()
          } else {
            this.update()
          }
        } else {
          return false
        }
      })
    },
    save () {
      globalUserApi.save(this.globalUser)
        .then((response) => {
          const res = response.data
          if (res.isSuccess) {
            this.isVisible = false
            this.$message({
              message: this.$t('tips.createSuccess'),
              type: 'success'
            })
            this.$emit('success')
          }
        })
    },
    update () {
      globalUserApi.update(this.globalUser)
        .then((response) => {
          const res = response.data
          if (res.isSuccess) {
            this.isVisible = false
            this.$message({
              message: this.$t('tips.updateSuccess'),
              type: 'success'
            })
            this.$emit('success')
          }
        })
    },
    reset () {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.globalUser = this.initGlobalUser()
    }
  }
}
</script>
<style lang="scss" >
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100px;
  height: 100px;
  line-height: 100px;
  text-align: center;
}
.avatar {
  width: 100px;
  height: 100px;
  display: block;
}
</style>
