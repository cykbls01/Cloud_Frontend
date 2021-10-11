<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel"
    :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-title">
      <span>修改容器</span>
    </p>
    <div>
      <Row>
        <div>
        </div>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">容器名称</span></Col>
        <Col span="17"><Input v-model="podName" placeholder="只能包含大小写字母，数字，下划线和中划线" clearable style="width: 350px" :disabled="true" /></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">镜像名称</span></Col>
        <Col span="17"><Input v-model="image" clearable style="width: 350px" :disabled="true" /></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role === 'admin' || role === 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">使用者id</span></Col>
        <Col span="17"><Input v-model="userId" clearable style="width: 350px" :disabled="true"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <div v-for="(item,index) in listPort" :key = "item.index">
          <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">添加端口号</span></Col>
          <Col span="17"><Input v-model="item.value" clearable style="width: 250px" ></Input>
            <Button style="height:32px" @click="removeDomain(index)">删除</Button></Col>
        </div>
        <Button style="margin-left:160px" @click="addDomain">增加端口</Button>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <div v-for="(item,index) in listEnv" :key = "item.index">
          <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">添加环境变量</span></Col>
          <Col span="17"><Input v-model="item.label" placeholder="name" clearable style="width: 100px" ></Input>:
            <Input v-model="item.value" placeholder="value" clearable style="width: 100px" ></Input>
            <Button style="height:32px" @click="removeEnv(index)">删除</Button></Col>
        </div>
        <Button style="margin-left:160px" @click="addEnv">增加环境变量</Button>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role !== 'admin' && role !== 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span class="input-item">使用截止时间</span>
        </Col>
        <Col span="17">
          <DatePicker v-model="due_time" :options="dueTimeOption" type="datetime" placeholder="请选择使用截止时间" style="width: 350px"></DatePicker>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role !== 'admin' && role !== 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">申请原因</span></Col>
        <Col span="17"><Input v-model="applyReason" type="textarea" style="width:350px" :rows="10" placeholder="请简述申请理由及使用时间"/></Col>
      </Row>
    </div>
    <div slot="footer">
      <Button type="text" size="large" @click="cancel">取消</Button>
      <Button type="primary" size="large" :loading="posting" @click="ok">确定</Button>
    </div>
  </Modal>
</template>

<script>
import axios from 'axios'
import go from '../../../assets/Global'
export default {
  name: 'modifyContainer',
  props: {
    showModalCheckVM: {
      type: Boolean,
      default: false
    },
    podName: {
      type: String,
      default: ''
    },
    userId: {
      type: String,
      default: ''
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalCheckVM
      },
      set: function () {
      }
    },
    dueTimeOption: function () {
      return {
        disabledDate (date) {
          return date.valueOf() < Date.now() - 86400000
        }
      }
    }
  },
  data () {
    return {
      port: '',
      image: '',
      listTemplate: [],
      postUrl: go.gourl + '/applyContainer',
      inputVMName: '',
      applyReason: '',
      vmTemplate: '',
      due_time: '',
      posting: false,
      kanbukan: false,
      listPort: [],
      listEnv: [],
      role: ''
    }
  },
  watch: {
    podName (newval, oldval) {
      if (this.showModalCheckVM === false) {
        return
      }
      this.getListTemplate()
    }
  },
  mounted () {
    this.role = this.$cookie.get('role')
  },
  methods: {
    addDomain () {
      this.listPort.push({
        value: ''
      })
    },
    removeDomain (index) {
      if (index !== -1) {
        this.listPort.splice(index, 1)
      }
    },
    addEnv () {
      this.listEnv.push({
        value: '',
        label: ''
      })
    },
    removeEnv (index) {
      if (index !== -1) {
        this.listEnv.splice(index, 1)
      }
    },
    showNoticeWarning (noticeTitle, noticeDesc) {
      this.$Notice.warning({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    showNoticeSuccess (noticeTitle, noticeDesc) {
      this.$Notice.success({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    showNoticeError (noticeTitle, noticeDesc) {
      this.$Notice.error({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    getListTemplate () {
      this.listPort = []
      this.listEnv = []
      let params = {
        podName: this.podName,
        userId: this.userId
      }
      axios.post(go.gourl + '/getContainer', params,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          if (res.data.code === 1001) {
            console.log(res.data.data)
            let listp = res.data.data.port
            if (listp !== null) {
              for (let i = 0; i < listp.length; i++) {
                let a = {
                  value: listp[i]
                }
                this.listPort.push(a)
              }
            }
            this.listEnv = res.data.data.env
            if (this.listEnv === null) {
              this.listEnv = []
            }
            if (this.listPort === null) {
              this.listPort = []
            }
            this.image = res.data.data.image
          } else {
            this.showNoticeError('发送申请失败', '错误信息：' + res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    clearModal () {
      this.podName = ''
      this.image = ''
      this.listPort = []
      this.listEnv = []
      this.applyReason = ''
      this.vmTemplate = ''
    },
    judgeParams (params, type) {
      let flag = true
      if (params.name === '') {
        this.showNoticeWarning('容器名称错误', '容器名称不能为空')
        flag = false
      } else if ((params.end_time === '' || typeof (params.end_time) === 'undefined') && type === 2) {
        this.showNoticeWarning('使用截止时间错误', '请选择容器的使用截止时间')
        flag = false
      } else if (params.msg === '' && type === 2) {
        this.showNoticeWarning('申请原因错误', '申请原因不能为空')
        flag = false
      }
      return flag
    },
    ok () {
      let listp = []
      for (let i = 0; i < this.listPort.length; i++) {
        listp.push(this.listPort[i].value)
      }
      let type = 0
      if (this.role === 'student' || this.role === 'teacher') {
        type = 2
      } else if (this.role === 'admin' || this.role === 'superAdmin') {
        type = 4
      }
      let params = {
        name: this.podName,
        podname: this.podName,
        port: listp,
        env: this.listEnv,
        image: this.image,
        userId: this.userId,
        msg: this.applyReason,
        end_time: this.transferDatetimeToStr(this.due_time),
        type: type
      }
      if (!this.judgeParams(params, type)) {
        return
      }
      this.posting = true
      axios.post(this.postUrl, params,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.posting = false
          if (res.data.code === 1001) {
            console.log(res.data.msg)
            if (type === 2) {
              this.showNoticeSuccess('已发送申请', '请等待管理员审核')
            } else {
              this.showNoticeSuccess('操作成功', '请等待容器修改')
            }
          } else {
            this.showNoticeError('操作失败', '错误信息：' + res.data.msg)
          }
          this.$emit('update:showModalCheckVM', false)
        })
        .catch(err => {
          console.log(err)
        })
    },
    cancel () {
      this.$emit('update:showModalCheckVM', false)
    },
    transferDatetimeToStr (datetime) {
      if (datetime === '') { return '' }
      let year = datetime.getFullYear().toString()
      let month = (datetime.getMonth() + 1).toString()
      let day = datetime.getDate().toString()
      let hour = datetime.getHours().toString()
      let minute = datetime.getMinutes().toString()
      let second = datetime.getSeconds().toString()
      if (month.length === 1) { month = '0' + month }
      if (day.length === 1) { day = '0' + day }
      if (hour.length === 1) { hour = '0' + hour }
      if (minute.length === 1) { minute = '0' + minute }
      if (second.length === 1) { second = '0' + second }
      return year + '/' + month + '/' + day + ' ' + hour + ':' + minute + ':' + second
    }
  }
}
</script>

<style scoped src="../../../assets/ModalStyle.css"></style>
<style scoped>

</style>
