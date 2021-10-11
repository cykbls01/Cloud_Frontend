<template>
  <div class="content-list-box">
    <div class="list-box">
      <div class="list-block-box">
        <label>实验名称</label>
        <Span>{{name}}</Span>
      </div>
      <div class="list-block-box">
        <label>所属课程</label>
        <Span>{{course}}</Span>
      </div>
      <div class="list-block-box">
        <label>指导教师</label>
        <Span>{{teacher}}</Span>
      </div>
      <div class="list-block-box">
        <label>开始时间</label>
        <Span>{{starttime}}</Span>
      </div>
      <div class="list-block-box">
        <label>结束时间</label>
        <Span>{{endtime}}</Span>
      </div>
      <div class="list-block-box">
        <label>延迟提交时间</label>
        <Span>{{deadline}}</Span>
      </div>
      <div class="list-block-box">
        <label>互评截止时间</label>
        <Span>{{peerDeadline}}</Span>
      </div>
      <div class="list-block-box">
        <label>申诉截止时间</label>
        <Span>{{appealDeadline}}</Span>
      </div>
    </div>
    <Row>
      <Col>
        <label>实验说明</label>
        <div class="list-block">{{details}}</div>
      </Col>
    </Row>
    <Row>
      <Col>
        <label>互评细则</label>
        <div class="list-block">{{peerRules}}</div>
      </Col>
    </Row>
    <Row v-if="peerDeadline !== '无'">
      <Col span="4">
        <label>互评状态</label>
      </Col>
      <Col offset="2" span="10">
        <span>{{peerStarted === true ? '已开始' : '未开始'}}</span>
      </Col>
    </Row>
    <Row>
      <Col span="4">
        <label>参考资源</label>
        <div>{{resource}}</div>
      </Col>
      <Col offset="2" span="10">
        <div v-if="resource !== ''">
          <ButtonGroup>
              <Button @click="downloadExpFile">
                <Icon type="md-download"></Icon>
                下载
              </Button>
<!--            <Button @click="previewExpFile">-->
<!--              <Icon type="md-arrow-down"></Icon>-->
<!--              预览-->
<!--            </Button>-->
          </ButtonGroup>
        </div>
        <div v-else>
          无
        </div>
      </Col>
    </Row>
    <Row>
      <Col span="4">
        <label>作业提交状态</label>
        <Span>{{status}}</Span>
      </Col>
      <Col offset="2" span="10">
        <Button @click="checkHomework">
          <Icon type="md-search"></Icon>
          查看作业
        </Button>
      </Col>
    </Row>
  </div>
</template>

