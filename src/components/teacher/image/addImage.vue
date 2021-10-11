<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel">
    <p slot="header" class="modal-title">
      <span>新增镜像</span>
    </p>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">镜像链接</span>
        </Col>
        <Col span="17">
          <Input v-model="url" clearable style="width: 300px" />
        </Col>
      </Row>
    </div>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">镜像名称</span>
        </Col>
        <Col span="17">
          <Input v-model="name" clearable style="width: 300px" />
        </Col>
      </Row>
    </div>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">端口号</span>
        </Col>
        <Col span="17">
          <Input v-model="port" clearable style="width: 300px" placeholder="使用空格分开"/>
        </Col>
      </Row>
    </div>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">环境配置</span>
        </Col>
        <Col span="17">
          <Input v-model="env" clearable style="width: 300px" placeholder="使用空格分开"/>
        </Col>
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
  name: 'addImage',
  props: {
    showModalAddExperiment: {
      type: Boolean,
      default: false
    }
  },
  watch: {
  },
  data () {
    return {
      postUrl: go.gourl + '/addImage',
      name: '',
      posting: false,
      url: '',
      port: '',
      env: ''
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalAddExperiment
      },
      set: function () {
      }
    }
  },
  methods: {
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
    judgeParams (params) {
      let flag = true
      if (params.image === '') {
        this.showNoticeWarning('镜像名称错误', '镜像名称不能为空')
        flag = false
      } else if (params.url === '') {
        this.showNoticeWarning('镜像链接错误', '镜像链接不能为空')
        flag = false
      }
      return flag
    },
    clearModal () {
      this.name = ''
      this.url = ''
      this.port = ''
      this.env = ''
    },
    ok () {
      let params = {
        port: this.port.split(' '),
        env: this.env.split(' '),
        url: this.url,
        tag: this.name,
        image: this.name
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
            this.showNoticeSuccess(res.data.msg)
            this.clearModal()
            this.$emit('refresh')
          } else {
            this.showNoticeError('添加镜像失败', '错误信息：' + res.data.msg)
          }
          this.$emit('update:showModalAddExperiment', false)
        })
        .catch(err => {
          console.log(err)
        })
    },
    cancel () {
      this.$emit('update:showModalAddExperiment', false)
    }
  }
}
</script>

<style scoped src="../../../assets/ModalStyle.css"></style>
<style scoped>

</style>
