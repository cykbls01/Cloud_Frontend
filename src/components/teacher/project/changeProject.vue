<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel">
    <p slot="header" style="margin-top:10px;height:25px;font-size: 20px;text-align:center;font-weight:normal">
      <span>修改项目</span>
    </p>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span style="font-size: 16px">项目名称：</span>
        </Col>
        <Col span="17">
          <Input v-model="courseName" clearable style="width: 300px" />
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
  name: 'changeProject',
  props: {
    showModalChangeCourse: {
      type: Boolean,
      default: false
    },
    originCourseID: {
      type: String,
      required: true
    },
    originCourseName: {
      type: String,
      required: true
    },
    originCourseTerm: {
      type: Number,
      required: true
    },
    listTerm: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  watch: {
    originCourseName: function (newval, oldval) {
      this.courseName = newval
    },
    originCourseTerm: function (newval, oldval) {
      this.courseTerm = newval
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalChangeCourse
      },
      set: function () { }
    }
  },
  data () {
    return {
      courseTerm: this.originCourseTerm,
      postUrl: go.gourl + '/updateProject',
      courseName: this.originCourseName,
      posting: false
    }
  },
  methods: {
    showNoticeWarning (noticeTitle, noticeDesc) {
      this.$Notice.warning({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    judgeParams (params) {
      let flag = true
      if (params.name === '' || typeof (params.name) === 'undefined') {
        this.showNoticeWarning('项目名称错误', '项目名称不能为空')
        flag = false
      }
      return flag
    },
    clearModal () {
      this.courseName = ''
    },
    ok () {
      let params = {
        proId: String(this.originCourseID),
        name: this.courseName
      }
      if (!this.judgeParams(params)) {
        return
      }
      this.posting = true
      axios.post(this.postUrl, params, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.clearModal()
            this.$emit('showOperationRes', '操作成功', '已成功修改课程信息')
            this.$emit('refresh')
          } else {
            this.$emit('showOperationRes', '操作失败', res.data.msg)
          }
          this.posting = false
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.$emit('update:showModalChangeCourse', false)
        })
    },
    cancel () {
      this.$emit('update:showModalChangeCourse', false)
    }
  }
}
</script>

<style scoped>

</style>
