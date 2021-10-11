<template>
  <div>
    <NavBar :username="this.$cookie.get('username')" infopage='/teacher/info' :breadlist="breadList">
      <div slot="navbarContent">
        <CardContent>
          <AddHomeworkModal @refresh="refreshData" :showModalAddHomework.sync="showModalAddHomework" :selectCourseID="courseID" :selectCourseName="courseName"></AddHomeworkModal>
          <AddExperimentModal @refresh="refreshData" :showModalAddExperiment.sync="showModalAddExperiment" :selectCourseID="courseID" :selectCourseName="courseName"></AddExperimentModal>
          <ChangeWorkingModal @refresh="refreshData" :showModalChangeWorking.sync="showModalChangeWorking" :workingParam="operWorkingParam" :selectCourseName="courseName"></ChangeWorkingModal>
          <AddVMModal :showModalAddVM.sync="showModalAddVM" :expID="operExpID" :expName="operExpName" :expApplyId="operExpApplyId"></AddVMModal>
          <DeleteExpModal @refresh="refreshData" :showModalDeleteExp.sync="showModalDeleteExp" :expID="operExpID" :expName="operExpName" :expType="operExpType"></DeleteExpModal>
          <TableComponent :loading="loading" @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="allData">
            <Button type="primary" @click="showModalAddExperiment = true">
              <span style="font-size: 14px">新增实验</span>
            </Button>
            <Button type="primary" style="margin-left:20px;" @click="showModalAddHomework = true">
              <span style="font-size: 14px">新增作业</span>
            </Button>
          </TableComponent>
        </CardContent>
      </div>
    </NavBar>
  </div>
</template>
<script>
import NavBar from '../../components/NavBar'
import TableComponent from '../../components/TableComponent'
import CardContent from '../../components/CardContent'
import AddHomeworkModal from '../../components/teacher/course/course_addHomeworkModal'
import AddExperimentModal from '../../components/teacher/course/course_addExperimentModal'
import AddVMModal from '../../components/teacher/vm/vm_addVMModal'
// import ApplyVMModal from '../../components/teacher/courseDetail_applyVMModal'
import DeleteExpModal from '../../components/teacher/courseDetail/courseDetail_deleteExpModal'
import ChangeWorkingModal from '../../components/teacher/courseDetail/courseDetail_changeWorkingModal'
import ChangePasswordModal from '../../components/teacher/courseDetail/courseDetail_changePasswordModal'

