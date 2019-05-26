<template>
    <div style="margin: 10px 10px 0 10px;">
<!--      <el-row>-->
<!--        <el-col :spn="6" :offset="6">-->
          <div class="upload-box">
            <el-button @click="showUpload = !showUpload" icon="el-icon-upload" style="width: 100%">上传文件</el-button>
            <div v-show="showUpload" style="padding: 10px 0">
              <el-upload
                class="upload-demo"
                ref="upload"
                :auto-upload="false"
                drag
                action="http://192.168.217.128:9528/upload"
                :on-success="uploadSuccess"
                :on-error="uploadFailed"
                :file-list="uploadList"
                :data="formData()"
                :on-change="checkFile"
                multiple>
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">一次最多上传3个文件,不能上传名称含有中文的文件,上传文件不能超过1G
                  <br>
                  <el-button style="margin-top: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
                </div>

              </el-upload>
            </div>

          </div>

      <div style="margin-top: 20px">
        <el-row>
          <el-col>
            <h1>Index of {{path}}</h1>
          </el-col>
        </el-row>

        <div>
          <el-row :gutter="10">
            <el-col :span="2.5"><el-button @click="toParentDirectory" :disabled="path === '/'" icon="el-icon-back" type="info">返回</el-button></el-col>
            <el-col :span="2.5"><el-button @click="getFiles(path)" icon="el-icon-refresh" type="primary">刷新</el-button></el-col>
            <el-col :span="3.5" :offset="15"><el-button @click="createFolder" icon="el-icon-plus" type="warning">创建文件夹</el-button></el-col>
          </el-row>
        </div>


        <div>
          <el-table
            :data="file_list"
            style="width: 100%">
            <el-table-column
              prop="name"
              label="文件名"
              sortable>
              <template slot-scope="scope">
                <div v-if="scope.row.type === 'file'">
                  <a :href="URL + path + scope.row.name" style="font-size: 15px;color: black">{{ scope.row.name }}</a>
                </div>
                <div v-else>
                  <a @click="changeDir(scope.row.name)" style="font-size: 15px;color: #1482f0">{{ scope.row.name }}</a>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="modify_time"
              label="最后修改时间"
              sortable>
            </el-table-column>
            <el-table-column
              prop="size"
              label="文件大小"
              sortable>
            </el-table-column>
            <el-table-column
              label="操作"
              width="100">
              <template slot-scope="scope">
                <el-button size="small" type="danger" @click="deleteFile(scope.row)" ico="el-icon-delete">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
    </div>
</template>

