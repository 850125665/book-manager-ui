<template>
    <div>
        <!--工具栏-->
        <el-form :inline="true" :model="formInline" size="mini" class="toolbar">
            <el-form-item label="关键字" >
                <el-input v-model="formInline.keyword" placeholder="书籍名称/BookID"></el-input>
            </el-form-item>
           <el-form-item label="分类" prop="dicCategory">
                <el-select v-model="formInline.dicCategory" clearable placeholder="请选择" style="width:120px">
                    <el-option v-for="item in categoryOptions" :key="item.id" :label="item.text" :value="item.id"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="频道" prop="dicChannel">
                <el-select v-model="formInline.dicChannel" clearable placeholder="请选择" style="width:120px">
                    <el-option v-for="item in channelOptions" :key="item.id" :label="item.text" :value="item.id"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSearch">查询</el-button>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="gotoAdd" >新增</el-button>
            </el-form-item>
        </el-form>
        <!--表格区-->
        <el-table :data="tableData" border style="width: 100%;" size="small">
            <template slot="empty">
                还没有数据呢~ (⊙︿⊙)
            </template>
            <el-table-column prop="bookId" label="图书ID" width="100">
            </el-table-column>
            <el-table-column prop="bookName" label="图书名称" >
            </el-table-column>
            <el-table-column prop="authorName" label="作者" width="100" >
            </el-table-column>
            <el-table-column prop="keyWord" label="标签" >
            </el-table-column>
            <el-table-column align="center" label="操作" width="240">
            <template slot-scope="scope" >
                <el-button size="mini" type="primary" plain 
                @click="handleEdit(scope.row.id)">编辑</el-button>
                <el-button size="mini" type="danger"  plain 
                @click="handleDelete(scope.row.id)">删除</el-button>
            </template>
            </el-table-column>
        </el-table>
        <!--分页区-->
        <div class="Pagination" style="text-align: left; margin-top: 10px;">
            <el-pagination
                background
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="currentPage"
                :page-size="limit"
                layout="total, prev, pager, next"
                :total="total">
            </el-pagination>
        </div>
    </div>
</template>

<script>
  export default {
    data() {
        return {
            categoryOptions:[],
            channelOptions:[],
            tableData: [],
            limit: 10,
            total: 0,
            currentPage: 1,
            formInline:{
                keyword:'',
                dicCategory:'',
                dicChannel:''
            }
        }
    },
    created(){
        this.getListData();
        this.getDictionaryOptions("category");
        this.getDictionaryOptions("channel");
    },
    methods:{
        handleEdit(id) {
            this.$router.push('/book/book-edit/'+id);
        },
        handleDelete(id) {
            this.$confirm('确定要删除吗?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.deleteRequest('/ms-book-manager/api/book/delete',{id:id}).then(resp => {
                    if (resp == "") {
                        this.getListData();
                    }
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消操作'
                });
            });
            
        },
        getDictionaryOptions(type){
            // 字典数据源
            this.getRequest('/ms-book-manager/api/dataDictionary/getSelectOptions', {type:type}).then(resp => {
                if (resp && !resp.status) {
                    if (type == "category"){
                        // 分类
                        this.categoryOptions = resp;
                    } else if (type == "channel"){
                        // 所属频道
                        this.channelOptions = resp;
                    } else if (type == "serial_status"){
                        // 连载状态
                        this.serialStatusOptions = resp;
                    }
                }
            })
        },
        getListData(){
            let form = {
                pageNum:this.currentPage,
                pageSize:this.limit,
                dicCategory:this.formInline.dicCategory,
                dicChannel:this.formInline.dicChannel,
                bookName:this.formInline.keyword
            }
            this.getRequest('/ms-book-manager/api/book/getList', form).then(resp => {
                if (resp && !resp.status) {
                    this.tableData = resp.list;
                    this.total = resp.total;
                }
            })
        },
        handleSizeChange(val) {
            console.log(`每页 ${val} 条`);
        },
        handleCurrentChange(val) {
            this.currentPage = val;
            this.getListData();
        },
        onSearch(){
            this.getListData();
        },
        gotoAdd(){
            this.$router.push("/book/book-add");
        }
    }
  };
</script>

<style scoped>
    .toolbar{
        height: 40px;
    }
</style>