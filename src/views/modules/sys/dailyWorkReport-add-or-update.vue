<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="本日完成工作" prop="workDoneThisWeek">
        <el-input v-model="dataForm.workDoneToday" placeholder="本日完成工作"></el-input>
      </el-form-item>
      <el-form-item label="本日未工作" prop="weekSummary">
        <el-input v-model="dataForm.unfinishedWork"  placeholder="本日未完成的工作"></el-input>
      </el-form-item>
      <el-form-item label="需要协调的工作" prop="coordinate">
        <el-input v-model="dataForm.coordinate" placeholder="需要协调的工作"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remarks">
        <el-input v-model="dataForm.remarks" placeholder="备注"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          workDoneToday: '',
          unfinishedWork: '',
          coordinate: '',
          remarks: ''
        }
        // dataRule: {
        //   projectName: [
        //     { required: true, message: '项目名称不能为空', trigger: 'blur' }
        //   ],
        //   taskName: [
        //     { required: true, message: '任务名称不能为空', trigger: 'blur' }
        //   ],
        //   personInCharge: [
        //     { required: true, message: '负责人不能为空', trigger: 'blur' }
        //   ],
        //   auditor: [
        //     { required: true, message: '审核人不能为空', trigger: 'blur' }
        //   ]
        // }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/role/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.roleList = data && data.code === 0 ? data.list : []
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/dailyWorkReport/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.workDoneToday = data.dailyWorkReport.workDoneToday
                this.dataForm.unfinishedWork = data.dailyWorkReport.unfinishedWork
                this.dataForm.coordinate = data.dailyWorkReport.coordinate
                this.dataForm.remarks = data.dailyWorkReport.remarks
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/dailyWorkReport/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'workDoneToday': this.dataForm.workDoneToday,
                'unfinishedWork': this.dataForm.unfinishedWork,
                'coordinate': this.dataForm.coordinate,
                'remarks': this.dataForm.remarks,
                'roleIdList': this.dataForm.roleIdList
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