<script>
// import axios from 'axios'
// import net from '../../../assets/Global'
export default {
  name: 'ExpDetail',
  props: {
    expID: {
      type: Number,
      required: true
    },
    starttime: {
      type: String,
      required: true
    },
    endtime: {
      type: String,
      required: true
    },
    peerStarted: {
      type: Boolean,
      default: false
    },
    breadlist: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      name: '',
      course: '',
      teacher: '',
      deadline: '',
      peerDeadline: '',
      appealDeadline: '',
      peerRules: '',
      details: '',
      status: '',
      resource: ''
    }
  },
  watch: {
    expID () {
      this.getDetails()
    }
  },
  mounted () {
    this.getDetails()
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
    checkHomework () {
      sessionStorage.setItem('checkHomework_breadList', JSON.stringify(this.breadlist))
      sessionStorage.setItem('checkHomework_expID', String(this.expID))
      sessionStorage.setItem('checkHomework_expName', String(this.name))
      sessionStorage.setItem('checkHomework_peerStarted', String(this.peerStarted))
      this.$router.push({
        path: '/teacher/homework'
      })
    },
    downloadExpFile () {
      this.stuId = this.$cookie.get('userId')
      this.token = this.$cookie.get('token')
      window.open(this.$baseUrl + '/downloadExpResource?expId=' + this.expID)
      /* axios.get(net.neturl + '/tryDownload?stuId="' + this.stuId + '"&expId=' + this.expID + '&type=0')
        .then(res => {
          if (res.data.code === 1001) {
            let file = res.data.data
            window.open(net.neturl + '/downloadViaNetdisk?' + 'type=1&' + 'uuid=' + file)
          } else {
            this.$Message.error(res.data.msg)
          }
        }).catch(err => {
          console.log(err)
        }) */
    },
    previewExpFile () {
      // this.$http.get('preview', {
      //   params: {
      //     id: this.resource
      //   }
      // }).then(res => {
      //   if (typeof res.data === 'string') {
      //     // sessionStorage.setItem('callbackHTML', res.data)
      //     sessionStorage.setItem('previewFrom', 'expDetails')
      //     sessionStorage.setItem('expDetailBreadList', JSON.stringify(this.breadlist))
      //     this.$router.push({
      //       path: '/teacher/preview',
      //       query: {
      //         id: this.resource
      //       }
      //     })
      //   } else {
      //     this.showNoticeWarning('预览失败', '该文件不支持预览')
      //   }
      // }).catch(err => {
      //   console.log(err)
      // })
      // -------------separate---------------
      // sessionStorage.setItem('previewFrom', 'expDetails')
      // sessionStorage.setItem('expDetailBreadList', JSON.stringify(this.breadlist))
      // this.$router.push({
      //   path: '/teacher/preview',
      //   query: {
      //     id: this.resource
      //   }
      // })
      // -------------separate---------------
      // let fileURL = encodeURIComponent('http://10.251.254.37/preview?id=' + this.resource)
      // window.open('/static/pdf/web/viewer.html?file=' + fileURL)
      window.open('http://10.251.254.37/preview?id=' + String(this.resource))
    },
    getDetails () {
      if (this.expID === -1) {
        return
      }
      this.$http.get('getExperimentDetail', {
        params: {
          id: this.expID
        }
      }).then(res => {
        console.log(res.data.data)
        let rdata = res.data.data
        if (res.data.code === 1001) {
          this.name = rdata.name
          this.course = rdata.course
          this.teacher = rdata.teacher
          this.deadline = rdata.deadline
          this.peerStarted = rdata.peerStarted
          this.peerDeadline = rdata.peerDeadline
          this.appealDeadline = rdata.appealDeadline
          this.peerRules = rdata.peerAssessmentRule
          this.details = rdata.details
          this.resource = rdata.resource === null ? '' : rdata.resource
          this.status = rdata.status
          if (typeof this.peerDeadline === 'undefined' || this.peerDeadline === null || this.peerDeadline === '') {
            this.peerDeadline = '无'
          }
          if (typeof this.appealDeadline === 'undefined' || this.appealDeadline === null || this.appealDeadline === '') {
            this.appealDeadline = '无'
          }
          if (typeof this.details === 'undefined' || this.details === null || this.details === '') {
            this.details = '无'
          }
          if (typeof this.peerRules === 'undefined' || this.peerRules === null || this.peerRules === '') {
            this.peerRules = '无'
          }
        } else {
          this.$Message.error('获取实验详情失败：' + res.data.msg)
        }
      }).catch(err => {
        console.log(err)
      })
    },
    uploadSuccess (response, file, fileList) {
      this.$Notice.info({
        title: '上传成功',
        desc: file.name
      })
    }
  }
}
</script>

<style scoped>
  .content-list-box{
    font-size: large;
  }
  .list-box{
    display: flex;
    flex-wrap: wrap;
  }
  .content-list-box label{
    font-size: large;
    margin: 0 10px 0 5px;
    line-height: 36px;
  }
  .content-list-box span{
    font-size: large;
    font-weight: lighter;
  }
  .list-block-box{
    width: 50%;
  }
  .list-block{
    max-height: 200px;
    min-height: 100px;
    padding: 10px;
    margin-bottom: 10px;
    overflow-y: auto;
  }
  .source-list{
    margin: 7px 20px;
  }
  .source-list span{
    width: 150px;
    font-size: medium;
    display: inline-block;
  }
</style>
