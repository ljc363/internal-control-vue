<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="项目名" prop="projectName">
        <el-input v-model="dataForm.projectName" placeholder="项目名称"></el-input>
      </el-form-item>
      <el-form-item label="任务名称" prop="taskName">
        <el-input v-model="dataForm.taskName"  placeholder="任务名称"></el-input>
      </el-form-item>
      <el-form-item label="子任务" prop="subTask">
        <el-input v-model="dataForm.subTask" placeholder="子任务"></el-input>
      </el-form-item>
      <el-form-item label="计划开始时间" prop="plannedStartTime">
        <el-date-picker v-model="dataForm.plannedStartTime" placeholder="计划开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="计划结束时间" size="mini" prop="plannedEndTime">
        <el-date-picker v-model="dataForm.plannedEndTime" placeholder="计划结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="实际开始时间" prop="actualStartTime">
        <el-date-picker v-model="dataForm.actualStartTime" placeholder="实际开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="实际结束时间" size="mini" prop="actualEndTime">
        <el-date-picker v-model="dataForm.actualEndTime" placeholder="实际结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">未开始</el-radio>
          <el-radio :label="1">开始中</el-radio>
          <el-radio :label="2">完成</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="负责人" prop="personInCharge">
        <el-input v-model="dataForm.personInCharge" placeholder="负责人"></el-input>
      </el-form-item>
      <el-form-item label="审核人" prop="auditor">
        <el-input v-model="dataForm.auditor"  auditor="审核人"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
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
          projectName: '',
          taskName: '',
          subTask: '',
          plannedStartTime: '',
          plannedEndTime: '',
          actualEndTime: '',
          personInCharge: '',
          actualStartTime: '',
          auditor: '',
          status: 1,
          remark: ''
        },
        dataRule: {
          projectName: [
            { required: true, message: '项目名称不能为空', trigger: 'blur' }
          ],
          taskName: [
            { required: true, message: '任务名称不能为空', trigger: 'blur' }
          ],
          personInCharge: [
            { required: true, message: '负责人不能为空', trigger: 'blur' }
          ],
          auditor: [
            { required: true, message: '审核人不能为空', trigger: 'blur' }
          ]
        }
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
              url: this.$http.adornUrl(`/sys/taskSchedule/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.projectName = data.taskSchedule.projectName
                this.dataForm.taskName = data.taskSchedule.taskName
                this.dataForm.subTask = data.taskSchedule.subTask
                this.dataForm.plannedStartTime = data.taskSchedule.plannedStartTime
                this.dataForm.plannedEndTime = data.taskSchedule.plannedEndTime
                this.dataForm.actualEndTime = data.taskSchedule.actualEndTime
                this.dataForm.actualStartTime = data.taskSchedule.actualStartTime
                this.dataForm.personInCharge = data.taskSchedule.personInCharge
                this.dataForm.auditor = data.taskSchedule.auditor
                this.dataForm.status = data.taskSchedule.status
                this.dataForm.remark = data.taskSchedule.remark
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
              url: this.$http.adornUrl(`/sys/taskSchedule/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectName': this.dataForm.projectName,
                'taskName': this.dataForm.taskName,
                'subTask': this.dataForm.subTask,
                'plannedStartTime': this.dataForm.plannedStartTime,
                'plannedEndTime': this.dataForm.plannedEndTime,
                'actualEndTime': this.dataForm.actualEndTime,
                'actualStartTime': this.dataForm.actualStartTime,
                'personInCharge': this.dataForm.personInCharge,
                'auditor': this.dataForm.auditor,
                'status': this.dataForm.status,
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
