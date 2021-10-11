<template>
  <div>
    <NavBar :breadlist="breadlist">
      <div slot="navbarContent">
        <LogContainer :showModalCheckVM.sync="showModalLogVM" :podName="podName" :namespace="port"></LogContainer>
        <CheckContainer :showModalCheckVM.sync="showModalCheckCon" :podName="podName" :url="url"></CheckContainer>
        <AddProjectPod @refresh="refreshData" :showModalAddVM.sync="showModalAddVM" :expId="expId" :projectId="projectId"></AddProjectPod>
        <DeployProject @refresh="refreshData" :showModalAddVM.sync="showModalDeployProject" :expId="expId" :projectId="projectId"></DeployProject>
        <div style="padding-left: 10px" >
<!--        <Button type="primary" @click="showModalAddVM = true">-->
<!--          <Icon type="md-add-circle" class="addIcon"/>-->
<!--          <span style="font-size: 14px">新增项目服务</span>-->
<!--        </Button>-->
        <Button type="primary" @click="showModalDeployProject = true">
          <Icon type="md-add-circle" class="addIcon"/>
          <span style="font-size: 14px">部署项目</span>
        </Button>
        </div>
        <div v-for="(item, index) in showData" :key="item.id" :value="item.name" :label="index">
        <CardContent>
          <span style="font-size: 30px">Service</span>
          <br>
          <br>
          <TableComponent1 :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="getService(item)">
          </TableComponent1>
          <br>
          <br>
          <span style="font-size: 30px">Pod</span>
          <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns" :allDataFromFather="getPod(item)">
          </TableComponent>
        </CardContent>
        </div>
      </div>
    </NavBar>
  </div>
</template>

