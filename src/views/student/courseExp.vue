<template>
  <div>
    <NavBar :breadlist="breadlist">
      <div slot="navbarContent">
        <AddExpProject :showModalAddExperiment.sync="showModalAddProject" :info="info1"></AddExpProject>
<!--        <div style="margin-left: 10px;">-->
<!--          <Button @click="getCourse" size="large">-->
<!--            <Icon type="md-refresh"></Icon>-->
<!--            刷新课程-->
<!--          </Button>-->
<!--        </div>-->
<!--        <div class="card-list-box">-->
<!--          <Card class="card-list-null" v-if="courselist.length === 0">没有课程</Card>-->
<!--          <Card class="card-list-null" v-else v-for="(item, index) in courselist" :key="index">-->
<!--            <div class="card-list-cover" @click="changeCourseShow(item)">-->
<!--              <img class="back" src="../../assets/img/course_back.png" />-->
<!--&lt;!&ndash;              <img class="title" src="../../assets/img/course_title.png"/>&ndash;&gt;-->
<!--              <p class="course-name">{{item.name}}</p>-->
<!--              <p class="teacher">{{item.teacher}}</p>-->
<!--            </div>-->
<!--          </Card>-->
<!--        </div>-->
        <CardContent :title="titleName">
          <TableComponent @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="expData">
            <div style="color: #9ea7b4; font-size: medium;">{{this.titleTeacher}}</div>
          </TableComponent>
        </CardContent>
<!--        <div class="card-list-box">-->
<!--          <Card class="card-list-null" v-if="courselist.length === 0">没有课程</Card>-->
<!--          <Card class="card-list" v-else v-for="(item, index) in courselist" :title="item.courseName" :key="index">-->
<!--            <div @click="changeCourseShow(item)" >-->
<!--              <Row>-->
<!--                <Col class="card-list-block">-->
<!--                  <label>教师</label>-->
<!--                  <Span>{{item.teacher}}</Span>-->
<!--                </Col>-->
<!--              </Row>-->
<!--              <Row>-->
<!--                <Col class="card-list-block">-->
<!--                  <label>学期</label>-->
<!--                  <Span>{{item.term}}</Span>-->
<!--                </Col>-->
<!--              </Row>-->
<!--              <Row>-->
<!--                <Col class="card-list-block">-->
<!--                  <label>实验数</label>-->
<!--                  <Span>{{item.expCount}}</Span>-->
<!--                </Col>-->
<!--              </Row>-->
<!--            </div>-->
<!--          </Card>-->
<!--        </div>-->
      </div>
    </NavBar>
  </div>
</template>

