<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="项目名" prop="projectName">
        <el-input v-model="dataForm.projectName" placeholder="项目名称"></el-input>
      </el-form-item>
      <el-form-item label="项目编号" prop="number">
        <el-input v-model="dataForm.number"  placeholder="项目编号"></el-input>
      </el-form-item>
      <el-form-item label="项目负责人" prop="personInCharge">
        <el-input v-model="dataForm.personInCharge" placeholder="项目负责人"></el-input>
      </el-form-item>
      <el-form-item label="开始时间" prop="startsTime">
        <el-date-picker v-model="dataForm.startsTime" placeholder="开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" size="mini" prop="endTime">
        <el-date-picker v-model="dataForm.endTime" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">未开始</el-radio>
          <el-radio :label="1">开发中</el-radio>
          <el-radio :label="2">延期</el-radio>
          <el-radio :label="3">完成</el-radio>
        </el-radio-group>
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
          number: '',
          personInCharge: '',
          startsTime: '',
          endTime: '',
          status: 1,
          remark: ''
        },
        dataRule: {
          projectName: [
            { required: true, message: '项目名不能为空', trigger: 'blur' }
          ],
          number: [
            { required: true, message: '项目编号不能为空', trigger: 'blur' }
          ],
          personInCharge: [
            { required: true, message: '项目负责人不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/sys/projectM/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.projectName = data.projectM.projectName
                this.dataForm.number = data.projectM.number
                this.dataForm.personInCharge = data.projectM.personInCharge
                this.dataForm.startsTime = data.projectM.startsTime
                this.dataForm.endTime = data.projectM.endTime
                this.dataForm.status = data.projectM.status
                this.dataForm.remark = data.projectM.remark
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
              url: this.$http.adornUrl(`/sys/projectM/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectName': this.dataForm.projectName,
                'number': this.dataForm.number,
                'personInCharge': this.dataForm.personInCharge,
                'startsTime': this.dataForm.startsTime,
                'endTime': this.dataForm.endTime,
                'remark': this.dataForm.remark,
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
