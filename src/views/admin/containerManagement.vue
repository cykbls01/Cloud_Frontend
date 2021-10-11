<template>
  <div>
    <NavBar :breadlist="breadlist">
      <div slot="navbarContent">
        <CardContent>
          <AddImage @refresh="refreshData" :showModalAddExperiment.sync="showModalAddImage"></AddImage>
          <ModifyImage @refresh="refreshData" :showModalAddExperiment.sync="showModalModifyImage" :old-image="oldImage"></ModifyImage>
          <AddContainer @refresh="refreshData" :showModalAddVM.sync="showModalAddVM"></AddContainer>
          <AddSharePod @refresh="refreshData" :showModalAddVM.sync="showModalAddSP"></AddSharePod>
          <ModifyContainer :showModalCheckVM.sync="showModalCheckVM" :podName="podName" :userId="userId"></ModifyContainer>
          <LogContainer :showModalCheckVM.sync="showModalLogVM" :podName="podName" :namespace="port"></LogContainer>
          <ListImage :showModalCheckStudent.sync="showModalCheckStudent"></ListImage>
          <CheckContainer :showModalCheckVM.sync="showModalCheckCon" :podName="podName" :url="url"></CheckContainer>
          <TableComponent :loading="load" @refresh="refreshData" ref="table" :columnsFromFather="columnList" :allDataFromFather="dataList">
            <Button type="primary" @click="showModalAddVM = true">
              <Icon type="md-add-circle" class="addIcon"/>
              <span style="font-size: 14px">新增容器</span>
            </Button>
            <Button type="primary" @click="showModalAddImage = true">
              <Icon type="md-add-circle" class="addIcon"/>
              <span style="font-size: 14px">新增镜像</span>
            </Button>
            <Button type="primary" @click="showModalAddSP = true">
              <Icon type="md-add-circle" class="addIcon"/>
              <span style="font-size: 14px">新增账户</span>
            </Button>
            <Button type="primary" @click="modalAdd = true">
              <Icon type="md-add-circle" class="addIcon"/>
              <span style="font-size: 14px">新增节点</span>
            </Button>
            <br>
            <br>
            <ButtonGroup>
              <Button :type="currentType === 'container' ? 'primary' : 'default'" @click="cli('container')"><Icon type="md-information-circle" class="buttonIcon"/>容器管理</Button>
              <Button :type="currentType === 'pod'       ? 'primary' : 'default'" @click="cli('pod')"><Icon type="md-information-circle" class="buttonIcon"/>账户管理</Button>
              <Button :type="currentType === 'image'     ? 'primary' : 'default'" @click="cli('image')"><Icon type="md-information-circle" class="buttonIcon"/>镜像管理</Button>
              <Button :type="currentType === 'node'      ? 'primary' : 'default'" @click="cli('node')"><Icon type="md-information-circle" class="buttonIcon"/>集群管理</Button>
            </ButtonGroup>
          </TableComponent>
        </CardContent>
      </div>
    </NavBar>
    <Modal v-model="modalAdd" class="modal" :mask-closable="false" :closable="false">
      <p slot="header" class="modal-header">扩容集群</p>
      <div class="modal-content modal-row">点击确认以扩容集群</div>
      <div slot="footer">
        <Button size="large" style="border: 0px" @click="cancel">取消</Button>
        <Button size="large" type="info" style="border: 0px" @click="addOk">确认</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import NavBar from '../../components/NavBar'