export default {
  name: 'courseDetail',
  components: {
    NavBar,
    TableComponent,
    CardContent,
    AddHomeworkModal,
    AddExperimentModal,
    AddVMModal,
    // ApplyVMModal,
    DeleteExpModal,
    ChangeWorkingModal,
    ChangePasswordModal
  },
  data () {
    return {
      courseName: '',
      courseID: '',
      operExpName: '',
      operExpID: -1,
      operExpType: '',
      operExpApplyId: '',
      operExpVmName: '',
      loading: true,
      highLightRow: true,
      showModalAddVM: false,
      showModalChangeWorking: false,
      showModalDeleteExp: false,
      breadList: [], // 面包屑
      showModalAddHomework: false,
      showModalAddExperiment: false,
      operWorkingParam: {},
      allData: [],
      // 请求数据的url
      allDataUrl: 'teacher/getExperiment',
      columns1: [
        {
          title: '虚拟机申请编号',
          key: 'vm_apply_id',
          width: 1,
          render: (h, params) => {
            return h('span', {
              style: {
                display: 'none'
              }
            })
          }
        },
        {
          title: '虚拟机名',
          key: 'vm_name',
          width: 1,
          render: (h, params) => {
            return h('span', {
              style: {
                display: 'none'
              }
            })
          }
        },
        {
          // 仅使用type:index可以达到显示行号的目的，但是将TableComponent组件中的item转为对象后发现index并不在属性中，做搜索时不太方便
          // 如果希望表格显示序号，请务必把序号这一列的key命名为index，如果有这一列则搜索的时候会搜索序号，没有就不搜索
          title: '序号',
          key: 'index',
          width: 100,
          sortable: 'number'
        },
        {
          title: '名称',
          key: 'name'
        },
        {
          title: '类型',
          key: 'type',
          width: 80
        },
        {
          title: '开始时间',
          key: 'start_time'
        },
        {
          title: '结束时间',
          key: 'end_time'
        },
        {
          title: '作业截止时间',
          key: 'deadline'
        },
        {
          title: '作业提交情况',
          key: 'assignment'
        },
        {
          title: '互评情况',
          key: 'peerAssessment'
        },
        {
          title: '实验虚拟机',
          key: 'vms'
        },
        {
          title: '操作',
          key: 'oper',
          width: 530,
          align: 'center',
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'info',
                  disabled: params.row.vms === '已删除'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px',
                  marginLeft: '5px',
                  border: '1px solid #50D6A3',
                  color: '#50D6A3'
                },
                on: {
                  click: () => {
                    this.operExpID = Number(params.row.id)
                    this.operExpName = params.row.name
                    this.operExpApplyId = params.row.vm_apply_id
                    this.operExpVmName = params.row.vm_name
                    this.showModalAddVM = true
                  }
                }
              }, '申请虚拟机'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px'
                },
                on: {
                  click: () => {
                    sessionStorage.setItem('expDetail_breadList', JSON.stringify(this.breadList))
                    sessionStorage.setItem('expDetail_expID', params.row.id)
                    sessionStorage.setItem('expDetail_expName', params.row.name)
                    sessionStorage.setItem('expDetail_expType', params.row.type)
                    sessionStorage.setItem('expDetail_expStartTime', params.row.start_time)
                    sessionStorage.setItem('expDetail_expEndTime', params.row.end_time)
                    this.$router.push({
                      path: '/teacher/expDetail'
                    })
                  }
                }
              }, '详情'),
              // h('Button', {
              //   props: {
              //     type: 'primary'
              //   },
              //   style: {
              //     marginRight: '5px',
              //     marginTop: '4px',
              //     marginBottom: '4px'
              //   },
              //   on: {
              //     click: () => {
              //       sessionStorage.setItem('expDetail_breadList', JSON.stringify(this.breadList))
              //       sessionStorage.setItem('expDetail_expID', params.row.id)
              //       sessionStorage.setItem('expDetail_expName', params.row.name)
              //       sessionStorage.setItem('expDetail_expType', params.row.type)
              //       sessionStorage.setItem('expDetail_expStartTime', params.row.start_time)
              //       sessionStorage.setItem('expDetail_expEndTime', params.row.end_time)
              //       this.$router.push({
              //         path: '/teacher/stuDetail'
              //       })
              //     }
              //   }
              // }, '学生看到的页面'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px'
                },
                on: {
                  click: () => {
                    this.operWorkingParam = params.row
                    this.showModalChangeWorking = true
                    sessionStorage.setItem('expDetail_expName', params.row.name)
                    sessionStorage.setItem('course_Id', params.course_id)
                  }
                }
              }, '修改'),
              h('Button', {
                props: {
                  type: 'info',
                  disabled: params.row.vms === '已删除' || params.row.vms === '无'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px'
                },
                on: {
                  click: () => {
                    sessionStorage.setItem('experimentalVM_fromBreadList', JSON.stringify(this.breadList))
                    sessionStorage.setItem('experimentalVM_expID', String(params.row.id))
                    sessionStorage.setItem('experimentalVM_expName', String(params.row.name))
                    sessionStorage.setItem('experimentalVM_expType', String(params.row.type))
                    sessionStorage.setItem('experimentalVM_VmName', String(params.row.vm_name))
                    sessionStorage.setItem('experimentalVM_defaultVMPWD', String(params.row.vm_passwd))
                    this.$router.push({
                      path: '/teacher/experimentalVM'
                    })
                  }
                }
              }, '实验虚拟机管理'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px'
                },
                on: {
                  click: () => {
                    sessionStorage.setItem('checkHomework_breadList', JSON.stringify(this.breadList))
                    sessionStorage.setItem('checkHomework_expID', String(params.row.id))
                    sessionStorage.setItem('checkHomework_expName', String(params.row.name))
                    sessionStorage.setItem('checkHomework_enablePeer', String(params.row.is_peer_assessment))
                    sessionStorage.setItem('checkHomework_peerStarted', String(params.row.peerStarted))
                    this.$router.push({
                      path: '/teacher/homework'
                    })
                  }
                }
              }, '查看作业'),
              h('Button', {
                props: {
                  type: 'error'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '4px',
                  marginBottom: '4px'
                },
                on: {
                  click: () => {
                    this.showModalDeleteExp = true
                    this.operExpName = params.row.name
                    this.operExpID = Number(params.row.id)
                    this.operExpType = params.row.type
                  }
                }
              }, '删除')
            ])
          }
        }
      ]
    }
  },
  mounted () {
    this.courseID = sessionStorage.getItem('courseDetail_courseID')
    this.courseName = sessionStorage.getItem('courseDetail_courseName')
    this.breadList = []
    this.breadList.push({name: '课程管理', path: '/teacher/course'})
    this.breadList.push({name: '课程' + this.courseName + '详情', path: this.$route.fullPath})
    this.loading = true
    this.getAllData()
    // this.allData = [{'id': '17', 'course_id': '1', 'name': 'testpeer', 'type': '作业', 'detail': 'ttt', 'resource': '', 'create_time': '2019/9/28 15:28:51', 'start_time': '2019/9/23 00:00:00', 'end_time': '2019/9/23 21:05:00', 'deadline': '2019/9/23 21:05:00', 'vm_status': '0', 'vm_name': 'test_peer_due_time', 'vm_apply_id': '56', 'vm_passwd': '', 'is_peer_assessment': 'True', 'peer_assessment_deadline': '2019-10-10 00:00:00', 'appeal_deadline': '2019-10-11 00:00:00', 'peer_assessment_rules': 't', 'peer_assessment_start': 'False', 'teacher_name': 'wwwww', 'course_name': 'zhj', 'term_id': '7', 'term_name': '2019 秋', 'assignment': '10/2', 'peerAssessment': '', 'vms': '0', 'peerStarted': 'False'}]
    // this.loading = false
  },
  methods: {
    refreshData () {
      this.loading = true
      this.getAllData()
    },
    getAllData: function () {
      if (this.courseID === '') {
        return
      }
      this.$http.get(this.allDataUrl, {
        params: {
          course_id: this.courseID
        }
      })
        .then(res => {
          this.loading = false
          if (res.data.code === 1001) {
            console.log(res.data.data)
            this.allData = res.data.data
            this.allData.forEach(function (item) {
              if (item.is_peer_assessment === 'True' && item.peerStarted === 'True') {
                item.peerAssessment = '互评已开始'
              }
              if (item.is_peer_assessment === 'True' && item.peerStarted === 'False') {
                item.peerAssessment = '互评未开始'
              }
              if (item.type === 'False') {
                item.type = '作业'
              } else {
                item.type = '实验'
              }
            })
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    }
  }
}
</script>

<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>
  .addIcon{
    padding-right:5px;
    padding-bottom: 3px;
  }
  .studentButton{
    border-color: cornflowerblue;
    background-color: cornflowerblue;
  }
</style>
