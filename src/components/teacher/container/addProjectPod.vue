<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel"
    :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-title">
      <span>为项目新增基础服务</span>
    </p>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">服务名称</span></Col>
        <Col span="17"><Input v-model="podName" placeholder="只能包含大小写字母，数字，中划线" clearable style="width: 350px"/></Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">镜像名称</span></Col>
        <Col span="17">
          <Select v-model="image" style="width:350px" @on-change="selectFn($event)">
            <Option v-for="(item) in listImage" :key="item.name" :value="item.name" :label="item.name">
              <span>{{item.name}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">镜像版本</span></Col>
        <Col span="17">
          <Select v-model="version" style="width:350px">
            <Option v-for="(item) in listVersion" :key="item.name" :value="item.name" :label="item.name">
              <span>{{item.name}}</span>
            </Option>
          </Select>
        </Col>
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
  name: 'addProjectPod',
  props: {
    showModalAddVM: {
      type: Boolean,
      default: false
    },
    expId: {
      type: String,
      default: ''
    },
    projectId: {
      type: String,
      default: ''
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
      version: '',
      podName: '',
      listImage: [],
      listVersion: [],
      postUrl: go.gourl + '/applyContainer123',
      applyReason: '',
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
    }
  },
  mounted () {
    this.role = this.$cookie.get('role')
    console.log(this.projectId)
    console.log(this.expId)
    this.gouzao()
  },
  methods: {
    selectFn (e) {
      for (let i = 0; i < this.listImage.length; i++) {
        if (this.listImage[i].name === e) {
          this.listVersion = this.listImage[i].versions
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
      axios.post(go.bugiturl + '/listImage',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.listImage = res.data
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
    },
    judgeParams (params) {
      let flag = true
      if (params.name === '') {
        this.showNoticeWarning('容器名称错误', '容器名称不能为空')
        flag = false
      } else if (params.image === '' || typeof (params.image) === 'undefined') {
        this.showNoticeWarning('镜像名称错误', '镜像名称不能为空')
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
      let params = {
        name: this.podName,
        podname: this.podName,
        env: this.listEnv,
        image: this.image,
        userId: this.userId
      }
      if (!this.judgeParams(params)) {
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
            this.showNoticeSuccess('已发送申请', '请等待管理员审核')
            this.clearModal()
            this.$emit('refresh')
          } else {
            this.showNoticeError('发送申请失败', '错误信息：' + res.data.msg)
          }
          this.$emit('update:showModalAddVM', false)
        })
        .catch(err => {
          console.log(err)
        })
    },
    cancel () {
      this.$emit('update:showModalAddVM', false)
    }
  }
}
</script>

<style scoped src="../../../assets/ModalStyle.css"></style>
<style scoped>

</style>
