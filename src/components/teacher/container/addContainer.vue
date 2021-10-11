<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel"
    :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-title">
      <span>新增容器</span>
    </p>
    <div>
      <Row>
        <div>
          如果不需要端口号，填写0即可，如果使用自选镜像，请在镜像名称选择option
        </div>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">容器名称</span></Col>
        <Col span="17"><Input v-model="podName" placeholder="只能包含大小写字母，数字，下划线" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">镜像名称</span></Col>
        <Col span="17">
          <Select v-model="vmTemplate" style="width:350px" @on-change="selectFn($event)">
            <Option v-for="(item) in listTemplate" :key="item.name" :value="item.name" :label="item.name">
              <span>{{item.name}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="vmTemplate === 'option'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">请输入镜像名称</span></Col>
        <Col span="17"><Input v-model="image" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role === 'admin' || role === 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">使用者id</span></Col>
        <Col span="17"><Input v-model="userId" placeholder="填写使用者学号" clearable style="width: 350px"/></Col>
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
          <DatePicker v-model="due_time" :options="dueTimeOption" type="datetime" placeholder="请选择使用截止时间" style="width: 350px" :disabled="changeVM"></DatePicker>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="role !== 'admin' && role !== 'superAdmin'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">申请原因</span></Col>
        <Col span="17"><Input v-model="applyReason" type="textarea" style="width:350px" :rows="10" placeholder="请简述申请理由及使用时间" :disabled="changeVM" /></Col>
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
  name: 'addContainer',
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
      postUrl: go.gourl + '/applyContainer',
      inputVMName: '',
      applyReason: '',
      vmTemplate: '',
      due_time: '',
      posting: false,
      listPort: [],
      listEnv: [],
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
      this.listPort = []
      this.listEnv = []
      for (let i = 0; i < this.listTemplate.length; i++) {
        if (this.listTemplate[i].name === e) {
          for (let j = 0; j < this.listTemplate[i].port.length; j++) {
            let a = {
              value: this.listTemplate[i].port[j]
            }
            this.listPort.push(a)
          }
          for (let j = 0; j < this.listTemplate[i].env.length; j++) {
            let a = {
              label: this.listTemplate[i].env[j]
            }
            this.listEnv.push(a)
          }
        }
      }
    },
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
      axios.post(go.gourl + '/listImage',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          if (res.data.code === 1001) {
            this.listTemplate = res.data.data
            var nnn = {name: 'option'}
            this.listTemplate.push(nnn)
          } else {
            this.$Message.error('获取模板信息失败，' + res.data.msg)
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
      } else if (params.image === '' || typeof (params.image) === 'undefined') {
        this.showNoticeWarning('镜像名称错误', '镜像名称不能为空')
        flag = false
      } else if ((params.end_time === '' || typeof (params.end_time) === 'undefined') && type === 1) {
        this.showNoticeWarning('使用截止时间错误', '请选择容器的使用截止时间')
        flag = false
      } else if (params.msg === '' && type === 1) {
        this.showNoticeWarning('申请原因错误', '申请原因不能为空')
        flag = false
      } else if (params.userId === '' && type === 3) {
        this.showNoticeWarning('使用者id错误', '使用者id不能为空')
        flag = false
      } else {
        let pattern = /^[\w-]+$/
        flag = pattern.test(params.name)
        flag = flag && (params.name.indexOf('_') === -1)
        pattern = /[0-9]/
        flag = flag && !pattern.test(params.name.substring(0, 1))
        if (!flag) { this.showNoticeWarning('容器名称错误', '容器名称只能包含大小写字母，数字，中划线，并且数字不能放在最前') }
      }
      return flag
    },
    extractNumber (str, timeNum = 1) {
      let matchRes = str.match(/^\d+/)
      return matchRes === null ? null : Number(matchRes[0]) * timeNum
    },
    ok () {
      if (this.vmTemplate !== 'option') {
        this.image = this.vmTemplate
      }
      let listp = []
      for (let i = 0; i < this.listPort.length; i++) {
        listp.push(this.listPort[i].value)
      }
      let type = 0
      if (this.role === 'student' || this.role === 'teacher') {
        type = 1
      } else if (this.role === 'admin' || this.role === 'superAdmin') {
        type = 3
      }
      let params = {
        name: this.podName,
        podname: this.podName,
        port: listp,
        env: this.listEnv,
        image: this.image,
        msg: this.applyReason,
        url: this.vmTemplate,
        end_time: this.transferDatetimeToStr(this.due_time),
        userId: this.userId,
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
            if (type === 1) {
              this.showNoticeSuccess('已发送申请', '请等待管理员审核')
            } else {
              this.showNoticeSuccess('操作成功', '请等待容器创建')
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
