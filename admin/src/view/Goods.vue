<template>
  <container>
    <!-- 商品查询 -->
    <el-form :inline="true" :model="query" ref="query" class="goods_query_form">
      <el-form-item prop="id">
        <el-input v-model.number="query.id" placeholder="商品ID"/>
      </el-form-item>
      <el-form-item prop="id">
        <el-input v-model.number="query.name" placeholder="商品名称"/>
      </el-form-item>
      <el-form-item prop="status">
        <el-select v-model="query.status" placeholder="商品状态">
          <el-option label="已上架" value="0"></el-option>
          <el-option label="未上架" value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" :icon="Search" @click="searchGoods">查询</el-button>
        <el-button type="primary" :icon="Brush" @click="resetForm('query')"/>
        <el-button type="primary" :icon="Plus" @click="add">商品</el-button>
      </el-form-item>
    </el-form>
    <!-- 商品列表 -->
    <el-table :data="goodsList" height="65vh" style="width: 100%;">
      <el-table-column prop="id" label="商品id" sortable/>
      <el-table-column prop="name" label="商品名称" min-width="160">
        <template #default="scope">
          <div style="display: inline-flex;">
            <!--            <div class="goods_image_box">-->
            <!--              <el-image class="goods_image" :src="scope.row.imageUrl"/>-->
            <!--            </div>-->
            <div style="display: block;">
              <div class="goods_title">{{ scope.row.name }}</div>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="price" label="价格" sortable/>
      <el-table-column prop="discount" label="折扣" sortable/>
      <el-table-column prop="sales" label="销量" sortable/>
      <el-table-column prop="deleted" label="状态">
        <template #default="scope">
          <el-tag v-if="scope.row.deleted === 0" size="small">已上架</el-tag>
          <el-tag v-if="scope.row.deleted === 1" size="small" type="success">已下架</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="categoryId" label="类目" sortable/>

      <el-table-column prop="createTime" label="创建时间" min-width="150" sortable>
        <template #default="scope">
          <div style="display: flex; align-items: center">
            <el-icon>
              <timer/>
            </el-icon>
            <span style="margin-left: 10px">{{ scope.row.createTime }}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="操作" min-width="130">
        <template #default="scope">
          <el-button size="small" :icon="Edit" @click="edit(scope.$index, scope.row)"/>
          <el-button v-if="scope.row.status === 2" size="small" type="primary" @click="changeStatus(1,scope.row)">上架
          </el-button>
          <el-button v-if="scope.row.status === 1" size="small" type="primary" @click="changeStatus(2,scope.row)">下架
          </el-button>
          <el-popconfirm title="此操作将永久删除该信息, 是否继续?"
                         confirmButtonText="确认"
                         cancelButtonText="取消"
                         cancelButtonType="default"
                         :icon="WarningFilled"
                         @confirm="deleteGoods(scope.row)">
            <template #reference>
              <el-button size="small" type="danger" :icon="Delete"/>
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
      <template #empty>
        <div style="margin: 50px 0;">
          <el-empty v-if="showEmpty" description="暂时还没有商品哦"/>
        </div>
      </template>
    </el-table>
    <div style="padding: 15px 0;">
      <el-pagination layout="total, prev, pager, next"
                     :current-page="pageNum"
                     :page-size="pageSize"
                     :total="total"
                     @current-change="handleCurrentChange"
                     @prev-click="handleCurrentChangePrev"
                     @next-click="handleCurrentChangeNext" background/>
    </div>
    <!-- 添加、编辑商品，通用对话框 -->
    <el-dialog :title="dialogTitle" v-model="goodsDialogVisible" :lock-scroll="false" top="5vh" width="45%" @close="cancel">
      <el-form ref="ruleForm" :rules="rules" :model="goods" label-width="80px">
        <el-form-item label="类目" prop="categoryId">
          <el-select
              size="small"
              v-model="goods.categoryId" ct
              placeholder="请选择"
              @change="addCatagory"
              clearable>
            <el-option
                v-for="item in option"
                :key="item.value"
                :label="item.label"
                :value="item.value"
            >
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="名称" prop="name">
          <el-input v-model="goods.name" style="width: 50%;" type="text" maxlength="10" show-word-limit/>
        </el-form-item>
        <el-form-item label="价格" prop="price">
          <el-input v-model.number="goods.price" style="width: 50%;">
            <template #append>元</template>
          </el-input>
        </el-form-item>
        <el-form-item label="销量" prop="sales">
          <el-input v-model.number="goods.sales" style="width: 50%;">
            <template #append>件</template>
          </el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="goods.remark"
                    style="width: 90%;"
                    type="textarea"
                    maxlength="100"
                    :autosize="{ minRows: 2}" show-word-limit/>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="cancel">取消</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
          <el-button type="primary" @click="confirmGoods">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </container>
