<template>
  <div>
    <NavBar :breadlist="breadlist">
      <div slot="navbarContent">
        <CardContent>
          <AddContainer @refresh="refreshData" :showModalAddVM.sync="showModalAddVM"></AddContainer>
          <AddSharePod @refresh="refreshData" :showModalAddVM.sync="showModalAddSP"></AddSharePod>
          <ModifyContainer :showModalCheckVM.sync="showModalCheckVM" :podName="podName" :userId="userId"></ModifyContainer>
          <LogContainer :showModalCheckVM.sync="showModalLogVM" :podName="podName" :namespace="port"></LogContainer>
          <CheckContainer :showModalCheckVM.sync="showModalCheckCon" :podName="podName" :url="url"></CheckContainer>
          <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns" :allDataFromFather="showData">
            <Button type="primary" @click="showModalAddVM = true">
              <span style="font-size: 14px">新增容器</span>
            </Button>
          </TableComponent>
        </CardContent>
        <CardContent>
          <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="showData1">
            <Button type="primary" @click="showModalAddSP = true">
              <span style="font-size: 14px">新增共享容器</span>
            </Button>
          </TableComponent>
        </CardContent>
      </div>
    </NavBar>
  </div>
</template>

<script>
import NavBar from '../../components/NavBar'
import CardContent from '../../components/CardContent'
import TableComponent from '../../components/TableComponent'
import AddContainer from '../../components/teacher/container/addContainer'
import ModifyContainer from '../../components/teacher/container/modifyContainer'
import LogContainer from '../../components/teacher/container/logContainer'
import CheckContainer from '../../components/teacher/container/checkContainer'
import AddSharePod from '../../components/teacher/container/addSharePod'
import axios from 'axios'
import go from '../../assets/Global'
export default {
  name: 'containerManagement',
  components: {AddContainer, TableComponent, CardContent, NavBar, ModifyContainer, LogContainer, CheckContainer, AddSharePod},
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
      operOS: '',
      userId: '',
      operIP: '',
      operName: '',
      breadlist: [
        {name: '容器管理', path: '/student/containerManagement'}
      ],
      columns1: [
        {
          title: '容器名称',
          key: 'pod_name'
        },
        {
          title: '到期时间',
          key: 'due_time'
        },
        {
          title: '镜像名称',
          key: 'con_name'
        },
        {
          title: '数据库名称(用户名)',
          key: 'db_name'
        },
        {
          title: '密码',
          key: 'password'
        },
        {
          title: '链接地址',
          key: 'url',
          align: 'center',
          width: '100',
          render: (h, params) => {
            return h('a', {
              attrs: { href: params.row.url, target: '_blank', title: params.row.url }
            }, '数据库链接')
          }
        },
        {
          title: '操作',
          key: 'operation',
          align: 'center',
          width: 300,
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'error'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '5px',
                  marginBottom: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '确认操作',
                      content: '确认删除共享容器' + params.row.pod_name + '请点击确认',
                      onOk () {
                        let postparams = {
                          id: parseInt(params.row.id)
                        }
                        axios.post(go.gourl + '/deleteSharePod', postparams, {
                          headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                            'Authorization': this.$cookie.get('token')
                          }
                        }).then(res => {
                          if (res.data.code === 1001) {
                            this.$Notice.success({
                              title: res.data.msg
                            })
                            this.refreshData()
                          } else {
                            this.$Notice.error({
                              title: res.data.msg
                            })
                          }
                        })
                          .catch(err => {
                            console.log(err)
                          })
                      }
                    })
                  }
                }
              }, '删除')
            ])
          }
        }
      ],
      columns: [
        {
          title: '容器名称',
          key: 'podname'
        },
        {
          title: '到期时间',
          key: 'due_time'
        },
        {
          title: '镜像名称',
          key: 'image'
        },
        {
          title: '运行状态',
          key: 'status',
          width: 120,
          render: (h, params) => {
            let status = 'error'
            let label = '其他问题'
            if (params.row.status === 'Pending') {
              status = 'error'
              label = '未创建成功'
            } else if (params.row.status === 'Running') {
              status = 'success'
              label = '正常运行'
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
          title: '容器端口',
          key: 'port'
        },
        {
          title: '已用内存(M)',
          key: 'usedMem'
        },
        {
          title: '已用CPU(MHz)',
          key: 'usedCPU'
        },
        {
          title: '容器控制台',
          key: 'cmd',
          align: 'center',
          width: '100',
          render: (h, params) => {
            return h('a', {
              attrs: { href: params.row.cmd, target: '_blank', title: params.row.cmd }
            }, '访问控制台')
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
                    this.userId = params.row.namespace
                    this.showModalCheckVM = true
                  }
                }
              }, '修改容器'),
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
                    console.log(this.url)
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
              }, '查看日志'),
              h('Button', {
                props: {
                  type: 'error'
                },
                style: {
                  marginRight: '5px',
                  marginTop: '5px',
                  marginBottom: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '确认操作',
                      content: '确认删除容器' + params.row.podname + '请点击确认',
                      onOk () {
                        let postparams = {
                          name: params.row.podname,
                          userId: params.row.namespace
                        }
                        axios.post(go.gourl + '/deleteContainer', postparams, {
                          headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                            'Authorization': this.$cookie.get('token')
                          }
                        }).then(res => {
                          if (res.data.code === 1001) {
                            this.$Notice.success({
                              title: res.data.msg
                            })
                            this.refreshData()
                          } else {
                            this.$Notice.error({
                              title: res.data.msg
                            })
                          }
                        })
                          .catch(err => {
                            console.log(err)
                          })
                      }
                    })
                  }
                }
              }, '删除')
            ])
          }
        }
      ],
      showData: [],
      showData1: [],
      wait: false,
      load: true,
      showModalAddVM: false
    }
  },
  created () {
    this.getData()
  },
  methods: {
    getData () {
      this.load = true
      axios.post(go.gourl + '/listUContainer', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      }).then(res => {
        this.load = false
        if (res.data.code === 1001) {
          this.showData = res.data.data
          this.showData.forEach(item => {
            item.usedCPU = item.info.usedCPU
            item.totalCPU = item.info.totalCPU
            item.usedMem = item.info.usedMem
            item.totalMem = item.info.totalMem
          })
          console.log(this.showData)
        } else {
          this.$Message.error(res.data.msg)
        }
      }).catch(res => {
        console.log(res)
      })
      axios.post(go.gourl + '/listSharePod', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      }).then(res => {
        this.load = false
        if (res.data.code === 1001) {
          this.showData1 = res.data.data
          this.showData1.forEach(item => {
            item.id = item.pod.id
            item.pod_name = item.pod.pod_name
            item.con_name = item.pod.con_name
            item.db_name = item.pod.db_name
            item.due_time = item.pod.due_time
            item.password = item.pod.password
            item.id = item.pod.id
          })
          console.log(this.showData1)
        } else {
          this.$Message.error(res.data.msg)
        }
      }).catch(res => {
        console.log(res)
      })
    },
    refreshData () {
      this.getData()
      this.$refs.table.changePage(1)
    }
  }
}
</script>
<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>

</style>
