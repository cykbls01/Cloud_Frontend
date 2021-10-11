<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel">
    <p slot="header" class="modal-title">
      <span>修改镜像</span>
    </p>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">镜像链接</span>
        </Col>
        <Col span="17">
          <Input v-model="url" clearable style="width: 300px" :disabled="true"/>
        </Col>
      </Row>
    </div>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">镜像名称</span>
        </Col>
        <Col span="17">
          <Input v-model="name" clearable style="width: 300px" :disabled="true"/>
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
  name: 'modifyImage',
  props: {
    showModalAddExperiment: {
      type: Boolean,
      default: false
    },
    oldImage: {
      type: Object,
      default: null
    }
  },
  watch: {
    oldImage (newval, oldval) {
      this.port = ''
      this.env = ''
      this.url = this.oldImage.url
      this.name = this.oldImage.name
      if (this.showModalAddExperiment === false) {
        return
      }
      for (let i = 0; i < this.oldImage.port.length - 1; i++) {
        this.port += this.oldImage.port[i] + ' '
      }
      this.port += this.oldImage.port[this.oldImage.port.length - 1]
      for (let i = 0; i < this.oldImage.env.length - 1; i++) {
        this.env += this.oldImage.env[i] + ' '
      }
      this.env += this.oldImage.env[this.oldImage.env.length - 1]
      if (typeof this.port === 'undefined' || this.port === null || this.port === 'undefined') {
        this.port = ''
      }
      if (typeof this.env === 'undefined' || this.env === null || this.env === 'undefined') {
        this.env = ''
      }
    }
  },
  data () {
    return {
      postUrl: go.gourl + '/updateImage',
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
    clearModal () {
      this.port = ''
      this.env = ''
    },
    ok () {
      let params = {
        id: this.oldImage.id,
        port: this.port.split(' '),
        env: this.env.split(' ')
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
            this.showNoticeSuccess('修改镜像成功')
            this.clearModal()
            this.$emit('refresh')
          } else {
            this.showNoticeError('修改镜像失败', '错误信息：' + res.data.msg)
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
