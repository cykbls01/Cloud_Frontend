<template>
  <Modal v-model="show" @on-ok="ok"
         @on-cancel="cancel"
         :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-header">查看日志</p>
    <div v-for="(item, index) in Text" :key="index">
    <p rows="3" cols="20" style="background: #1c1f2b; color: #2baee9; font-size: medium">
      {{index}}. {{item}}
    </p>
    </div>
    <div slot="footer">
      <Button size="large" style="border: 0px" @click="cancel">取消</Button>
      <Button size="large" type="info" style="border: 0px" @click="getData">获取日志</Button>
    </div>
  </Modal>
</template>

<script>
import axios from 'axios'
import go from '../../../assets/Global'
export default {
  name: 'logContainer',
  props: {
    showModalCheckVM: {
      type: Boolean,
      default: false
    },
    podName: {
      type: String,
      default: ''
    },
    namespace: {
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
    }
  },
  created () {
    this.Text = ''
  },
  watch: {
  },
  methods: {
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
    showNoticeWarning (noticeTitle, noticeDesc) {
      this.$Notice.warning({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    getData () {
      let params = {
        podName: this.podName,
        namespace: this.namespace
      }
      axios.post(go.gourl + '/podLog', params,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          // this.posting = false
          if (res.data.code === 1001) {
            console.log(res.data.msg)
            this.Text = res.data.data
            // this.showNoticeSuccess('已发送申请', '请等待管理员审核')
          } else {
            this.showNoticeError('发送申请失败', '错误信息：' + res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    ok () {
      this.Text = ''
      this.$emit('update:showModalCheckVM', false)
    },
    cancel () {
      this.Text = ''
      this.$emit('update:showModalCheckVM', false)
    }
  },
  data () {
    return {
      loginUrl: null,
      OSName: '',
      Text: ''
    }
  }
}
</script>

<style scoped>
  .modal{
    color: #777777;
    text-align: right;
  }
  .modal-header {
    font-size: large;
    height: 40px;
    align-content: center;
    text-align: center;
    line-height: 50px;
    color: #777777;
  }
  .modal-row label{
    margin-top: 5px;
  }
  .modal-content{
    text-align: right;
    font-size: medium;
    font-family: 等线;
  }
  .modal-content Input{
    width: 50%;
    margin-left: 5px;
  }
</style>
