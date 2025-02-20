<template>
    <div>
        <div class="search">
            <el-input placeholder="请输入专业名称" style="width: 200px" v-model="name"></el-input>
            <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
            <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
        </div>

        <div class="operation">
            <el-button type="primary" plain @click="handleAdd">新增</el-button>
            <el-button type="danger" plain @click="delBatch">批量删除</el-button>
        </div>

        <div class="table">
            <el-table :data="tableData" stripe  @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="序号" width="80" align="center" sortable></el-table-column>
                <el-table-column prop="name" label="专业名称" show-overflow-tooltip></el-table-column>
                <el-table-column prop="content" label="专业描述" show-overflow-tooltip></el-table-column>
                <el-table-column prop="collegeName" label="所属学院" show-overflow-tooltip></el-table-column>
                <el-table-column prop="score" label="学分限定" show-overflow-tooltip></el-table-column>

                <el-table-column label="操作" width="180" align="center">
                    <template v-slot="scope">
                        <el-button plain type="primary" @click="handleEdit(scope.row)" size="mini">编辑</el-button>
                        <el-button plain type="danger" size="mini" @click=del(scope.row.id)>删除</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <div class="pagination">
                <el-pagination
                        background
                        @current-change="handleCurrentChange"
                        :current-page="pageNum"
                        :page-sizes="[5, 10, 20]"
                        :page-size="pageSize"
                        layout="total, prev, pager, next"
                        :total="total">
                </el-pagination>
            </div>
        </div>


        <el-dialog title="信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
            <el-form label-width="100px" style="padding-right: 50px" :model="form" :rules="rules" ref="formRef">
                <el-form-item prop="name" label="专业名称">
                    <el-input v-model="form.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item prop="content" label="专业描述">
                    <el-input type="textarea" :rows="5" v-model="form.content" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item prop="score" label="学分限定">
                    <el-input v-model="form.score" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item prop="collegeId" label="所属学院">
                    <el-select v-model="form.collegeId" placeholder="请选择学院" style="width: 100%">
                        <el-option v-for="item in collegeData" :label="item.name" :value="item.id"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="fromVisible = false">取 消</el-button>
                <el-button type="primary" @click="save">确 定</el-button>
            </div>
        </el-dialog>


    </div>
</template>

<script>
    export default {
        name: "Speciality",
        data() {
            return {
                tableData: [],
                pageNum: 1,
                pageSize: 10,
                total: 0,
                name: null,
                fromVisible: false,
                form: {},
                user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
                rules: {
                    name: [
                        {required: true, message: '请输入专业名称', trigger: 'blur'},
                    ],
                },
                ids: [],
                collegeData: []
            }
        },
        created() {
            this.load(1)
            this.loadCollege()
        },
        methods: {
            loadCollege(){
                this.$request.get('/college/selectAll').then(res => {
                    if (res.code === '200') {
                        this.collegeData = res.data
                    } else {
                        this.$message.error(res.msg)
                    }
                })
            },
            handleAdd() {
                this.form = {}
                this.fromVisible = true
            },
            handleEdit(row) {
                this.form = JSON.parse(JSON.stringify(row))
                this.fromVisible = true
            },
            save() {
                this.$refs.formRef.validate((valid) => {
                    if (valid) {
                        this.$request({
                            url: this.form.id ? '/speciality/update' : '/speciality/add',
                            method: this.form.id ? 'PUT' : 'POST',
                            data: this.form
                        }).then(res => {
                            if (res.code === '200') {
                                this.$message.success('保存成功')
                                this.load(1)
                                this.fromVisible = false
                            } else {
                                this.$message.error(res.msg)
                            }
                        })
                    }
                })
            },
            del(id) {
                this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
                    this.$request.delete('/speciality/delete/' + id).then(res => {
                        if (res.code === '200') {
                            this.$message.success('操作成功')
                            this.load(1)
                        } else {
                            this.$message.error(res.msg)
                        }
                    })
                }).catch(() => {
                })
            },
            handleSelectionChange(rows) {
                this.ids = rows.map(v => v.id)
            },
            delBatch() {
                if (!this.ids.length) {
                    this.$message.warning('请选择数据')
                    return
                }
                this.$confirm('您确定批量删除这些数据吗？', '确认删除', {type: "warning"}).then(response => {
                    this.$request.delete('/speciality/delete/batch', {data: this.ids}).then(res => {
                        if (res.code === '200') {
                            this.$message.success('操作成功')
                            this.load(1)
                        } else {
                            this.$message.error(res.msg)
                        }
                    })
                }).catch(() => {
                })
            },
            load(pageNum) {
                if (pageNum) this.pageNum = pageNum
                this.$request.get('/speciality/selectPage', {
                    params: {
                        pageNum: this.pageNum,
                        pageSize: this.pageSize,
                        name: this.name,
                    }
                }).then(res => {
                    this.tableData = res.data?.list
                    this.total = res.data?.total
                })
            },
            reset() {
                this.name = null
                this.load(1)
            },
            handleCurrentChange(pageNum) {
                this.load(pageNum)
            },
        }
    }
</script>

<style scoped>

</style>
