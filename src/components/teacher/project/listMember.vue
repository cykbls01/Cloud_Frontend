<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel" width="800px">
    <p slot="header" style="margin-top:10px;height:25px;font-size: 20px;text-align:center;font-weight:normal">
      <span>查看项目<strong>{{courseName}}</strong>中的成员</span>
    </p>
    <div style="margin-top:10px;margin-left:20px;margin-right:20px;">
      <TableComponent :buttonSpan="12" :loading="loading" @refresh="refreshData2" ref="table" :columnsFromFather="columns2" :allDataFromFather="allData3">
      </TableComponent>
    </div>
    <div slot="footer">
      <Button type="text" size="large" @click="cancel">取消</Button>
      <Button type="primary" size="large" @click="ok">确定</Button>
    </div>
  </Modal>
</template>

<script>
import TableComponent from '../../TableComponent'
import axios from 'axios'
import go from '../../../assets/Global'
export default {
  name: 'ListMember',
  components: {
    TableComponent
  },
  props: {
    showModalCheckStudent: {
      type: Boolean,
      default: false
    },
    courseName: {
      type: String,
      required: true
    },
    courseID: {
      type: String,
      required: true
    }
  },
  computed: {
    show: {
      get: function () {
        return this.showModalCheckStudent
      },
      set: function () {
      }
    }
  },
  data () {
    return {
      allData2: [],
      allData3: [],
      allDataUrl2: go.bugiturl + '/projects/',
      allDataUrl1: go.gourl + '/getUser',
      loading: true,
      columns2: [
        // {
        //   title: '序号',
        //   key: 'index',
        //   sortable: 'custom',
        //   sortType: 'number'
        // },
        {
          title: '学生学号',
          key: 'StudentID',
          width: '150'
        },
        {
          title: '学生姓名',
          key: 'Name'
        },
        // {
        //   title: '所在学院',
        //   key: 'department',
        //   sortable: 'custom'
        // },
        {
          title: '学生邮箱',
          key: 'Email',
          sortable: 'custom'
        }
      ]
    }
  },
  watch: {
    courseID: function (newval, oldval) {
      if (this.show === false) {
        return
      }
      console.log(this.courseID)
      this.loading = true
      this.getAllData2()
    }
  },
  methods: {
    getAllData3 () {
      this.allData2 = ['17373275', '17373274']
      axios.post(this.allDataUrl1, {
        stuId: this.allData2
      }, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.allData3 = res.data.data
          } else {
            this.$Message.error(res.data.msg)
            this.allData2 = []
          }
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    getAllData2 () {
      axios.get(this.allDataUrl2 + String(this.courseID) + '/members', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          this.allData2 = res.data
          if (this.allData2 === null) { this.allData2 = [] }
          this.allData3 = this.allData2
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    refreshData2 () {
      this.loading = true
      this.getAllData2()
    },
    showNoticeWarning (noticeTitle, noticeDesc) {
      this.$Notice.warning({
        title: noticeTitle,
        desc: noticeDesc
      })
    },
    ok () {
      this.$emit('update:showModalCheckStudent', false)
    },
    cancel () {
      this.$emit('update:showModalCheckStudent', false)
    }
  }
}
</script>

<style scoped>
  .delIcon{
    padding-right:2px;
    padding-bottom: 3px;
  }
  /deep/ .ivu-btn-group-default{
    display: inline;
    float: left;
    margin-left: -50px;
  }
  /deep/ .ivu-poptip-rel{
    display: inline !important;
    float: left;
  }
  /deep/ .ivu-input {
    display: inline;
  }
</style>
