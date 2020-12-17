<template>
<!--  包含文件导出功能 -->
  <div class="com_project">
    <el-card class="box-card">
      <!-- 筛选部分 -->
      <div slot="header" class="clearfix">
        <span>仓库进设备记录 </span>
        <el-form :inline="true" class="demo-form-inline" style="float: right;">
          <el-form-item label>
            <el-input @clear="handleClear" placeholder="请输入关键字" clearable prefix-icon="el-icon-search" size="mini"
              v-model="keywords">
            </el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" size="mini" @click="filterData">查询</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" plain size="mini" @click="handleEdit(1)">增加记录</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="success" plain size="mini" @click="exportExcel(1)">
              全部导出
            </el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" plain size="mini" @click="exportExcel(0)"
              v-if="torole=='super_editor'&& multipleSelection.length>=1">批量导出</el-button>
          </el-form-item>
        </el-form>
      </div>
      <!-- 项目列表部分 -->
      <div class="pro_list">
        <el-table v-loading="listLoading" :data="list" element-loading-text="Loading" border fit highlight-current-row
          @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column v-for="(item,index) in tableHeaderList" :key="index" align="center" :label="item.label"
            :prop="item.prop">
          </el-table-column>
          <el-table-column align="center" label="照片" prop="zhaopian">
            <template slot-scope="scope">
              <span>
                <viewer>
                  <img class="photo-style" :src="imageLogo + scope.row.zhaopian" alt v-if="scope.row.zhaopian" />
                </viewer>
              </span>
            </template>
          </el-table-column>
          <el-table-column v-if="torole=='super_editor'" label="操作" width="200" align="center">
            <template slot-scope="scope">
              <el-button size="mini" type="warning" @click="handleEdit(0,scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div v-if="pageShow" style="text-align: center;margin-top: 30px;">
          <BasePagination :totalCount="total" :nowPage.sync="currentPage"></BasePagination>
        </div>
      </div>
    </el-card>
    <!-- 弹窗部分 -->
    <div v-if="editVisible">
      <el-dialog :title="dialogTitle+'仓库进设备记录'" :visible.sync="editVisible" class="dialog_part">
        <el-form :model="editForm" :rules="rules" ref="editForm" label-position="right" :label-width="formLabelWidth">
          <el-row type="flex" class="row-bg" justify="left">
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="设备名称" prop="shebeimingcheng">
                  <el-input v-model="editForm.shebeimingcheng" autocomplete="off" placeholder="请输入设备名称"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple-light"></div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="日期" prop="riqi">
                  <el-date-picker v-model="editForm.riqi" type="date" value-format="yyyy-MM-dd" placeholder="选择日期"
                    style="float: left;width: 190px;margin-right: 10px;"></el-date-picker>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row type="flex" class="row-bg" justify="left">
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="数量" prop="shuliang">
                  <el-input v-model="editForm.shuliang" autocomplete="off" placeholder="请输入数量"
                    @input="(value)=>{limitToNumberStr(value,'shuliang')}"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple-light"></div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="详情1" prop="xiangqing1">
                  <el-input v-model="editForm.xiangqing1" autocomplete="off" placeholder="请输入详情1"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row type="flex" class="row-bg" justify="left">
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="详情2" prop="xiangqing2">
                  <el-input v-model="editForm.xiangqing2" autocomplete="off" placeholder="请输入详情2"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple-light"></div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="详情3" prop="xiangqing3">
                  <el-input v-model="editForm.xiangqing3" autocomplete="off" placeholder="请输入详情3"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row type="flex" class="row-bg" justify="left">
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="归属人" prop="shebeiguishuren">
                  <el-input v-model="editForm.shebeiguishuren" placeholder="请输入设备归属人" autocomplete="off"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple-light"></div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple">
                <el-form-item label="备注" prop="remark">
                  <el-input v-model="editForm.remark" placeholder="请输入备注" autocomplete="off"
                    style="float: left;width: 190px;margin-right: 10px;" clearable></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row type="flex" class="row-bg" justify="left">
            <el-col :span="24">
              <div class="grid-content bg-purple">
                <el-form-item label="上传照片" prop="zhaopian">
                  <BaseImgUpload :zhaopian="editForm.zhaopian" @updateImg="handleUpImg"></BaseImgUpload>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content bg-purple-light"></div>
            </el-col>
          </el-row>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click=" editVisible = false">取 消</el-button>
          <el-button type="primary" @click="sureEdit">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>
