<template>
  <div>
    <NavBar :breadlist="breadlist">
      <div slot="navbarContent">
        <CardContent>
          <TableComponent @refresh="refreshData" ref="table" :columnsFromFather="columns1" :allDataFromFather="allData">
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
import PeerDetails from '../student/peerDetails'
// import axios from 'axios'
export default {
  name: 'machineManagement',
  components: {PeerDetails, TableComponent, CardContent, NavBar},
  data () {
    return {
      breadlist: [{name: '主机管理', path: '/machine/admin'}],
      allData: [],
      allData1: [],
      columns1: [
        {
          title: '序号',
          key: 'index',
          width: 80
        },
        {
          title: '主机地址',
          key: 'HostName'
        },
        {
          title: '是否连接',
          key: 'IsConnected',
          width: 100,
          render: (h, params) => {
            let status = '否'
            if (params.row.IsConnected) {
              status = '是'
            }
            return h('div', {}, status)
          }
        },
        {
          title: '运行状态',
          key: 'RunTimeState',
          width: 120,
          render: (h, params) => {
            let status = 'success'
            let label = '正常'
            if (params.row.RunTimeState === 'red') {
              status = 'error'
              label = '错误'
            } else if (params.row.RunTimeState === 'yellow') {
              status = 'error'
              label = '警告'
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
          key: 'MemoryTotal'
        },
        {
          title: '已用(M)',
          key: 'MemoryUsed'
        },
        {
          title: '内存使用率',
          key: 'memoryUsage',
          render: (h, params) => {
            let usage = Math.round(params.row.MemoryUsed / params.row.MemoryTotal * 10000) / 100.00
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
          key: 'CPUTotal'
        },
        {
          title: '已用(MHz)',
          key: 'CPUUsed'
        },
        {
          title: 'CPU使用率',
          key: 'cpuUsage',
          render: (h, params) => {
            let usage = Math.round(params.row.CPUUsed / params.row.CPUTotal * 10000) / 100.00
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
          title: '虚拟机数目',
          key: 'vmCount',
          align: 'center',
          maxWidth: 120,
          render: (h, params) => {
            return h('div', {}, params.row.VirtualMachineList.length)
          }
        }
      ],
      columns2: [
        {
          title: '序号',
          key: 'index',
          width: 80
        },
        {
          title: '主机地址',
          key: 'name'
        },
        // {
        //   title: '运行状态',
        //   key: 'RunTimeState',
        //   width: 120,
        //   render: (h, params) => {
        //     let status = 'success'
        //     let label = '正常'
        //     if (params.row.RunTimeState === 'red') {
        //       status = 'error'
        //       label = '错误'
        //     } else if (params.row.RunTimeState === 'yellow') {
        //       status = 'error'
        //       label = '警告'
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
            // let usage = Math.round(params.row.CPUUsed / params.row.CPUTotal * 10000) / 100.00
            // return h('div', {}, usage + '%')
          }
        },
        {
          title: '容器数目',
          key: 'vmCount',
          align: 'center',
          maxWidth: 120,
          render: (h, params) => {
            return h('div', {}, params.row.VirtualMachineList.length)
          }
        }
      ],
      createMachineModal: false,
      hostname: '',
      username: '',
      password: '',
      vmData: {
        user_id: '',
        vm_config: {
          Name: '',
          CPU: 0,
          Memory: 0,
          Disk: 0,
          TemplateName: ''
        }
      },
      modalAdd: false,
      MvmData: {
        vmName: '',
        cpu: 0,
        memory: 0,
        disk: 0
      }
    }
  },
  mounted () {
    this.getAllData()
  },
  methods: {
    addOk () {
      if (this.vmData.cpu === '' || this.vmData.memory === '' || this.vmData.disk === '') {
        this.$Message.info('请完善表格内容')
      } else {
        this.loading = true
        this.MvmData.cpu = parseInt(this.MvmData.cpu)
        this.MvmData.memory = parseInt(this.MvmData.memory) * 1024
        this.MvmData.disk = parseInt(this.MvmData.disk) * 1024
        this.vmData.vm_config.CPU = this.MvmData.cpu
        this.vmData.vm_config.Disk = this.MvmData.disk
        this.vmData.vm_config.Memory = this.MvmData.memory
        console.log(this.vmData)
        this.$http.post('/vm/createNode', this.vmData)
          .then(res => {
            console.log(this.vmData)
            if (res.data.code === 1001) {
              this.$Message.info('创建正在进行,稍后查看')
              this.refreshData()
            } else {
              this.$Message.error(res.data.msg)
              this.loading = false
            }
          })
        this.cancel()
      }
    },
    getAllData () {
      this.$http.get('/vm/getHosts')
        .then(res => {
          if (res.data.code === 1001) {
            this.allData = res.data.data
          } else {
            this.$Message.error(res.data.msg)
          }
        }).catch(err => {
          console.log(err)
        })
      // axios.post('http://localhost:8080/api/nodeMetric', {
      //   headers: {
      //     'Content-Type': 'application/x-www-form-urlencoded',
      //     'Authorization': this.token
      //   }
      // }).then(res => {
      //   this.load = false
      //   if (res.data.code === 1001) {
      //     this.allData1 = res.data.data
      //     console.log(this.allData1)
      //   } else {
      //     this.$Message.error(res.data.msg)
      //   }
      // }).catch(res => {
      //   console.log(res)
      // })
    },
    refreshData () {
      this.getAllData()
    },
    cancel () {
      this.createMachineModal = false
    },
    submit () {
      if (this.hostname !== '' && this.username !== '' && this.password !== '') {
        let data = {
          hostname: this.hostname,
          username: this.username,
          password: this.password
        }
        this.$http.post('/machine/add', data)
          .then(res => {
            if (res.data.code === 1001) {
              this.$Notice.success({
                title: '添加成功',
                desc: '主机' + this.hostname + '已添加'
              })
              this.getAllData()
            } else {
              this.$Message.error(res.data.msg)
            }
          })
        this.createMachineModal = false
        this.hostname = ''
        this.username = ''
        this.password = ''
      } else {
        this.$Message.error('请完善信息')
      }
    },
    strockColor (rate) {
      if (rate > 0.8) {
        return '#ed4014'
      } else if (rate > 0.6) {
        return '#ff9900'
      } else {
        return '#19be6b'
      }
    }
  }
}
</script>

<style scoped src="../../assets/CommonStyle.css"></style>
<style scoped>
  /deep/ .ivu-col-span-15{
    display: none;
  }
  /deep/ .ivu-col-span-9{
    float: left;
    text-align: left !important;
  }
  .vertical-center-modal{
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .center-modal-box{
    padding: 0 18px 10px 10px;
  }
  .center-modal-box label{
    margin-right: 10px;
    line-height: 30px;
    font-size: medium;
    font-weight: bold;
  }
  .center-modal-box span{
    font-size: small;
    font-weight: lighter;
  }
  .card-list{
    margin: 10px 5px 10px 10px;
    padding-left: 10px;
    padding-right: 10px;
  }
  .card-list-circle{
    justify-content: space-between;
  }
  .card-list-box{
    display: flex;
    flex-wrap: wrap;
  }
  .modal-block label{
    margin-left: 10px;
    width: 20%;
    display: inline-block;
    text-align: right;
  }
  .modal-block{
    margin-top: 10px;
  }
</style>
