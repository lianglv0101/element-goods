<template>
  <el-dialog
    :close-on-click-modal="false"
    :title="!form.id ? '新增' : !disabled ? '修改' : '查看'"
    width="780px"
    :visible.sync="dialogFormVisible"
    @close="dialogClosed"
  >
    <el-form
      ref="dataForm"
      :inline="true"
      :rules="rules"
      :model="form"
      label-position="right"
      label-width="100px"
    >
      <el-form-item label="商品名称" prop="name">
        <el-input :title="form.name" :disabled="disabled" show-word-limit maxlength="30" style="width: 250px" size="small" v-model="form.name"/>
      </el-form-item>
      <el-form-item label="商品价格" prop="price">
        <el-input :title="form.price" :disabled="disabled" type="number" style="width: 250px" size="small" v-model="form.price"/>
      </el-form-item>
      <el-form-item label="发布时间" prop="fbsj">
        <el-date-picker :title="form.fbsj" :disabled="disabled" value-format="yyyy-MM-dd HH:mm:ss" size="small" v-model="form.fbsj" style="width: 250px" type="datetime" placeholder="请选择发布时间" />
      </el-form-item>
      <el-form-item label="商品描述" prop="details">
        <el-input
          :disabled="disabled"
          style="width: 610px"
          size="small"
          type="textarea"
          placeholder="请输入内容"
          v-model="form.details"
          maxlength="300"
          show-word-limit
        >
        </el-input>
      </el-form-item>
      <el-form-item label="商品封面" prop="">
        <el-upload
          :disabled="disabled"
          action="/attachment/upload"
          name="multipartFile"
          :data="uploadData"
          list-type="picture-card"
          :on-success="handleAvatarSuccess"
          :file-list="fileList"
        >
          <i slot="default" class="el-icon-plus" />
          <div slot="file" slot-scope="{file}">
            <img
              class="el-upload-list__item-thumbnail"
              :src="file.url"
              alt=""
            >
            <span class="el-upload-list__item-actions">
              <span
                class="el-upload-list__item-preview"
                @click="handlePictureCardPreview(file)"
              >
                <i class="el-icon-zoom-in" />
              </span>
              <span
                v-if="!disabled"
                class="el-upload-list__item-delete"
                @click="handleDownload(file)"
              >
                <i class="el-icon-download" />
              </span>
              <span
                v-if="!disabled"
                class="el-upload-list__item-delete"
                @click="handleRemove(file)"
              >
                <i class="el-icon-delete" />
              </span>
            </span>
          </div>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible" append-to-body>
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisible = false">
        取消
      </el-button>
      <el-button type="primary" @click="createData()">
        确定
      </el-button>
    </div>
  </el-dialog>
</template>
<script>
import request from '@/utils/request'
import { v4 as uuidv4 } from 'uuid'
export default {
  data() {
    return {
      // 附件集合
      fileList: [],
      // 放大图片url
      dialogImageUrl: '',
      // 图片放大对话框可见
      dialogVisible: false,
      // 禁用
      disabled: false,
      // 表单
      form: {},
      // 上传附件额外参数
      uploadData: {},
      // 表单对话框可见
      dialogFormVisible: false,
      // 验证
      rules: {
        name: [{ required: true, message: '必填', trigger: 'blur' }],
        fbsj: [{ required: true, message: '必填', trigger: 'blur' }],
        price: [
          { required: true, message: '必填', trigger: 'blur' },
          { pattern: /(^[1-9][0-9]{0,7}$)|(^((0\.0[1-9]$)|(^0\.[1-9]\d?)$)|(^[1-9][0-9]{0,7}\.\d{1,2})$)/, message: '请输入价格正确格式，小数最多两位！' }
        ]
      }
    }
  },
  methods: {
    dialogClosed() {
      this.fileList = []
      this.form = {}
    },
    // 附件移除
    handleRemove(file) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request({
          url: '/attachment/deleteFile/' + file.id,
          method: 'delete',
          params: { src: file.src }
        }).then(res => {
          this.fileList = this.fileList.filter(item => item !== file)
          this.$message.success('删除成功')
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    // 附件上传成功回调
    handleAvatarSuccess(response, file, fileList) {
      if (response.code === 200) {
        file.id = response.data.id
        file.url = response.data.url
        file.src = response.data.src
        this.fileList.push(file)
        this.$message.success('上传成功')
      } else {
        this.$message.error(response.message)
      }
    },
    // 放大图片
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    // 下载图片
    handleDownload(file) {
      // window.location.href = 'http://localhost:8080/attachment/download?fileName=' + file.name + '&src=' + file.src
      request({
        url: '/attachment/download',
        method: 'get',
        responseType: 'blob',
        params: { fileName: file.name, src: file.src }
      }).then(res => {
        const url = window.URL.createObjectURL(new Blob([res]))
        const link = document.createElement('a')
        link.style.display = 'none'
        link.href = url
        link.setAttribute('download', file.name)
        document.body.appendChild(link)
        link.click()
        // 释放内存
        window.URL.revokeObjectURL(link.href)
      })
    },
    createData() {
      this.$refs.dataForm.validate(valid => {
        if (valid) {
          request({
            url: '/goods/save',
            method: 'post',
            data: JSON.stringify(this.form),
            headers: {
              'content-type': 'application/json'
            }
          }).then(res => {
            this.$message.success('保存成功')
            // 子组件调用父组件方法，并传递参数
            this.$emit('refreshDataList')
          })
        }
      })
    },
    // 初始化方法
    init(id, disabled) {
      if (id) {
        request({
          url: '/goods/getGoodsXxById/' + id,
          method: 'get'
        }).then(res => {
          this.uploadData.uuid = res.data.coverUuid
          if (res.data.cover) {
            this.fileList.push(res.data.cover)
          }
          this.form = res.data
        })
      } else {
        this.form.coverUuid = uuidv4()
        this.uploadData.uuid = this.form.coverUuid
      }
      this.disabled = disabled
      this.dialogFormVisible = true
    }
  }
}
</script>

<style scoped>

</style>
