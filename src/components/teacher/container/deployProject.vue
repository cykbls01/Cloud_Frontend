<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel"
    :styles="{top: '50px', width: '580px'}">
    <p slot="header" class="modal-title">
      <span>部署项目</span>
    </p>
    <div>
      <Row>
        <div>
          请选择要部署的仓库和分支
        </div>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">仓库名称</span></Col>
        <Col span="17">
          <Select v-model="repoName" style="width:350px" @on-change="selectFn($event)">
            <Option v-for="(item) in listRepo" :key="item.Name" :value="item.ID" :label="item.Name">
              <span>{{item.Name}}</span>
            </Option>
          </Select>
        </Col>
      </Row>
      <Row justify="end" class="code-row-bg" style="padding-top:2%">
        <Col span="7" style="text-align: center;margin-top:1%"><span class="input-item">仓库分支</span></Col>
        <Col span="17">
          <Select v-model="branchName" style="width:350px">
            <Option v-for="(item) in listBranch" :key="item.Name" :value="item.Name" :label="item.Name">
              <span>{{item.Name}}</span>
            </Option>
          </Select>
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
  name: 'deployProject',
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
      podName: '',
      listRepo: [],
      listBranch: [],
      postUrl: go.bugiturl + '/projects/' + this.projectId + '/deploy',
      applyReason: '',
      due_time: '',
      posting: false,
      repoName: '',
      branchName: '',
      role: ''
    }
  },
  watch: {
    changeVM (newval, oldval) {
      if (newval) {
      } else {
      }
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
      for (let i = 0; i < this.listRepo.length; i++) {
        if (this.listRepo[i].ID === e) {
          console.log(this.listRepo)
          this.listBranch = this.listRepo[i].Branches
        }
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
      axios.get(go.bugiturl + '/projects/' + this.projectId + '/repos',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.listRepo = res.data
        })
        .catch(err => {
          console.log(err)
        })
    },
    gouzao () {
      this.getListTemplate()
    },
    clearModal () {
      this.branchName = ''
      this.repoName = ''
    },
    judgeParams (params) {
      let flag = true
      if (params.RepoID === '') {
        this.showNoticeWarning('未选择仓库', '请选择仓库')
        flag = false
      } else if (params.Branch === '' || typeof (params.Branch) === 'undefined') {
        this.showNoticeWarning('未选择分支', '请选择分支')
        flag = false
      } else {
      }
      return flag
    },
    ok () {
      let params = {
        RepoID: this.repoName,
        Branch: this.branchName
      }
      if (!this.judgeParams(params)) {
        return
      }
      this.posting = true
      axios.post(this.postUrl, params,
        {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.posting = false
          console.log(res.data.msg)
          this.showNoticeSuccess('已提交部署', '请等待部署')
          this.clearModal()
          this.$emit('refresh')
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