<script>
import NavBar from '../../components/NavBar'
import CardContent from '../../components/CardContent'
import TableComponent from '../../components/TableComponent'
import TableComponent1 from '../../components/TableComponent1'
import AddContainer from '../../components/teacher/container/addContainer'
import ModifyContainer from '../../components/teacher/container/modifyContainer'
import LogContainer from '../../components/teacher/container/logContainer'
import CheckContainer from '../../components/teacher/container/checkContainer'
import AddSharePod from '../../components/teacher/container/addSharePod'
import AddProjectPod from '../../components/teacher/container/addProjectPod'
import DeployProject from '../../components/teacher/container/deployProject'
import axios from 'axios'
import go from '../../assets/Global'
export default {
  name: 'projectContainer',
  components: {AddContainer, TableComponent, CardContent, NavBar, ModifyContainer, LogContainer, CheckContainer, AddSharePod, AddProjectPod, DeployProject, TableComponent1},
  data () {
    return {
      url: [],
      podName: '',
      port: '',
      consoleUrl: '',
      showModalCheckVM: false,
      showModalCheckCon: false,
      showModalLogVM: false,
      showModalAddSP: false,
      showModalDeployProject: false,
      role: '',
      operOS: '',
      userId: '',
      operIP: '',
      operName: '',
      breadlist: [
        {name: '容器管理', path: '/student/studentProjectManagement'}
      ],
      columns1: [
        {
          title: '服务名称',
          key: 'name'
        },
        {
          title: '服务端口',
          key: 'ports'
        }
        // {
        //   title: '容器状态',
        //   key: 'status'
        // },
        // {
        //   title: '容器端口',
        //   key: 'port'
        // }
      ],
      columns: [
        {
          title: '容器名称',
          key: 'podname'
        },
        {
          title: '镜像名称',
          key: 'image'
        },
        {
          title: '容器状态',
          key: 'status'
        },
        {
          title: '容器端口',
          key: 'port'
        },
        // {
        //   title: '总内存(M)',
        //   key: 'totalMem'
        // },
        {
          title: '已用内存(M)',
          key: 'usedMem'
        },
        // {
        //   title: '内存使用率',
        //   key: 'memoryUsage',
        //   render: (h, params) => {
        //     let usage = Math.round(params.row.usedMem / params.row.totalMem * 10000) / 100.00
        //     let color = '#22C688'
        //     if (usage > 75) {
        //       color = '#FF7875'
        //     }
        //     return h('div', {
        //       style: {
        //         display: 'flex'
        //       }
        //     }, [
        //       h('i-circle', {
        //         props: {
        //           percent: usage,
        //           size: 24,
        //           'stroke-color': color,
        //           'trail-color': '#E8E8E8',
        //           'stroke-width': 20,
        //           'trail-width': 20,
        //           'stroke-linecap': 'square'
        //         }
        //       }),
        //       h('div', {}, usage + '%')]
        //     )
        //   }
        // },
        // {
        //   title: '总CPU(MHz)',
        //   key: 'totalCPU'
        // },
        {
          title: '已用CPU(MHz)',
          key: 'usedCPU'
        },
        // {
        //   title: 'CPU使用率',
        //   key: 'cpuUsage',
        //   render: (h, params) => {
        //     let usage = Math.round(params.row.usedCPU / params.row.totalCPU * 10000) / 100.00
        //     let color = '#22C688'
        //     if (usage > 75) {
        //       color = '#FF7875'
        //     }
        //     return h('div', {
        //       style: {
        //         display: 'flex'
        //       }
        //     }, [
        //       h('i-circle', {
        //         props: {
        //           percent: usage,
        //           size: 24,
        //           'stroke-color': color,
        //           'trail-color': '#E8E8E8',
        //           'stroke-width': 20,
        //           'trail-width': 20,
        //           'stroke-linecap': 'square'
        //         }
        //       }),
        //       h('div', {}, usage + '%')]
        //     )
        //   }
        // },
        {
          title: '容器控制台',
          key: 'cmd',
          align: 'center',
          width: '100',
          render: (h, params) => {
            return h('a', {
              attrs: { href: params.row.cmd, target: '_blank', title: params.row.cmd }
            }, '访问链接')
          }
        },
        {
          title: '操作',
          key: 'operation',
          align: 'center',
          width: 300,
          render: (h, params) => {
            return h('div', [
              // TODO: 根据虚拟机类型判断是否有查看（直接vnc网页登录虚拟机）功能
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '5px',
                  marginBottom: '5px'
                },
                on: {
                  click: () => {
                    this.podName = params.row.podname
                    this.url = params.row.url
                    this.showModalCheckCon = true
                    console.log(this.showModalCheckCon)
                  }
                }
              }, '查看容器链接'),
              h('Button', {
                props: {
                  type: 'info'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '5px',
                  marginBottom: '5px'
                },
                on: {
                  click: () => {
                    this.podName = params.row.name
                    this.port = params.row.namespace
                    this.showModalLogVM = true
                    console.log(this.showModalCheckVM)
                  }
                }
              }, '查看日志')
            ])
          }
        }
      ],
      showData: [],
      showData1: [],
      wait: false,
      load: true,
      showModalAddVM: false,
      projectId: '',
      expId: ''
    }
  },
  created () {
    this.role = this.$cookie.get('role')
    this.projectId = String(this.$route.params.projectId)
    this.expId = String(this.$route.params.expId)
    if (this.projectId !== undefined && this.projectId !== 'undefined') {
      localStorage.setItem('pid', this.projectId)
      localStorage.setItem('eid', this.expId)
    }
    console.log(this.projectId)
    this.getData()
    this.refreshData()
  },
  methods: {
    getPod (item) {
      return item.pods
    },
    getService (item) {
      let svc = item.service
      svc.name = svc.metadata.name
      svc.ports = []
      svc.spec.ports.forEach(v => {
        let port = v.port + ' : ' + v.nodePort + ' ' + v.protocol
        svc.ports.push(port)
      })
      let svcs = []
      svcs.push(svc)
      return svcs
    },
    getData () {
      this.projectId = localStorage.getItem('pid')
      this.expId = localStorage.getItem('eid')
      this.load = true
      let params = {
        expId: this.expId, projectId: this.projectId
      }
      axios.post(go.gourl + '/listPContainer',
        params, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$cookie.get('token')
          }
        }).then(res => {
        this.load = false
        if (res.data.code === 1001) {
          this.showData = res.data.data
          this.showData.forEach(item => {
            if (item.pods === null) {
              return true
            }
            item.pods.forEach(it => {
              it.usedCPU = it.info.usedCPU
              it.totalCPU = it.info.totalCPU
              it.usedMem = it.info.usedMem
              it.totalMem = it.info.totalMem
            })
          })
          console.log(this.showData)
        } else {
          this.$Message.error(res.data.msg)
        }
      }).catch(res => {
        console.log(res)
      })
      this.load = false
    },
    refreshData () {
      this.getData()
    }
  }
}
</script>
<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>

</style>