import CardContent from '../../components/CardContent'
import TableComponent from '../../components/TableComponent'
import AddContainer from '../../components/teacher/container/addContainer'
import ModifyContainer from '../../components/teacher/container/modifyContainer'
import LogContainer from '../../components/teacher/container/logContainer'
import ListImage from '../../components/teacher/image/listImage'
import axios from 'axios'
import AddSharePod from '../../components/teacher/container/addSharePod'
import CheckContainer from '../../components/teacher/container/checkContainer'
import AddImage from '../../components/teacher/image/addImage'
import ModifyImage from '../../components/teacher/image/modifyImage'
import go from '../../assets/Global'
export default {
  name: 'containerManagement',
  components: {AddContainer, TableComponent, CardContent, NavBar, ModifyContainer, LogContainer, ListImage, AddSharePod, CheckContainer, AddImage, ModifyImage},
  data () {
    return {
      currentType: 'container',
      podName: '',
      port: '',
      oldImage: null,
      url: [],
      consoleUrl: '',
      nodeList: [],
      podList: [],
      column: [],
      containerList: [],
      dataList: [],
      imageList: [],
      showModalAddImage: false,
      showModalModifyImage: false,
      showModalCheckCon: false,
      showModalCheckVM: false,
      showModalAddSP: false,
      showModalLogVM: false,
      showModalCheckStudent: false,
      userId: '',
      breadlist: [
        {name: '容器管理', path: '/admin/containerManagement'}
      ],
      podColumn: [
        {
          title: '容器名称',
          key: 'pod_name'
        },
        {
          title: '到期时间',
          key: 'due_time'
        },
        {
          title: '使用者id',
          key: 'user_id'
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
      containerColumn: [
        {
          title: '容器名称',
          key: 'podname'
        },
        {
          title: '所有者id',
          key: 'namespace'
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
          title: '操作',
          key: 'operation',
          align: 'center',
          width: 300,
          render: (h, params) => {
            return h('div', [
              // TODO: 根据虚拟机类型判断是否有查看（直接vnc网页登录虚拟机）功能
              h('Button', {
                props: {
                  type: 'info',
                  disabled: params.row.namespace === 'kube-system' || params.row.namespace === 'share'
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
                    console.log(this.showModalCheckVM)
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
                    // console.log(this.url)
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
                  type: 'error',
                  disabled: params.row.namespace === 'kube-system' || params.row.namespace === 'share'
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
                              title: res.data.msg,
                              desc: '已成功删除容器'
                            })
                            this.refreshData()
                          } else {
                            console.log(res.data.msg)
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
      imageColumn: [
        {
          title: '镜像id',
          key: 'id',
          width: '150',
          sortable: 'custom'
        },
        {
          title: '镜像名',
          key: 'name',
          sortable: 'custom'
        },
        {
          title: '镜像端口号',
          key: 'port',
          sortable: 'custom'
        },
        {
          title: '镜像env',
          key: 'env',
          sortable: 'custom'
        },
        {
          title: '操作',
          key: 'oper',
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
                    this.showModalModifyImage = true
                    this.oldImage = params.row
                  }
                }
              }, '修改'),
              h('Button', {
                props: {
                  type: 'error'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '确认操作',
                      content: '确认删除镜像' + params.row.name + '请点击确认',
                      onOk () {
                        this.$http.post(go.gourl + '/deleteImage', {
                          id: String(params.row.id)
                        }, {
                          headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                            'Authorization': this.$cookie.get('token')
                          }
                        })
                          .then(res => {
                            if (res.data.code === 1001) {
                              this.$Notice.success({
                                title: '已成功删除镜像'
                              })
                              this.refreshData()
                            } else {
                              this.$Message.error(res.data.msg)
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
      nodeColumn: [
        {
          title: '序号',
          key: 'index',
          width: 80
        },
        {
          title: '主机地址',
          key: 'ip'
        },
        {
          title: '运行状态',
          key: 'status',
          width: 120,
          render: (h, params) => {
            let status = 'success'
            let label = '正常'
            if (params.row.status === 'Pending') {
              status = 'error'
              label = '离线'
            } else if (params.row.status === 'None') {
              status = 'error'
              label = '节点未创建'
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
          title: '总内存(M)',
          key: 'totalMem'
        },
        {
          title: '已用(M)',
          key: 'usedMem'
        },
        {
          title: '内存使用率',
          key: 'memoryUsage',
          render: (h, params) => {
            let usage = Math.round(params.row.usedMem / params.row.totalMem * 10000) / 100.00
            let color = '#22C688'
            if (usage > 75) {
              color = '#FF7875'
            }
            return h('div', {
              style: {
                display: 'flex'
              }
            }, [
              h('i-circle', {
                props: {
                  percent: usage,
                  size: 24,
                  'stroke-color': color,
                  'trail-color': '#E8E8E8',
                  'stroke-width': 20,
                  'trail-width': 20,
                  'stroke-linecap': 'square'
                }
              }),
              h('div', {}, usage + '%')]
            )
          }
        },
        {
          title: '总CPU(MHz)',
          key: 'totalCPU'
        },
        {
          title: '已用(MHz)',
          key: 'usedCPU'
        },
        {
          title: 'CPU使用率',
          key: 'cpuUsage',
          render: (h, params) => {
            let usage = Math.round(params.row.usedCPU / params.row.totalCPU * 10000) / 100.00
            let color = '#22C688'
            if (usage > 75) {
              color = '#FF7875'
            }
            return h('div', {
              style: {
                display: 'flex'
              }
            }, [
              h('i-circle', {
                props: {
                  percent: usage,
                  size: 24,
                  'stroke-color': color,
                  'trail-color': '#E8E8E8',
                  'stroke-width': 20,
                  'trail-width': 20,
                  'stroke-linecap': 'square'
                }
              }),
              h('div', {}, usage + '%')]
            )
            // let usage = Math.round(params.row.CPUUsed / params.row.CPUTotal * 10000) / 100.00
            // return h('div', {}, usage + '%')
          }
        },
        {
          title: '总磁盘(Mb)',
          key: 'totalStorage'
        },
        {
          title: '已用磁盘(Mb)',
          key: 'usedStorage'
        },
        {
          title: '磁盘使用率',
          key: 'cpuUsage',
          render: (h, params) => {
            let usage = Math.round(params.row.usedStorage / params.row.totalStorage * 10000) / 100.00
            let color = '#22C688'
            if (usage > 75) {
              color = '#FF7875'
            }
            return h('div', {
              style: {
                display: 'flex'
              }
            }, [
              h('i-circle', {
                props: {
                  percent: usage,
                  size: 24,
                  'stroke-color': color,
                  'trail-color': '#E8E8E8',
                  'stroke-width': 20,
                  'trail-width': 20,
                  'stroke-linecap': 'square'
                }
              }),
              h('div', {}, usage + '%')]
            )
          }
        },
        {
          title: '容器数目',
          key: 'podNum',
          align: 'center'
        },
        {
          title: '操作',
          key: 'oper',
          align: 'center',
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'info',
                  disabled: params.row.status === 'Running'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '确认操作',
                      content: '确认向集群中加入主机' + params.row.name + '请点击确认',
                      onOk () {
                        this.$http.post(go.gourl + '/addCluster', {
                          nodeName: String(params.row.name)
                        }, {
                          headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                            'Authorization': this.$cookie.get('token')
                          }
                        })
                          .then(res => {
                            if (res.data.code === 1001) {
                              this.$Notice.success({
                                title: '开始扩容'
                              })
                              this.refreshData()
                            } else {
                              this.$Message.error(res.data.msg)
                            }
                          })
                          .catch(err => {
                            console.log(err)
                          })
                      }
                    })
                  }
                }
              }, '扩容主机'),
              h('Button', {
                props: {
                  type: 'error',
                  disabled: params.row.status !== 'Running'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '确认操作',
                      content: '确认从集群中删除主机' + params.row.name + '请点击确认',
                      onOk () {
                        this.$http.post(go.gourl + '/deleteCluster', {
                          nodeName: String(params.row.name)
                        }, {
                          headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                            'Authorization': this.$cookie.get('token')
                          }
                        })
                          .then(res => {
                            if (res.data.code === 1001) {
                              this.$Notice.success({
                                title: '开始缩容'
                              })
                              this.refreshData()
                            } else {
                              this.$Message.error(res.data.msg)
                            }
                          })
                          .catch(err => {
                            console.log(err)
                          })
                      }
                    })
                  }
                }
              }, '缩容主机')
            ])
          }
        }
      ],
      showData: [],
      showData1: [],
      columnList: [],
      allData1: [],
      wait: false,
      load: true,
      showModalAddVM: false,
      modalAdd: false
    }
  },
  computed: {
  },
  created () {
    this.getData()
    this.timer = setTimeout(() => { // 设置延迟执行
      this.cli('container')
    }, 1000)
  },
  methods: {
    cli (a) {
      this.currentType = a
      console.log(this.currentType)
      if (this.currentType === 'container') {
        this.columnList = this.containerColumn
        this.dataList = this.containerList
      } else if (this.currentType === 'pod') {
        this.columnList = this.podColumn
        this.dataList = this.podList
      } else if (this.currentType === 'image') {
        this.columnList = this.imageColumn
        this.dataList = this.imageList
      } else if (this.currentType === 'node') {
        this.columnList = this.nodeColumn
        this.dataList = this.nodeList
      }
    },
    addOk () {
      axios.post(go.gourl + '/expandCluster', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      }).then(res => {
        if (res.data.code === 1001) {
          this.$Message.info(res.data.msg)
        } else {
          this.$Message.error(res.data.msg)
          this.loading = false
        }
      })
      this.cancel()
    },
    cancel () {
      this.modalAdd = false
    },
    getData () {
      this.load = true
      axios.post(go.gourl + '/listAContainer', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      }).then(res => {
        if (res.data.code === 1001) {
          this.showData = res.data.data
          this.showData.forEach(item => {
            if (item.url === null || item.url === undefined) {
              item.url = []
            }
            item.usedCPU = item.info.usedCPU
            item.totalCPU = item.info.totalCPU
            item.usedMem = item.info.usedMem
            item.totalMem = item.info.totalMem
          })
          this.containerList = this.showData
        } else {
          this.$Message.error(res.data.msg)
        }
      }).catch(res => {
        console.log(res)
      })
      axios.post(go.gourl + '/nodeMetric', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      }).then(res => {
        if (res.data.code === 1001) {
          this.allData1 = res.data.data
          this.nodeList = this.allData1
          this.load = false
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
        if (res.data.code === 1001) {
          this.showData1 = res.data.data
          this.showData1.forEach(item => {
            item.id = item.pod.id
            item.pod_name = item.pod.pod_name
            item.con_name = item.pod.con_name
            item.db_name = item.pod.db_name
            item.due_time = item.pod.due_time
            item.password = item.pod.password
            item.user_id = item.pod.user_id
          })
          this.podList = this.showData1
          console.log(this.podList)
        } else {
          this.$Message.error(res.data.msg)
        }
      }).catch(res => {
        console.log(res)
      })
      axios.post(go.gourl + '/listImage', {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.imageList = res.data.data
          } else {
            this.$Message.error(res.data.msg)
            this.imageList = []
          }
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    refreshData () {
      this.getData()
      this.timer = setTimeout(() => { // 设置延迟执行
        this.cli('container')
      }, 1000)
      this.$refs.table.changePage(1)
    }
  }
}
</script>
<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>

</style>
