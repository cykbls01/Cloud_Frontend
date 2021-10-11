<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel">
    <p slot="header" class="modal-title">
      <span>新增项目</span>
    </p>
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span class="input-item">实验名称</span>
        </Col>
        <Col span="17">
          <Select v-model="courseID" style="width:300px">
            <Option v-for="(item, index) in listCourse" :key="item.id" :value="item.id" :label="item.name">
              <span>{{item.name}}</span>
              <span style="float:right;color:#ccc">{{index + 1}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span  class="input-item">项目名称</span>
        </Col>
        <Col span="17">
          <Input v-model="name" clearable style="width: 300px" />
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
  name: 'addProject',
  props: {
    showModalAddExperiment: {
      type: Boolean,
      default: false
    },
    listCourse: {
      type: Array,
      default: function () {
        return []
      }
    },
    selectCourseID: {
      type: String,
      default: ''
    },
    selectCourseName: {
      type: String,
      default: ''
    }
  },
  watch: {
  },
  data () {
    return {
      postUrl: go.gourl + '/addProject',
      courseID: this.selectCourseID,
      fillCourseName: this.selectCourseName,
      name: '',
      beginTime: '',
      endTime: '',
      delayTime: '',
      description: '',
      enablePeerReview: false,
      posting: false,
      allDataUrl2: go.bugiturl + '/api/v1/stu/orgs',
      peerReviewDeadline: '',
      peerReviewDescription: '',
      appealDeadline: '',
      uploadServerUrl: 'fakeUpload',
      uploadParams: {
        id: '14211054',
        type: 2,
        parent: '1b44a1f8-5061-4de1-8a49-58de3fff2fd4'
      },
      file: null
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
    getAllData2 () {
      axios.get(this.allDataUrl2, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.listCourse = res.data.data
            if (this.listCourse === null) { this.listCourse = [] }
          } else {
            this.$Message.error(res.data.msg)
            this.listCourse = []
          }
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
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
    judgeParams (params) {
      let flag = true
      if (params.course_id === '' || typeof (params.course_id) === 'undefined') {
        this.showNoticeWarning('课程名称错误', '请选择课程名称')
        flag = false
      } else if (params.name === '') {
        this.showNoticeWarning('实验名称错误', '实验名称不能为空')
        flag = false
      }
      return flag
    },
    clearModal () {
      if (this.selectCourseID === '') { this.courseID = '' }
      this.name = ''
    },
    ok () {
      let params = {
        expId: Number(this.courseID),
        name: this.name
      }
      console.log(params)
      axios.post(this.postUrl, params,
        {
          headers: {
            'Content-Type':
                      'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          if (res.data.code === 1001) {
            console.log(res.data.msg)
            this.showNoticeSuccess('添加项目成功')
            this.$emit('refresh')
          } else {
            this.showNoticeError('添加项目失败', '错误信息：' + res.data.msg)
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