</template>

<script>
import {ArrowDown, ArrowUp, Brush, Delete, Edit, Plus, Search, Timer, WarningFilled} from "@element-plus/icons-vue";
import {ElMessage} from 'element-plus'
import container from "../components/container";

export default {
  name: "Goods",
  components: {container, Timer,},
  setup() {
    return {Search, Brush, Plus, Edit, ArrowUp, ArrowDown, Delete, WarningFilled}
  },
  data() {
    return {

      // 查询条件
      query: {
        id: '',
        name: '',
        status: ''
      },

      // 类目选项
      categoryOption: [],

      option: [
        {
          value: "01",
          label: "一级父类",
        },
        {
          value: "02",
          label: "二级父类"
        }
      ],


      // 商品
      goodsList: [],
      goods: {
        id: '',
        categoryId: '',
        name: '',
        price: '',
        sales: '',
        discount: '',
        imageUrl: '',
        remark: '',
        deleted: '',
        createTime: ''
      },

      rules: {
        categoryId: {required: true, message: '请选择一个类目', trigger: 'blur'},
        name: {required: true, message: '请输入一个名称', trigger: 'blur'},
        price: [{required: true, message: '价格不能为空', trigger: 'blur'}, {
          type: 'number',
          message: '价格必须为数字',
          trigger: 'blur'
        }],
        sales: [{required: true, message: '数量不能为空', trigger: 'blur'}, {
          type: 'number',
          message: '数量必须为数字',
          trigger: 'blur'
        }],

      },

      // 图片列表
      pictureList: [],

      // 分页
      total: 0,
      pageNum: 1,
      pageSize: 10,

      // 请求认证
      token: '',

      // 操作状态
      dialogTitle: '',
      operateType: '',
      goodsDialogVisible: false,

      // 空状态
      showEmpty: false
    }
  },
  mounted() {
    this.getGoodsList()
    this.token = localStorage.getItem('token')
    this.goodsDialogVisible = this.$route.params.dialog
  },
  methods: {

    // 分页，处理函数
    handleCurrentChangePrev(val) {
      this.pageNum = val;
      console.log(`上一页: ${val}`);
    },
    handleCurrentChange(val) {
      this.pageNum = val;
      this.getGoodsList();
      console.log(`当前页: ${val}`);
    },
    handleCurrentChangeNext(val) {
      this.pageNum = val;
      console.log(`下一页: ${val}`);
    },

    // 图片上传，处理函数
    handleImagePreview(file) {
      this.goods.imageUrl = file.url
    },
    handleImageRemove(file, fileList) {
      this.goods.imageUrl = ''
      console.log(file, fileList)
    },
    handleImageSuccess(response) {
      this.goods.imageUrl = response.data
    },

    // 添加、编辑商品
    add() {
      this.empty()
      this.dialogTitle = '添加商品'
      this.operateType = 'add'
      this.goodsDialogVisible = true
    },
    edit(index, row) {
      this.dialogTitle = '编辑商品'
      this.operateType = 'edit'
      this.goods.id = row.id
      this.goods.categoryId = row.categoryId
      this.goods.name = row.name
      this.goods.price = row.price
      this.goods.sales = row.sales
      this.goods.imageUrl = row.imageUrl
      this.pictureList.push({url: row.imageUrl})
      this.goods.remark = row.remark
      this.goods.status = row.status
      this.goodsDialogVisible = true
    },

    changeCategory(value) {
      this.goods.categoryId = value[1]
    },


    addCatagory() {
      console.log("种类:", this.goods.categoryId);
    },
    // 获取商品列表
    getGoodsList() {
      this.$axios.get('/goods', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          sid: parseInt(localStorage.getItem('sid'))
        }
      }).then((response) => {
        this.total = response.data.data.total;
        this.goodsList = response.data.data.list;
        console.log(this.goodsList)
        if (this.goodsList.length === 0) {
          this.showEmpty = true
        }
      }).catch((error) => {
        console.log(error)
      })
    },

    // 商品上架、下架
    changeStatus(status, row) {
      this.$axios.put('/goods/status/update', {
        id: row.id,
        status: parseInt(status)
      }).then((response) => {
        if (response.data.code === 200) {
          ElMessage({message: response.data.message, type: 'success'})
        }
        this.getGoodsList()
      }).catch((error) => {
        console.log(error)
      })
      this.getGoodsList()
    },

    // 确认添加或编辑商品
    confirmGoods() {
      this.$refs['ruleForm'].validate((valid) => {
        if (valid) {
          if (this.operateType === 'add') {
            // 添加商品
            this.$axios.post('/goods/add', {
              categoryId: parseInt(this.goods.categoryId),
              name: this.goods.name,
              price: parseInt(this.goods.price),
              sales: parseInt(this.goods.sales),
              imageUrl: this.goods.imageUrl,
              remark: this.goods.remark,
              sid: parseInt(localStorage.getItem('sid'))
            }).then((response) => {
              if (response.data.code === 200) {
                ElMessage({message: response.data.message, type: 'success'})
              }
              this.getGoodsList()
            }).catch((error) => {
              console.log(error)
            })
          }
          if (this.operateType === 'edit') {
            // 编辑商品
            this.$axios.put('/goods/update', this.goods).then((response) => {
              if (response.data.code === 200) {
                ElMessage({message: response.data.message, type: 'success'})
              }
              this.getGoodsList()
            }).catch((error) => {
              console.log(error)
            })
          }
          this.goodsDialogVisible = false
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },

    //查询商品
    // querryGoods() {
    //   this.$axios.get('/goods', {
    //     params: {
    //       id: this.query.id,
    //       categoryId: this.query.categoryId[1],
    //       status: this.query.status,
    //       pageNum: this.pageNum,
    //       pageSize: this.pageSize,
    //       sid: parseInt(localStorage.getItem('sid'))
    //     }
    //   }).then((response) => {
    //     this.total = response.data.data.total;
    //     this.goodsList = response.data.data.list;
    //     console.log(this.goodsList)
    //     if (this.goodsList.length === 0) {
    //       this.showEmpty = true
    //     }
    //   }).catch((error) => {
    //     console.log(error)
    //   })
    // },
    // 删除商品
    deleteGoods(row) {
      this.$axios.delete('/goods/delete', {
        params: {
          id: row.id
        }
      }).then((response) => {
        if (response.data.code === 200) {
          ElMessage({message: response.data.message, type: 'success'})
        }
        this.getGoodsList()
      }).catch((error) => {
        console.log(error)
      })
    },

    // 重置表单
    resetForm(formName) {
      this.$refs[formName].resetFields()
      switch (formName) {
        case 'query':
          this.getGoodsList();
          break;
        case 'ruleForm':
          this.empty();
          break;
        default:
          console.log('invalid resetForm');
      }
    },

    // 取消
    cancel() {
      this.$refs['ruleForm'].resetFields()
      this.empty()
      this.goodsDialogVisible = false
    },

    // 清空数据
    empty() {
      this.goods.id = ''
      this.goods.categoryId = ''
      this.goods.name = ''
      this.goods.price = ''
      this.goods.sales = ''
      this.goods.picture = ''
      this.goods.remark = ''
      this.goods.status = ''
      this.operateType = ''
      this.pictureList = []
    },
    searchGoods() {
      this.$axios.post('/goods/search', {
          id: this.query.id,
          deleted: this.query.status,
          name:this.query.name,
          pageNum: this.pageNum,
          pageSize: this.pageSize,
      }).then((response) => {
        this.total = response.data.data.total;
        this.goodsList = response.data.data.list;
        console.log(this.goodsList)
        if (this.goodsList.length === 0) {
          this.showEmpty = true
        }
      }).catch((error) => {
        console.log(error)
      })
    },

  }
}
</script>
<style>
.goods_query_form {
  margin: 6px 0;
  display: inline-flex;
}

.goods_image_box {
  margin: 4px 5px 5px 0;
}

.goods_image_box, .goods_image {
  width: 62px;
  height: 62px;
  border-radius: 5px;
}

.goods_title {
  font-size: 13px;
  font-weight: 450;
  margin: 2px 3px 1px;
}

.goods_id {
  margin: 0 3px;
  font-size: 12px;
  font-weight: 450;
}

.goods_image_upload_icon {
  color: #9aa9b9;
  font-size: 24px;
  font-weight: 200;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.el-dialog {
  border-radius: 10px !important;
}

.el-upload {
  width: 100px;
  height: 100px;
  line-height: 100px;
  font-size: 16px;
}

.el-upload--picture-card {
  background-color: #F2F4F7;
  border: none;
}

.el-upload--picture-card {
  width: 100px;
  height: 100px;
  margin-top: 0;
  font-size: 16px !important;
}

.el-upload-list--picture-card .el-upload-list__item {
  width: 100px;
  height: 100px;
  line-height: 100px;
  font-size: 16px;
}

.el-upload-list--picture-card .el-upload-list__item-thumbnail {
  width: 100px;
  height: 100px;
  line-height: 100px;
  font-size: 16px;
}

.el-table__header tr, .el-table__header th {
  background-color: #eff6ff !important;
  font-weight: 450 !important;
  color: #627a94 !important;
  border-bottom: none !important;
}
</style>