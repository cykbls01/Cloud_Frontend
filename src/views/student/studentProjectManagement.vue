<template>
  <div>
    <NavBar :username="this.$cookie.get('username')" infopage='/teacher/info' :breadlist="breadList">
      <div slot="navbarContent">
        <CardContent>
          <ListMember :showModalCheckStudent.sync="showModalCheckStudent" :courseID="operCourseID" :courseName="operCourseName" :operLevel="currentType"></ListMember>
          <CheckProject :showModalCheckStudent.sync="showModalCheckProject" :courseID="operCourseID" :courseName="operCourseName" :operLevel="currentType"></CheckProject>
          <TableComponent :loading="loading" @refresh="refreshData" :columnsFromFather="columns1" :allDataFromFather="showData">
<!--            <a :href="registerUrl" target="_blank">如果没有注册代码平台，请先注册</a>-->
            <br>
            <br>
            <ButtonGroup>
              <Button :type="currentType === 'student' ? 'primary' : 'default'" @click="currentType = 'student'"><Icon type="md-information-circle" class="buttonIcon"/>参与项目</Button>
              <Button :type="currentType === 'assistant' ? 'primary' : 'default'" @click="currentType = 'assistant'"><Icon type="md-help-circle" class="buttonIcon"/>课程项目</Button>
            </ButtonGroup>
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
import AddProject from '../../components/teacher/project/addProject'
import AddHomeworkModal from '../../components/teacher/course/course_addHomeworkModal'
import AddExperimentModal from '../../components/teacher/course/course_addExperimentModal'
import ChangeCourseModal from '../../components/teacher/course/course_changeCourseModal'
import DeleteCourseModal from '../../components/teacher/course/course_deleteCourseModal'
import OperationResModal from '../../components/teacher/operationResModal'
import CheckStudentModal from '../../components/teacher/course/course_checkStudentModal'
import BatchAddStudentsModal from '../../components/teacher/course/course_batchAddStudentsModal'
import DeleteProject from '../../components/teacher/project/deleteProject'
import ListMember from '../../components/teacher/project/listMember'
import AddMember from '../../components/teacher/project/addMember'
import ChangeProject from '../../components/teacher/project/changeProject'
import CheckProject from '../../components/teacher/project/checkProject'
import axios from 'axios'
import go from '../../assets/Global'
export default {
  name: 'studentProjectManagement',
  components: {
    ChangeProject,
    AddMember,
    ListMember,
    DeleteProject,
    NavBar,
    TableComponent,
    CardContent,
    AddProject,
    AddHomeworkModal,
    AddExperimentModal,
    ChangeCourseModal,
    DeleteCourseModal,
    OperationResModal,
    CheckStudentModal,
    CheckProject,
    BatchAddStudentsModal
  },
  data () {
    return {
      showdata: [],
      idList: ' ',
      allExp: [],
      registerUrl: go.bugiturl1 + '/user/sign_up',
      currentType: 'assistant',
      highLightRow: true,
      loading: false,
      breadList: [{name: '项目管理', path: '/student/studentProjectManagement'}], // 面包屑
      showModalAddCourse: false,
      showModalAddHomework: false,
      showModalAddExperiment: false,
      showModalCheckProject: false,
      showModalChangeCourse: false,
      showModalDeleteCourse: false,
      showModalCheckStudent: false,
      showModalBatchAddStudents: false,
      operationTitle: '',
      operationContent: '',
      termUrl: '/teacher/listTerm',
      showModalOperationRes: false,
      operCourseID: '',
      operCourseName: '',
      operCourseTerm: -1,
      operCourseDepartment: '',
      allData: [],
      listTerm: [],
      allDataUrl: go.bugiturl + '/user/projects',
      columns1: [
        {
          title: '项目名称',
          key: 'name',
          sortable: 'custom'
        },
        {
          title: '关联实验',
          key: 'exp_name',
          sortable: 'custom'
        },
        {
          title: '关联课程',
          key: 'course_name',
          sortable: 'custom'
        },
        {
          title: '运行状态',
          key: 'status',
          width: 120,
          render: (h, params) => {
            let status = 'success'
            let label = '运行中'
            if (params.row.status === 0) {
              status = '#000000'
              label = '开发中'
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
        {
          title: '操作',
          key: 'oper',
          width: 450,
          align: 'center',
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.operCourseID = String(params.row.ID)
                    this.operCourseName = params.row.name
                    this.showModalCheckStudent = true
                  }
                }
              }, '查看成员'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.operCourseID = String(params.row.ID)
                    this.operCourseName = params.row.name
                    this.showModalCheckProject = true
                  }
                }
              }, '查看部署'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '跳转项目',
                      content: '确认跳转到项目首页',
                      onOk () {
                        window.open(go.bugiturl1 + '/project/' + params.row.ID + '?Authorization=' + this.$cookie.get('token'))
                      }
                    })
                  }
                }
              }, '跳转到项目首页'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$router.push(
                      {
                        name: 'projectContainer',
                        params: {
                          expId: params.row.Sender.ID,
                          projectId: params.row.ID
                        }
                      }
                    )
                  }
                }
              }, '查看项目容器')
            ])
          }
        }
      ]
    }
  },
  // TODO:隐藏导入学生按钮，将其放到查看学生的二级页面中
  mounted () {
    this.getAllData()
    this.getAllData2()
    this.getListTerm()
    this.currentType = 'student'
  },
  computed: {
    showData: function () {
      console.log(this.currentType)
      if (this.currentType === 'student') {
        if (this.allData.project1 === null) return []
        return this.allData.project1
      } else {
        if (this.allData.project3 === null) return []
        return this.allData.project3
      }
    }
  },
  methods: {
    getListTerm () {
      this.$http.get(this.termUrl)
        .then(res => {
          if (res.data.code === 1001) {
            this.listTerm = res.data.data
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    showOperationRes (title, content) {
      this.operationTitle = title
      this.operationContent = content
      this.showModalOperationRes = true
    },
    refreshData () {
      this.getAllData()
    },
    getAllData1: function () {
      axios.post(go.gourl + '/getCourse',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.loading = false
          if (res.data.code === 1001) {
            for (var i = 0; i < res.data.data.length; i++) {
              this.idList = this.idList + res.data.data[i].id + ','
            }
            console.log(this.idList)
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    getAllData2 () {
      axios.get(go.bugiturl + '/user/projects/by/courses',
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.loading = false
          this.allData.project3 = res.data
          console.log(this.allData)
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    getAllData () {
      axios.get(this.allDataUrl,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': this.$cookie.get('token')
          }
        })
        .then(res => {
          this.allData.project1 = res.data
          console.log(this.allData)
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
        })
    }
  }
}
</script>

<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>
  .studentButton{
    border-color: cornflowerblue;
    background-color: cornflowerblue;
  }
</style>
