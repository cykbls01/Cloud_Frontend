<template>
  <Modal v-model="show" @on-ok="ok"
         @on-cancel="cancel"
         :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-header">查看容器链接</p>
    <Row>
      <div style="font-size: medium">
        如果不处于校园网环境，请复制该链接，于d.buaa.edu.cn进行访问
      </div>
    </Row>
    <div v-for="(item, index) in url" :key="index" style="font-size: medium">
      <a :href="item" target="view_window">链接{{index}}</a>
    </div>
    <div slot="footer">
      <Button size="large" style="border: 0px" @click="cancel">取消</Button>
      <Button size="large" type="info" style="border: 0px" @click="cancel">确认</Button>
    </div>
  </Modal>
</template>

<script>
export default {
  name: 'checkContainer',
  props: {
    showModalCheckVM: {
      type: Boolean,
      default: false
    },
    podName: {
      type: String,
      default: ''
    },
    url: {
      type: Array
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
    ok () {
      console.log(this.url)
      this.$emit('update:showModalCheckVM', false)
    },
    cancel () {
      console.log(this.url)
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