<script>
import NavBar from '../../components/NavBar'
import CardContent from '../../components/CardContent'
import TableComponent from '../../components/TableComponent'
import AddExpProject from '../../components/teacher/project/addExpProject'
export default {
  name: 'courseExp',
  components: {TableComponent, CardContent, NavBar, AddExpProject},
  data () {
    return {
      breadlist: [
        {name: '实验信息', path: '/student/experiments'},
        {name: '课程实验', path: '/course/experiments'}
      ],
      columns1: [
        {
          title: '序号',
          type: 'index',
          width: 60,
          align: 'center'
        },
        // {
        //   title: '学期',
        //   key: 'term'
        // },
        {
          title: '实验名',
          key: 'name'
        },
        {
          title: '开始时间',
          key: 'start_time',
          sortable: true
        },
        {
          title: '结束时间',
          key: 'end_time',
          sortable: true
        },
        {
          title: '延期时间',
          key: 'deadline',
          sortable: true
        },
        // {
        //   title: '提交情况',
        //   key: 'status',
        //   align: 'center',
        //   render: (h, params) => {
        //     let status = '未提交'
        //     if (params.row.status) {
        //       status = '已提交'
        //     }
        //     return h('div', {}, status)
        //   },
        //   filters: [
        //     {
        //       label: '未完成实验'
        //     }
        //   ],
        //   filterMultiple: false,
        //   filterMethod (value, row) {
        //     return row.status === false
        //   }
        // },
        {
          title: '操作',
          key: 'oper',
          width: 250,
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
                    this.$store.commit('setParamsMessage',
                      {
                        id: params.row.id,
                        name: params.row.name,
                        courseName: this.course.courseName,
                        teacher: this.course.teacher,
                        courseId: this.course.courseId
                      })
                    this.$router.push(
                      {
                        name: 'expDetails'
                      })
                  }
                }
              }, '详情'),
              h('Button', {
                props: {
                  type: 'info',
                  disabled: !params.row.peer_assessment_start
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$store.commit('setParamsMessage',
                      {
                        id: params.row.id,
                        courseName: this.course.courseName,
                        teacher: this.course.teacher,
                        courseId: this.course.courseId
                      })
                    this.$router.push(
                      {
                        name: 'expPeer'
                      })
                  }
                }
              }, '互评'),
              h('Button', {
                props: {
                  type: 'info',
                  disabled: !params.row.peer_assessment_start
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    console.log(this.course.courseName)
                    this.$store.commit('setParamsMessage',
                      {
                        id: params.row.id,
                        name: params.row.name,
                        courseName: this.course.courseName,
                        teacher: this.course.teacher,
                        courseId: this.course.courseId
                      })
                    this.$router.push(
                      {
                        name: 'peerDetails'
                      })
                  }
                }
              }, '查看互评'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    console.log(this.course)
                    let info1 = {
                      expId: params.row.id,
                      expName: params.row.name,
                      courseName: this.course.courseName,
                      teacher: this.course.teacher,
                      courseId: this.course.courseId
                    }
                    this.info1 = info1
                    this.showModalAddProject = true
                  }
                }
              }, '创建项目'),
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
                        name: 'studentProjectManagement'
                      })
                  }
                }
              }, '查看项目列表')
            ])
          }
        }
      ],
      expData: [],
      course: JSON.parse(localStorage.getItem('paramsMessage')),
      courselist: [],
      term: '',
      titleName: '',
      titleTeacher: '',
      termId: 0,
      showModalAddProject: false,
      info1: null
    }
  },
  mounted () {
    localStorage.setItem('courseId', this.course.courseId)
    console.log(this.course.courseId)
    console.log('hello')
    this.getCurrentTermFirst()
  },
  methods: {
    getCurrentTermFirst () {
      this.$http.get('/getNowTerm')
        .then(res => {
          console.log(res.data.data)
          this.termId = res.data.data.id
          this.term = res.data.data.name
          this.titleName = this.course.courseName
          this.titleTeacher = this.course.teacher
          this.getExp(this.course.courseId)
          this.getCourse()
        })
        .catch(err => {
          console.log(err)
        })
    },
    getExp (courseid) {
      this.$http.get('/student/getExperimentByCourseId', {params: {courseId: courseid}})
        .then(res => {
          console.log(res)
          if (res.data.code === 1001) {
            this.expData = res.data.data
            // this.showData = this.expData.slice(0, this.pageSize)
            // this.totalExpCount = res.data.data.length
            // console.log(this.courseData)
            console.log(res.data.data)
          } else {
            this.$Message.error(res.data.msg)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    refreshData () {
      this.getExp()
    },
    getCourse () {
      this.$http.get('/getCourseListByStuIdAndTerm', {params: {term: this.term}})
        .then(res => {
          if (res.data.code === 1001) {
            this.courselist = res.data.data
          } else {
            this.courselist = []
            this.$Message.error(res.data.msg)
          }
        }).catch(err => {
          console.log(err)
        })
    },
    changeCourseShow (item) {
      this.titleName = item.name
      this.titleTeacher = item.teacher
      this.$store.commit('setParamsMessage', {
        courseId: item.id,
        courseName: item.name,
        teacher: item.teacher,
        department: item.department
      })
      this.course = JSON.parse(localStorage.getItem('paramsMessage'))
      this.getExp(item.id)
    }
  }
}
</script>
<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>
  .card-list-box{
    display: flex;
    flex-wrap: wrap;
    margin-bottom: 10px;
  }
  .card-list{
    margin: 10px 5px 10px 10px;
    width: 300px;
  }
  .card-list-block label{
    font-weight: bold;
    margin-right: 10px;
  }
  .card-list-null{
    font-size: large;
    text-align: center;
    width: 230px;
    margin: 10px 5px 10px 10px;
  }
  .card-list-cover{
    text-align: center;
    overflow: hidden;
  }
  .card-list-cover .back{
    width: 100%;
    height: 90px;
    opacity: .4;
  }
  .card-list-cover .teacher{
    bottom: 20px;
    position: absolute;
    /*color: #9ea7b4;*/
    font-size: large;
    right: 20px;
  }
  .card-list-cover .course-name{
    top: 20px;
    padding-left: 15px;
    position: absolute;
    font-size: xx-large;
  }
  .cover{
    position: relative;
    width: 100%;
  }
  .cover .back{
    margin-top: 20px;
    width: 100%;
    opacity: .5;
  }
  .cover .title{
    position: absolute;
    height:10px;
    top:20px;
    left:20px;
  }
  .cover .course-name{
    border-top: 2px #3a90b7 solid;
    border-bottom: 2px #3a90b7 solid;
    position: absolute;
    top: 52px;
    left:50%;
    transform: translateX(-50%);
    width: max-content;
    text-align: center;
    font-size: 26px;
    padding: 5px 0;
    font-family: 黑体,sans-serif;
  }
  .cover .teacher-name{
    top:126px;
    position: absolute;
    left: 180px;
    font-size: 16px;
  }
</style>
