<template>
  <Modal
    v-model="show"
    @on-ok="ok"
    @on-cancel="cancel" width="800px">
    <p slot="header" style="margin-top:10px;height:25px;font-size: 20px;text-align:center;font-weight:normal">
      <span>管理镜像</span>
    </p>
    <div style="margin-top:10px;margin-left:20px;margin-right:20px;">
      <AddImage @refresh="refreshData2" :showModalAddExperiment.sync="showModalAddImage"></AddImage>
      <ModifyImage @refresh="refreshData2" :showModalAddExperiment.sync="showModalModifyImage" :old-image="oldImage"></ModifyImage>
      <TableComponent :buttonSpan="12" :loading="loading" @refresh="refreshData2" ref="table" :columnsFromFather="columns2" :allDataFromFather="allData2">
        <Button type="primary" @click="showModalAddImage = true">
          <Icon type="md-add-circle" class="addIcon"/>
          <span style="font-size: 14px">新增镜像</span>
        </Button>
      </TableComponent>
    </div>
    <div slot="footer">
      <Button type="text" size="large" @click="cancel">取消</Button>
      <Button type="primary" size="large" @click="ok">确定</Button>
    </div>
  </Modal>
</template>

<script>
import TableComponent from '../../TableComponent'
import AddImage from '../image/addImage'
import ModifyImage from '../image/modifyImage'
import axios from 'axios'
import go from '../../../assets/Global'
export default {
  name: 'listImage',
  components: {
    TableComponent, AddImage, ModifyImage
  },
  props: {
    showModalCheckStudent: {
      type: Boolean,
      default: false
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
      allDataUrl2: go.gourl + '/listImage',
      showModalAddImage: false,
      showModalModifyImage: false,
      oldImage: null,
      loading: false,
      columns2: [
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
          width: '150',
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
                          this.refreshData2()
                        } else {
                          this.$Message.error(res.data.msg)
                        }
                      })
                      .catch(err => {
                        console.log(err)
                      })
                  }
                }
              }, '删除')
            ])
          }
        }
      ]
    }
  },
  watch: {
  },
  mounted () {
    this.getAllData2()
  },
  methods: {
    getAllData2 () {
      axios.post(this.allDataUrl2, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Authorization': this.$cookie.get('token')
        }
      })
        .then(res => {
          if (res.data.code === 1001) {
            this.allData2 = res.data.data
          } else {
            this.$Message.error(res.data.msg)
            this.allData2 = []
          }
        })
        .catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
    },
    refreshData2 () {
      this.loading = false
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
