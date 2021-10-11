<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel">
    <p slot="header" class="modal-title">
      <span>新增项目</span>
    </p>
<!--    <a :href="registerUrl" target="_blank">如果没有注册代码平台，请先注册</a>-->
    <div>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%">
          <span class="input-item">选择小组</span>
        </Col>
        <Col span="17">
          <Select v-model="organName" style="width:300px">
            <Option v-for="(item, index) in listorgan" :key="item.id" :value="item.username" :label="item.username">
              <span>{{item.username}}</span>
              <span style="float:right;color:#ccc">{{index + 1}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%" v-if="organName === '新建小组'">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">请输入小组名称</span></Col>
        <Col span="17"><Input v-model="newOrganName" clearable style="width: 300px"/></Col>
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
  name: 'addExpProject',
  props: {
    showModalAddExperiment: {
      type: Boolean,
      default: false
    },
    selectCourseID: {
      type: String,
      default: ''
    },
    info: {
      type: Object,
      default: null
    }
  },
  watch: {
    info (newval, oldval) {
      if (this.showModalAddExperiment === false) {
        return
      }
      this.getData()
    }
  },
  data () {
    return {
      postUrl: go.bugiturl + '/project',
      courseID: this.selectCourseID,
      name: '',
      newOrganName: '',
      registerUrl: go.bugiturl1 + '/user/sign_up',
      beginTime: '',
      endTime: '',
      delayTime: '',
      description: '',
      enablePeerReview: false,
      posting: false,
      listorgan: [],
      peerReviewDeadline: '',
      peerReviewDescription: '',
      appealDeadline: '',
      organName: '',
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
    getData () {
      axios.get(go.bugiturl + '/user/orgs', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          this.listorgan = res.data
          if (this.listorgan === null) {
            this.listorgan = []
          }
          var nnn = {username: '新建小组', id: -1}
          this.listorgan.push(nnn)
          console.log(this.listorgan)
        })
        .catch(err => {
          console.log(err)
        })
      // var nnn = {full_name: '新建小组', id: -1}
      // this.listorgan.push(nnn)
      // console.log(this.listorgan)
    },
    clearModal () {
      if (this.selectCourseID === '') { this.courseID = '' }
      this.name = ''
    },
    ok () {
      if (this.organName === '新建小组') {
        this.organName = this.newOrganName
      }
      let params = {
        organizationName: this.organName,
        projectName: this.name,
        expName: this.info.expName,
        expId: parseInt(this.info.expId),
        courseName: this.info.courseName,
        courseId: parseInt(this.info.courseId)
      }
      console.log(params)
      axios.post(this.postUrl, params,
        {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          console.log(res.data)
          this.showNoticeSuccess('添加项目成功')
          this.$emit('refresh')
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
