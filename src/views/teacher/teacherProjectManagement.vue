<template>
  <div>
    <NavBar :username="this.$cookie.get('username')" infopage='/teacher/info' :breadlist="breadList">
      <div slot="navbarContent">
        <CardContent>
          <ListMember :showModalCheckStudent.sync="showModalCheckStudent" :courseID="operCourseID" :courseName="operCourseName" :operLevel="currentType"></ListMember>
          <CheckProject :showModalCheckStudent.sync="showModalCheckProject" :courseID="operCourseID" :courseName="operCourseName" :operLevel="currentType"></CheckProject>
          <TableComponent :loading="loading" @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="allData">
<!--            <a :href="registerUrl" target="_blank">如果没有注册代码平台，请先注册</a>-->
            <br>
            <br>
            <div style="margin-left: 10px;">
              <Row>
                切换课程:
                <Select v-model="courseTerm" style="width:300px">
                  <Option v-for="(item) in listTerm" :key="item.id" :value="item.id" :label="item.name">
                    <span>{{item.name}}</span>
                  </Option>
                </Select>
              </Row>
            </div>
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
import CheckProject from '../../components/teacher/project/checkProject'
import AddMember from '../../components/teacher/project/addMember'
import ChangeProject from '../../components/teacher/project/changeProject'
import axios from 'axios'
import go from '../../assets/Global'
export default {
  name: 'teacherProjectManagement',
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
    BatchAddStudentsModal,
    CheckProject
  },
  data () {
    return {
      allExp: [],
      currentType: 'all',
      highLightRow: true,
      loading: true,
      registerUrl: go.bugiturl1 + '/user/sign_up',
      breadList: [{name: '项目管理', path: '/teacher/teacherProjectManagement'}], // 面包屑
      showModalAddCourse: false,
      showModalAddHomework: false,
      showModalAddExperiment: false,
      showModalChangeCourse: false,
      showModalDeleteCourse: false,
      showModalCheckStudent: false,
      showModalCheckProject: false,
      showModalBatchAddStudents: false,
      allData1: [],
      operationTitle: '',
      operationContent: '',
      // collegeUrl: '/teacher/listCollege',
      termUrl: '/teacher/listTerm',
      showModalOperationRes: false,
      operCourseID: '',
      operCourseName: '',
      operCourseTerm: -1,
      operCourseDepartment: '',
      allData: [],
      listTerm: [],
      courseTerm: -1,
      // listCollege: [],
      // 请求数据的url
      allDataUrl: go.bugiturl + '/user/projects/by/courses',
      columns1: [
        // {
        //   // 仅使用type:index可以达到显示行号的目的，但是将TableComponent组件中的item转为对象后发现index并不在属性中，做搜索时不太方便
        //   // 如果希望表格显示序号，请务必把序号这一列的key命名为index，如果有这一列则搜索的时候会搜索序号，没有就不搜索
        //   title: '序号',
        //   key: 'index',
        //   sortable: 'custom',
        //   sortType: 'number'
        // },
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
        // {
        //   title: '运行状态',
        //   key: 'status',
        //   width: 120,
        //   render: (h, params) => {
        //     let status = 'success'
        //     let label = '运行中'
        //     if (params.row.status === 0) {
        //       status = '#000000'
        //       label = '开发中'
        //     }
        //     return h('div', [
        //       h('Tag', {
        //         props: {
        //           color: status
        //         }
        //       }, label)
        //     ])
        //   }
        // },
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
    axios.post(go.gourl + '/getCourse', {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': this.$cookie.get('token')
      }
    })
      .then(res => {
        if (res.data.code === 1001) {
          this.listTerm = res.data.data
          // this.listTerm.push({id: -1, name: '全部课程'})
          console.log(this.listTerm)
        } else {
          this.$Message.error(res.data.msg)
        }
      })
      .catch(err => {
        console.log(err)
      })
    this.getAllData()
  },
  watch: {
    courseTerm: {
      deep: true,
      handler: function (newVal, oldVal) {
        this.termId = this.courseTerm
        this.allData = []
        for (var term1 of this.listTerm) {
          if (term1.id === this.courseTerm) { this.term = term1.id }
        }
        console.log(this.term)
        for (var item of this.allData1) {
          if (item.course_id === this.term) {
            this.allData.push(item)
          }
        }
        console.log(this.allData)
      }
    }
  },
  computed: {
  },
  methods: {
    showOperationRes (title, content) {
      this.operationTitle = title
      this.operationContent = content
      this.showModalOperationRes = true
    },
    refreshData () {
      this.getAllData()
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
          this.loading = false
          this.allData = res.data
          this.allData1 = res.data
          console.log(this.allData)
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
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