import { ware } from "@/http/ware/ware";
// import { staffService } from "@/http/staff/staff";
import { getTorole } from "@/utils/auth";
import BasePagination from '@/components/BasePagination'
import BaseImgUpload from '@/components/BaseImgUpload'
export default {
  components: {
    BasePagination,
    BaseImgUpload
  },
  data() {
    return {
      fileList: [],
      imageLogo: this.$global.code_path,
      keywords: '',
      list: [],
      listLoading: false,
      addVisible: false, //是否显示增加记录
      editVisible: false, //是否显示编辑
      dialogTitle: undefined,
      zhaopian_file: undefined,
      zhaopian_name: undefined,
      tableHeaderList: [
        { label: '设备名称', prop: 'shebeimingcheng' },
        { label: '日期', prop: 'riqi' },
        { label: '数量', prop: 'shuliang' },
        { label: '详情1', prop: 'xiangqing1' },
        { label: '详情2', prop: 'xiangqing2' },
        { label: '详情3', prop: 'xiangqing3' },
        { label: '设备归属人', prop: 'shebeiguishuren' },
        { label: '备注', prop: 'remark' },
      ],
      options: [
        {
          label: '已扣除',
          value: 1
        },
        {
          label: '未扣除',
          value: 0
        },
      ],
      editForm: {
        zhaopian: ''
      },
      rules: {
        shebeimingcheng: [{ required: true, message: '请输入设备名称', trigger: 'blur' }],
        riqi: [{ required: true, message: '请选择日期', trigger: 'blur' }],
        shuliang: [{ required: true, message: '请输入数量', trigger: 'blur' }],
        xiangqing1: [{ required: true, message: '请输入详情1', trigger: 'blur' }],
        xiangqing2: [{ required: true, message: '请输入详情2', trigger: 'blur' }],
        xiangqing3: [{ required: true, message: '请输入详情3', trigger: 'blur' }],
        shebeiguishuren: [{ required: true, message: '请输入设备归属人', trigger: 'blur' }],
        remark: [{ required: true, message: '请输入备注', trigger: 'blur' }],
        zhaopian: [{ required: true, message: '请上传照片', trigger: 'blur' }],
      },
      formLabelWidth: "80px",
      type: 0, //工地付款记录
      category_id: 0,
      rowid: "",
      // 分页
      total: 0,
      pagesize: 10,
      currentPage: 1,
      pageShow: false,
      torole: "", //权限
      e_type: 1, //工地付款记录
      multipleSelection: [], //选择数组
      excel_path: this.$global.excel_path //文件下载前缀
    };
  },
  watch: {
    currentPage() {
      this.fetchData()
    }
  },
  mounted() {
    // this.category_id = this.$route.query.category_id;
    this.fetchData();
    let torole = eval(getTorole());
    this.torole = torole[0];
  },
  methods: {
    handleClear() {
      this.fetchData()
    },
    handleUpImg(file, name) {
      this.zhaopian_file = file
      this.editForm.zhaopian = name
    },
    //查询
    filterData() {
      this.currentPage === 1 ? this.fetchData() : (this.currentPage = 1)
    },
    //获取列表内容
    fetchData() {
      this.listLoading = true;
      let obj = {
        size: this.pagesize,
        page: this.currentPage,
        keywords: this.keywords,
      };
      ware.warehouseIn(obj).then(response => {
        if (response.code == 200) {
          this.list = response.data.lists;
          this.total = response.data.total;
          if (response.data.total == 0) {
            this.pageShow = false;
          } else {
            this.pageShow = true;
          }
          this.listLoading = false;
        } else {
          this.$message.error(response.info);
        }
      });
    },
    //打开编辑或新增
    handleEdit(flag, row) {
      if (flag) {
        // 新增
        this.dialogTitle = "新增"
        this.editForm = {
          zhaopian: ""
        }
        this.rowid = undefined
      } else {
        // 编辑
        this.dialogTitle = "编辑"
        this.rowid = row.id
        this.editForm = { ...this.editForm, ...row }
      }
      this.editVisible = true;
    },
    // 工具函数
    limitToNumberStr(valStr, name) {
      let tempValue = valStr.toString()
      this.editForm[name] = tempValue
        .replace(/[^0-9.]/g, '')
        .replace('.', '#*')
        .replace(/\./g, '')
        .replace('#*', '.')
    },
    updateObj(targetObj, dateObj) {
      const temp = {}
      Object.keys(targetObj).map((item, index) => {
        if (dateObj[item] !== undefined || dateObj[item] !== null) {
          temp[item] = dateObj[item]
        }
      })
      return temp
    },
    //删除
    handleDelete(row) {
      this.$confirm("此操作永久删除该记录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let obj = {
            id: row.id
          };
          ware
            .warehouseInDelete(obj)
            .then(response => {
              if (response.code == 200) {
                this.$message({
                  message: response.info,
                  type: "success"
                });
                this.fetchData();
              } else {
                this.$message.error(response.info);
              }
            })
            .catch(err => {
              console.log(err);
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //确定编辑
    sureEdit() {
      this.$refs['editForm'].validate((valid) => {
        if (valid) {
          let obj = {}, data = new FormData();
          // 编辑
          if (this.rowid) {
            obj = { id: this.rowid, ...this.editForm };
            obj.zhaopian_file = this.zhaopian_file
            for (let key in obj) {
              data.append(key, obj[key])
            }
            ware.warehouseInEdit(data).then(response => {
              if (response.code == 200) {
                this.$message({
                  message: response.info,
                  type: "success"
                });
                this.fetchData();
                this.editVisible = false;
              } else {
                this.$message.error(response.info);
              }
            });
          } else {
            // 新增
            obj = this.editForm
            obj.zhaopian_file = this.zhaopian_file
            for (let key in obj) {
              data.append(key, obj[key])
            }
            ware.warehouseInAdd(data).then(response => {
              if (response.code == 200) {
                this.$message({
                  message: response.info,
                  type: "success"
                });
                this.fetchData();
                this.editVisible = false;
              } else {
                this.$message.error(response.info);
              }
            });
          }
        }
      })

    },
    //下载任务文件事件
    downloadFilse(url, name) {
      const aLink = document.createElement("a"); //创建a链接
      aLink.style.display = "none";
      aLink.href = url;
      aLink.download = name;
      document.body.appendChild(aLink);
      aLink.click();
      document.body.removeChild(aLink); //点击完成后记得删除创建的链接
    },
    //批量选择
    handleSelectionChange(val) {
      let arr = [];
      val.map((item, index) => {
        arr.push(item.id);
      });
      this.multipleSelection = arr;
    },
    //导出文件
    exportExcel(flag) {
      let obj = {}
      // 部分导出
      if (!flag) {
        obj = { export_arr: this.multipleSelection, }
      }
      ware
        .warehouseInExport(obj)
        .then(response => {
          if (response.code == 200) {
            let url = `${this.excel_path}${response.data.path}`;
            let name = response.data.name;
            this.downloadFilse(url, name);
            this.$message({
              message: response.info,
              type: "success"
            });
          } else {
            this.$message({
              message: response.info,
              type: "error"
            });
          }
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
};
</script>
<style scoped lang='scss'>
.com_project {
  color: #333;
  padding: 20px;
  .pro_list {
    padding: 20px 0;
    .page_part {
      padding: 20px;
      float: right;
    }
    .no-appear {
      color: #fc7c7c;
    }
  }
  .dialog_part {
    width: 65%;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
  }
  ::v-deep .el-form {
    .el-form-item__error {
      min-width: 150px;
    }
  }
  .photo-style {
    width: 70px;
    height: 70px;
  }
}
</style>
<style lang="scss">
.el-table .cell {
  text-align: center;
}
</style>