<template>
 <div>
   <NavBar :breadlist="breadlist">
     <div slot="navbarContent">
       <CardContent>
         <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="allData"></TableComponent>
       </CardContent>
       <CardContent>
         <div style="margin-left: 10px;">
<!--           <Row>-->
<!--             切换日志:-->
<!--             <Select v-model="logFile" style="width:300px">-->
<!--               <Option v-for="(item) in listLog" :key="item.id" :value="item" :label="item">-->
<!--                 <span>{{item}}</span>-->
<!--               </Option>-->
<!--             </Select>-->
<!--           </Row>-->
         </div>
         <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns2" :allDataFromFather="allData1"></TableComponent>
       </CardContent>
     </div>
   </NavBar>
 </div>
</template>

<script>
import NavBar from '../../components/NavBar'
import CardContent from '../../components/CardContent'
import TableComponent from '../../components/TableComponent'
import axios from 'axios'
import go from '../../assets/Global'
export default {
  name: 'operationLog',
  components: {TableComponent, CardContent, NavBar},
  data () {
    return {
      breadlist: [{name: '操作日志', path: '/operation/log'}],
      logFile: '',
      listLog: [],
      allData: [],
      allData1: [],
      load: true,
      columns1: [
        {
          title: '序号',
          key: 'index',
          align: 'center',
          maxWidth: 70
        },
        // {
        //   title: '操作对象类型',
        //   key: 'operate_type',
        //   align: 'center'
        // },
        {
          title: '操作类型',
          key: 'operate_target_type',
          align: 'center',
          sortable: 'custom'
        },
        {
          title: '操作对象ID',
          key: 'operate_target_id',
          sortable: true
        },
        {
          title: '操作者ID',
          key: 'operator_id',
          sortable: true
        },
        {
          title: '操作者角色',
          key: 'operator_role',
          sortable: 'custom',
          render: (h, params) => {
            let text = '学生'
            switch (params.row.operator_role) {
              case 1: text = '学生'; break
              case 2: text = '教师'; break
              case 3: text = '管理员'; break
              case 4: text = '管理员'; break
              default: text = 'unknown role'
            }
            return h('div', text)
          }
        },
        {
          title: '操作时间',
          key: 'time',
          sortable: 'custom'
        },
        {
          title: '状态',
          key: 'state',
          tooltip: true,
          sortable: 'custom',
          align: 'center'
          // render: (h, params) => {
          //   let status = {
          //     color: '',
          //     text: ''
          //   }
          //   switch (params.row.state) {
          //     case 'complete': status.color = 'primary'; status.text = '完成'; break
          //     case 'error': status.color = 'error'; status.text = '异常'; break
          //     case 'running': status.color = 'success'; status.text = '进行'
          //   }
          //   return h('div', [
          //     h('Tag', {
          //       props: {
          //         color: status.color
          //       }
          //     }, status.text)
          //   ])
          // }
        },
        {
          title: '完成时间',
          key: 'complete_time',
          sortable: true
        },
        {
          title: '操作者IP',
          key: 'user_ip'
        },
        // {
        //   title: '操作',
        //   align: 'center',
        //   render: (h, params) => {
        //     return h('div', [
        //       h('Button', {
        //         props: {
        //           type: 'info'
        //         },
        //         style: {
        //           marginRight: '5px'
        //         },
        //         on: {
        //           click: () => {
        //           }
        //         }
        //       }, '详情')
        //     ])
        //   }
        // }
        {
          title: '详情',
          key: 'content',
          tooltip: true
        }
      ],
      columns2: [
        {
          title: '序号',
          key: 'index',
          align: 'center',
          maxWidth: 70
        },
        {
          title: '操作接口',
          key: 'path',
          align: 'center',
          sortable: 'custom'
        },
        {
          title: '操作者ID',
          key: 'user_id',
          sortable: true
        },
        // {
        //   title: '操作者角色',
        //   key: 'operator_role',
        //   sortable: 'custom',
        //   render: (h, params) => {
        //     let text = '学生'
        //     switch (params.row.operator_role) {
        //       case 1: text = '学生'; break
        //       case 2: text = '教师'; break
        //       case 3: text = '管理员'; break
        //       case 4: text = '管理员'; break
        //       default: text = 'unknown role'
        //     }
        //     return h('div', text)
        //   }
        // },
        {
          title: '操作时间',
          key: 'time',
          sortable: 'custom'
        },
        {
          title: '运行状态',
          key: 'status',
          align: 'center',
          render: (h, params) => {
            let status = 'success'
            let label = '正常'
            if (params.row.status === 'err') {
              status = 'error'
              label = '错误'
            } else if (params.row.status === 'panic') {
              status = 'error'
              label = '异常'
            }
            return h('div', [
              h('Tag', {
                props: {
                  color: status
                }
              }, label)
            ])
          }
        },
        // {
        //   title: '状态',
        //   key: 'status',
        //   tooltip: true,
        //   sortable: 'custom',
        //   align: 'center'
        // },
        // {
        //   title: '操作者IP',
        //   key: 'user_ip'
        // },
        {
          title: '详情',
          key: 'param',
          tooltip: true
        }
      ]
    }
  },
  mounted () {
    this.getAllData()
    // this.getLogList()
    this.getAllData1()
  },
  watch: {
    logFile: {
      deep: true,
      handler: function (newVal, oldVal) {
        console.log(this.logFile)
        // this.termId = this.courseTerm
        // for (var term1 of this.listTerm) {
        //   if (term1.id === this.courseTerm) { this.term = term1.name }
        // }
        this.getAllData1()
      }
    }
  },
  methods: {
    getAllData () {
      this.load = true
      this.$http.get('/admin/getLogs')
        .then(res => {
          console.log(res)
          this.load = false
          if (res.data.code === 1001) {
            this.allData = res.data.data
            this.allData.sort(this.sortById)
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    getAllData1 () {
      this.load = true
      axios.post(go.gourl + '/getLog', {logFile: this.logFile}, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          console.log(res)
          this.load = false
          if (res.data.code === 1001) {
            this.allData1 = res.data.data
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    getLogList () {
      this.load = true
      axios.post(go.gourl + '/listLog', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          console.log(res)
          this.load = false
          if (res.data.code === 1001) {
            this.listLog = res.data.data
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    refreshData () {
      this.getAllData()
    },
    sortById (a, b) {
      return b.id - a.id
    }
  }
}
</script>

<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>

</style>