<script>
  import axios from 'axios'

    export default {
      name: 'file',
      data() {
        return {
          URL: 'http://192.168.217.128:9528',
          header: { 'Content-Type': 'application/json' },
          path: '/',
          uploadList: [],
          file_list: [],
          showUpload: false
        }
      },
      created() {
        this.getFiles(this.path)
      },
      methods: {
        checkProgress(event, file, fileList) {
          console.log(event, file, fileList)
        },
        createFolder() {
          this.$prompt('请输入文件夹名称', '创建文件夹', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            inputPattern: /^[a-zA-Z0-9\-_]+$/,
            inputErrorMessage: '文件夹命名不规范'
          }).then(({ value }) => {
            axios.get(this.URL + '/create', {
              params: {
                folder: this.path + value
              }
            })
              .then((result) => {
                if(result.data.status === 1) {
                  this.$message.success('文件夹创建成功');
                  this.getFiles(this.path)
                } else {
                  this.$message.error(result.data.message)
                }
              })
              .catch((error) => {
                console.error(error)
              })

          })
        },
        changeDir(foldername) {
          console.log(foldername)
          this.getFiles(this.path + foldername)
        },
        downloadFile(filename) {
          axios.get(this.URL + this.path + filename)
            .then((result) => {
              console.log(resutl)
            })
            .catch((error) => {
              console.error(error)
            })
        },
        toParentDirectory() {
          console.log(this.path)
          let currpath = this.path
          currpath = (currpath.substring(currpath.length-1)==='/')? currpath.substring(0,currpath.length-1):currpath;
          // let path_tmp = (this.path.substring(this.path.length-1)==='/')? this.path.substring(0,this.path.length-1):this.path;
          // console.log('currpath:', currpath)
          let index = currpath.lastIndexOf('/')
          // console.log(index)
          let parentDir = currpath.substr(0,index + 1)
          console.log('parentDir:', parentDir)
          // // this.path = this.path.substr(0,index)
          this.getFiles(parentDir)
        },
        getFiles(path) {
          // console.log('URL:', this.URL + path)
          axios.get(this.URL + path)
            .then((result) => {
              console.log(result)
              if(result.data.status === 1){
                this.path = path
                this.file_list = result.data.data.file_list
                console.log('path:', path)
              } else {
                this.$message.error(result.data.message)
              }
            })
            .catch((error) => {
              console.error(error)
            })
        },
        checkFile(file, uploadList) {
          if(/.*[\u4e00-\u9fa5]+.*$/.test(file.name)) {
            this.$message.error(file.name + '包含中文,禁止上传')
            if(uploadList.indexOf(file) !== -1) {
              uploadList.splice(this.uploadList.indexOf(file), 1)
            }
            return false
          }
          if(file.size > 1024*1024*1024) {
            this.$message.error(file.name + '超过限制(1G),禁止上传')
            if(uploadList.indexOf(file) !== -1) {
              uploadList.splice(this.uploadList.indexOf(file), 1)
            }
            return false
          }
          this.file_list.forEach((existsfile) => {
            if(existsfile.name === file.name) {
              this.$message.error(file.name + '已存在,禁止上传')
              if(uploadList.indexOf(file) !== -1) {
                uploadList.splice(this.uploadList.indexOf(file), 1)
              }
              return false
            }
          })
          let count = 0
          uploadList.forEach((upfile) => {
            if(upfile.name === file.name) {
              count++
            }
          })
          if(count > 1) {
            this.$message.error(file.name + '禁止上传同名文件')
            if(uploadList.indexOf(file) !== -1) {
              uploadList.splice(this.uploadList.indexOf(file), 1)
            }
            return false
          }
          this.uploadList = uploadList.slice(-3);
        },
        submitUpload() {
          this.$refs.upload.submit();
        },
        formData() {
          return {
            'path': this.path
          }
        },
        uploadSuccess(response, file, uploadList) {
          // console.log(response, file, uploadList)
          if(response.status === 0) {
            this.$message.error(response.message)
            // if(this.uploadList.indexOf(file) !== -1) {
            //   this.uploadList.splice(this.uploadList.indexOf(file), 1)
            // }
            file.status = 'ready'
          }
          this.getFiles(this.path)
        },
        uploadFailed(err, file, uploadList) {
          console.error('上传失败', err, file, uploadList)
        },
        beforeRemove(file, uploadList) {
          console.log(file)
          if(file.status === 'success') {
            this.deleteFile(file)
            return false
          } else {
            return true
          }

        },
        deleteFile(file) {
          this.$confirm('此操作将删除文件服务器上该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            center: true
          })
            .then(() => {
              axios.get('http://192.168.217.128:9528/delete', {
                params: {
                  'filename': this.path + file.name
                }
              })
                .then((result) => {
                  console.log(result)
                  if(result.data.status === 1) {
                    this.$message.success('删除成功')
                    // this.uploadList.splice(this.uploadList.indexOf(file), 1)
                    this.getFiles(this.path)
                  } else {
                    this.$message.error(result.data.message)
                  }
                })
                .catch((error) => {
                  console.error(error)
                })
            })
            .catch(() => {
              this.$message({
                type: 'info',
                message: '已取消删除'
              });
            });
        }
      }
    }
</script>

<style scoped>

  .upload-box {
    box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
  }

  .upload-demo {
    width: 50%;
    text-align:center;
    margin: auto;
  }

  .el-upload__tip {
    color: red;
  }

  /*.upload-demo >>> .el-upload-list {*/
  /*  text-align: left;*/
  /*}*/
</style>
