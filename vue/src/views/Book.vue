<template>
  <div style="padding: 10px">
    <div>
      <el-button type="primary" @click="add">新增</el-button>
    </div>

    <div style="margin: 10px 0">
      <el-input v-model="search" placeholder="请输入关键字" style="width: 20%" clearable></el-input>
      <el-button type="primary" style="margin-left: 5px" @click="load">查询</el-button>
    </div>
    <el-table :data="tableData" border stripe style="width: 100%">
      <el-table-column prop="id" label="ID" sortable/>
      <el-table-column prop="name" label="名称"/>
      <el-table-column prop="price" label="单价" />
      <el-table-column prop="author" label="作者" />
      <el-table-column prop="createTime" label="出版时间" />
      <el-table-column  label="封面">
        <template #default="scope">
          <el-image
              style="width: 100px; height: 100px"
              :src="scope.row.cover"
              :preview-src-list="[scope.row.cover]"
          />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template #default="scope">
          <el-button size="mini" @click="handleEdit(scope.row)">编辑</el-button>
          <el-popconfirm title="确认删除吗？" @confirm="handleDelete(scope.row.id)">
            <template #reference>
              <el-button size="mini" type="danger">删除</el-button>
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <div style="margin: 10px 0">
      <el-pagination
          v-model:currentPage="currentPage4"
          v-model:page-size="pageSize"
          :page-sizes="[5, 10, 20]"
          :page-size="pageSize"
          :small="small"
          :disabled="disabled"
          :background="background"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
      />

      <el-dialog v-model="dialogVisible" title="提示" width="30%" :before-close="handleClose">
        <el-form :model="form" label-width="120px">
          <el-form-item label="名称">
            <el-input v-model="form.name" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="价格">
            <el-input v-model="form.price" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="作者">
            <el-input v-model="form.author" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="出版时间">
            <el-date-picker v-model="form.createTime" value-format="YYYY-MM-DD" type="date" style="width: 80%" clearable></el-date-picker>
          </el-form-item>
          <el-form-item label="封面">
            <el-upload ref="upload" action="http://localhost:9090/files/upload" :on-success="fileUploadSuccess">
              <el-button type="primary">点击上传</el-button>
            </el-upload>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="dialogVisible = false">取消</el-button>
            <el-button type="primary" @click="save">确定</el-button>
          </span>
        </template>
      </el-dialog>

    </div>
  </div>
</template>

<script>

import request from "@/utils/request";

export default {
  name: 'Book',
  components: {

  },
  data(){
    return{
      form: {},
      dialogVisible: false,
      search: '',
      currentPage4: 1,
      pageSize: 10,
      total: 0,
      tableData: []
    }
  },
  created(){
    this.load()
  },
  methods: {
    fileUploadSuccess(res) {
      console.log(res)
      this.form.cover = res.data
    },
    load() {
      request.get("/book",{
        params: {
          pageNum: this.currentPage4,
          pageSize: this.pageSize,
          search: this.search
        }
      }).then(res => {
        console.log(res)
        this.tableData = res.data.records
        this.total = res.data.total
      })
    },
    add() {
      this.dialogVisible = true
      this.form = {}
      this.$refs['upload'].clearFiles() //清除历史文件列表
    },
    save() {
      if (this.form.id){  //更新
        request.put("/book", this.form).then(res => {
          console.log(res)
          if (res.code === '0'){
            this.$message({
              type: "success",
              message: "更新成功"
            })
          } else {
            this.$message({
              type: "error",
              message: res.msg
            })
          }
          this.load() //刷新表格数据
          this.dialogVisible = false
        })
      } else { //新增
        request.post("/book", this.form).then(res => {
          console.log(res)
          if (res.code === '0'){
            this.$message({
              type: "success",
              message: "新增成功"
            })
          } else {
            this.$message({
              type: "error",
              message: res.msg
            })
          }
          this.load() //刷新表格数据
          this.dialogVisible = false
        })
      }
    },
    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true
      this.$nextTick(() => {
        this.$refs['upload'].clearFiles() //清除历史文件列表
      })
    },
    handleDelete(id){
      console.log(id)
      request.delete("/book/" + id).then(res => {
        if (res.code === '0'){
          this.$message({
            type: "success",
            message: "删除成功"
          })
        } else {
          this.$message({
            type: "success",
            message: res.msg
          })
        }
      })
      this.load()
    },
    handleSizeChange() {  //改变当前每页的个数触发
      this.load()
    },
    handleCurrentChange(){  //改变当前页码触发
      this.load()
    }
  }
}
</script>
