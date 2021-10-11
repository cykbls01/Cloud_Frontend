<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel"
    :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-title">
      <span>新增共享容器</span>
    </p>
    <div>
      <Row>
<!--        <div>-->
<!--          如果不需要端口号，填写0即可，如果使用自选镜像，请在镜像名称选择option-->
<!--        </div>-->
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">选择共享容器</span></Col>
        <Col span="17">
          <Select v-model="vmTemplate" style="width:350px" @on-change="selectFn($event)">
            <Option v-for="(item) in listTemplate" :key="item.name" :value="item.name" :label="item.name">
              <span>{{item.name}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">请输入数据库名称(用户名)</span></Col>
        <Col span="17"><Input v-model="dbname" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">请输入密码</span></Col>
        <Col span="17"><Input v-model="password" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role === 'admin' || role === 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">使用者id</span></Col>
        <Col span="17"><Input v-model="userId" placeholder="填写使用者学号" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role !== 'admin' && role !== 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span class="input-item">使用截止时间</span>
        </Col>
        <Col span="17">
          <DatePicker v-model="due_time" :options="dueTimeOption" type="datetime" placeholder="请选择使用截止时间" style="width: 350px" ></DatePicker>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role !== 'admin' && role !== 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">申请原因</span></Col>
        <Col span="17"><Input v-model="applyReason" type="textarea" style="width:350px" :rows="10" placeholder="请简述申请理由及使用时间" /></Col>
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
  name: 'addSharePod',
  props: {
    showModalAddVM: {
      type: Boolean,
      default: false
    },
    expID: {
      type: Number,
      default: -1
    },
    expName: {
      type: String,
      default: ''
    },
    expApplyId: {
      type: Number,
      default: -1
    },
    changeVM: {
      type: Boolean,
      default: false
    },
    operVMConf: {
      type: Object,
      required: this.changeVM
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalAddVM
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
      podName: '',
      listTemplate: [],
      postUrl: go.gourl + '/createSharePod',
      inputVMName: '',
      applyReason: '',
      vmTemplate: '',
      due_time: '',
      posting: false,
      listPort: [],
      listEnv: [],
      dbname: '',
      password: '',
      role: '',
      userId: ''
    }
  },
  watch: {
    changeVM (newval, oldval) {
      if (newval) {
        this.inputVMName = this.operVMConf.name
        this.vmTemplate = this.operVMConf.template
        this.due_time = this.operVMConf.due_time
      } else {
        this.inputVMName = ''
        this.vmTemplate = ''
        this.due_time = ''
      }
    }
  },
  mounted () {
    this.role = this.$cookie.get('role')
    this.gouzao()
  },
  methods: {
    selectFn (e) {
      for (let i = 0; i < this.listTemplate.length; i++) {
        if (this.listTemplate[i].name === e) {
          this.image = this.listTemplate[i].conname
          this.podName = this.listTemplate[i].name
          return
        }
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
      axios.post(go.gourl + '/listShareContainer',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          if (res.data.code === 1001) {
            this.listTemplate = res.data.data
            console.log(this.listTemplate)
          } else {
            this.$Message.error('获取容器信息失败，' + res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    gouzao () {
      this.getListTemplate()
    },
    clearModal () {
      this.podName = ''
      this.image = ''
      this.userId = ''
      this.dbname = ''
      this.applyReason = ''
      this.password = ''
    },
    judgeParams (params, type) {
      let flag = true
      if (params.db_name === '') {
        this.showNoticeWarning('账户名称错误', '账户名称不能为空')
        flag = false
      } else if (params.con_name === '' || typeof (params.con_name) === 'undefined') {
        this.showNoticeWarning('容器选择错误', '请选择容器')
        flag = false
      } else if (params.password === '' || typeof (params.password) === 'undefined') {
        this.showNoticeWarning('账户密码错误', '账户密码不能为空')
        flag = false
      } else if ((params.due_time === '' || typeof (params.due_time) === 'undefined') && type === 1) {
        this.showNoticeWarning('使用截止时间错误', '请选择容器的使用截止时间')
        flag = false
      } else if (params.msg === '' && type === 1) {
        this.showNoticeWarning('申请原因错误', '申请原因不能为空')
        flag = false
      } else if (params.user_id === '' && type === 3) {
        this.showNoticeWarning('使用者id错误', '使用者id不能为空')
        flag = false
      } else {
        let pattern = /^[\w-]+$/
        flag = pattern.test(params.name)
        if (!flag) { this.showNoticeWarning('容器名称错误', '容器名称只能包含大小写字母，数字，下划线和中划线') }
      }
      return flag
    },
    extractNumber (str, timeNum = 1) {
      let matchRes = str.match(/^\d+/)
      return matchRes === null ? null : Number(matchRes[0]) * timeNum
    },
    ok () {
      let type = 0
      if (this.role === 'student' || this.role === 'teacher') {
        type = 1
      } else if (this.role === 'admin' || this.role === 'superAdmin') {
        type = 3
      }
      let params = {
        db_name: this.dbname,
        pod_name: this.podName,
        con_name: this.image,
        msg: this.applyReason,
        due_time: this.transferDatetimeToStr(this.due_time),
        user_id: this.userId,
        password: this.password
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
            if (type === 1) {
              this.showNoticeSuccess('已发送申请', res.data.msg)
            } else {
              this.showNoticeSuccess('操作成功', res.data.msg)
            }
            this.clearModal()
            this.$emit('refresh')
          } else {
            this.showNoticeError('操作失败', '错误信息：' + res.data.msg)
          }
          this.$emit('update:showModalAddVM', false)
        })
        .catch(err => {
          console.log(err)
        })
      this.posting = false
    },
    cancel () {
      this.$emit('update:showModalAddVM', false)
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
