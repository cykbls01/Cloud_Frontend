<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel" width="800px">
    <p slot="header" style="margin-top:10px;height:25px;font-size: 20px;text-align:center;font-weight:normal">
      <span>查看项目<strong>{{courseName}}</strong>部署状态</span>
    </p>
    <div style="margin-top:10px;margin-left:20px;margin-right:20px;">
      <TableComponent1 :buttonSpan="12" :loading="loading" @refresh="refreshData2" ref="table" :columnsFromFather="columns2" :allDataFromFather="allData3">
      </TableComponent1>
    </div>
    <div slot="footer">
      <Button type="text" size="large" @click="cancel">取消</Button>
      <Button type="primary" size="large" @click="ok">确定</Button>
    </div>
  </Modal>
</template>

<script>
import TableComponent1 from '../../TableComponent1'
import axios from 'axios'
import go from '../../../assets/Global'
export default {
  name: 'CheckProject',
  components: {
    TableComponent1
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
      loading: true,
      columns2: [
        {
          title: '仓库名',
          key: 'RepoName',
          width: '150'
        },
        {
          title: '仓库分支',
          key: 'Branch'
        },
        {
          title: '部署状态',
          key: 'Status',
          render: (h, params) => {
            let status = 'error'
            let label = '异常状态'
            if (params.row.Status === 1) {
              status = '#000000'
              label = '未进行部署'
            } else if (params.row.Status === 2) {
              status = 'success'
              label = '部署中'
            } else if (params.row.Status === 3) {
              status = 'success'
              label = '部署完成'
            }
            return h('div', [
              h('Tag', {
                props: {
                  color: status
                }
              }, label)
            ])
          }
        }
        // {
        //   title: '部署参数',
        //   key: 'str'
        // }
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
    getAllData2 () {
      axios.get(this.allDataUrl2 + String(this.courseID) + '/deploy', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          this.allData2 = res.data
          if (this.allData2 === null) {
            this.allData2 = []
          }
          this.allData2.forEach(it => {
            if (it.Status === 1 || it.Status === 2 || it.Status === 3) {
              it.str = JSON.stringify(it)
              this.allData3.push(it)
            }
          })
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
