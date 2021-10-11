<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel" width="600px">
    <p slot="header" style="margin-top:10px;height:25px;font-size: 20px;text-align:center;font-weight:normal">
      <span>为项目<strong>《{{courseName}}》</strong>批量添加组员</span>
    </p>
    <div>
      <Input v-model="studentsText" type="textarea" :rows="8" placeholder="请输入学生学号，并以空格分割..." />
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
  name: 'addMember',
  props: {
    showModalBatchAddStudents: {
      type: Boolean,
      default: false
    },
    courseID: {
      type: String,
      required: true
    },
    courseName: {
      type: String,
      required: true
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalBatchAddStudents
      },
      set: function () {
      }
    }
  },
  data () {
    return {
      studentsText: '',
      posting: false
    }
  },
  methods: {
    sleep (time) {
      return new Promise(resolve => setTimeout(resolve, time))
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
    judgeText () {
      let pattern = /^[\w\s\d]*$/
      let flag = pattern.test(this.studentsText)
      if (!flag) {
        this.showNoticeWarning('输入错误', '输入应只包含数字与空格')
      }
      return flag
    },
    clearModal () {
      this.studentsText = ''
    },
    ok () {
      if (!this.judgeText()) {
        return
      }
      let stuList = this.studentsText.trim().split(/\s+/)
      this.posting = true
      axios.post(go.gourl + '/addMember', {
        proId: String(this.courseID),
        stuId: stuList
      }, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.clearModal()
            console.log(res.data)
            this.showNoticeSuccess(res.data.msg, res.data.data)
            this.sleep(1500).then(() => {
              window.location.reload()
            })
          } else {
            this.showNoticeError(res.data.msg, res.data.data)
          }
          this.posting = false
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.$emit('update:showModalBatchAddStudents', false)
        })
    },
    cancel () {
      this.$emit('update:showModalBatchAddStudents', false)
    }
  }
}
</script>

<style scoped>

</style>
