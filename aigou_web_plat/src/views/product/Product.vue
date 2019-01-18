<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="名称" width="200" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="品牌" width="100" sortable>
			</el-table-column>
			<!--有点屌-->
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="state" label="状态" width="100" :formatter="formatState" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="120" sortable>
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间"  min-width="120" sortable>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="10" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="formVisible" :close-on-click-modal="false">
			<el-form :model="form" label-width="80px" :rules="formRules" ref="form">
				<el-form-item label="名称" prop="name">
					<el-input v-model="form.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副名称" prop="subName">
					<el-input v-model="form.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="类型" prop="productTypeId">
					<!--<el-input v-model="form.productTypeId" auto-complete="off"></el-input>-->
					<select-tree width="200" :options="productTypes" v-model="form.productTypeId" />
				</el-form-item>
				<el-form-item label="品牌" prop="brandId">

					<el-select v-model="form.brandId" placeholder="请选择">
						<el-option
								v-for="item in brands"
								:key="item.id"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="medias" prop="medias">
					<el-upload
							class="upload-demo"
							action="http://127.0.0.1:9527/services/common/upload"
							:on-preview="handlePreview"
							:on-remove="handleRemove"
							:file-list="fileList2"
							:on-success="handleSuccess"
							list-type="picture"
					       >
						<el-button size="small" type="primary">点击上传</el-button>
						<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
					</el-upload>
				</el-form-item>
				<el-form-item label="简介" prop="description">
					<el-input type="textarea" v-model="form.productExt.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="详情" prop="richContent">
					<!--<el-input  v-model="form.productExt.richContent" auto-complete="off"></el-input>-->
					<quill-editor
							v-model="form.productExt.richContent"
							ref="myQuillEditor"
							:options="editorOption"
							@blur="onEditorBlur($event)" @focus="onEditorFocus($event)"
							@change="onEditorChange($event)">
					</quill-editor>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="formVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
    import Vue from 'vue';
    import  VueQuillEditor from 'vue-quill-editor';
    // require styles 引入样式
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.bubble.css';

    Vue.use(VueQuillEditor);
    import SelectTree from '@/components/SelectTree.vue';
	export default {
	    components:{
	        SelectTree
		},
		data() { //数据
			return {
				filters: {
                    keyword: ''
				},
                editorOption:{},
				brands:[],
				productTypes:[],
				products: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列
                fileList2: [],
				formVisible: false,//编辑界面是否显示
				editLoading: false,
				formRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				form: {
					id: 0,
					name: '',
					subName: '',
					productTypeId: 0,
					brandId: '',
					//medias: []//,
					productExt:{}
                }
			}
		},
		methods: { //方法
            //性别显示转换
            formatState: function (row, column) {
                return row.state == 1 ? '上架' :"下架";
            },
            handleSuccess(response, file, fileList){
                //上传成功回调
				this.form.logo = file.response.resultObj;
			},
            handleRemove(file, fileList) {
                var filePath =file.response.resultObj;
                this.$http.delete("/common/del?filePath="+filePath)
					.then(res=>{
					    if(res.data.success){
                            this.$message({
                                message: '删除成功!',
                                type: 'success'
                            });
						}else{
                            this.$message({
                                message: '删除失败!',
                                type: 'error'
                            });
						}
					})
            },
            handlePreview(file) {
                console.log(file);
            },
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取用户列表
            getProducts() {
				let para = {
					page: this.page,
					keyword: this.filters.keyword
				};
				this.listLoading = true;

				this.$http.post("/product/product/json",para)
                    .then((res) => {
                        console.log(this);
                        this.total = res.data.total;
                        this.products = res.data.rows;
                        this.listLoading = false;
                    });
			},
			getBrands(){
                this.listLoading = true;
                this.$http.get("/product/brand/list")
                    .then((res) => {
                        this.brands = res.data;
                        this.listLoading = false;
                    });
			},
            getProductTypes(){
                this.$http.get("/product/productType/treeData")
                    .then(res=>{
                        this.productTypes = res.data;
                    });
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { id: row.id };
					removeUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getProducts();
					});
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.formVisible = true;
				//回显 要提交后台
				console.debug(row);
				this.form = Object.assign({}, row);
				//回显缩略图
				// this.fileList2.push({
				// 	"url":this.$staticIp+row.logo
				// })
			},
			//显示新增界面
			handleAdd: function () {
				this.formVisible = true;
				this.form = {
                    name: '',
                    subName: '',
                    productTypeId: 0,
                    brandId: '',
                    //medias: [],
                    productExt:{}
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.form.validate((valid) => {
					if (valid) {
                        console.log(this.fileList2);
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							let para = Object.assign({}, this.form);
							this.$http.post("/product/product/save",para).then((res) => {
								this.editLoading = false;
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['form'].resetFields();
								this.formVisible = false;
								this.getProducts();
							});
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getProducts();
					});
				}).catch(() => {

				});
			},
            onEditorBlur(){//失去焦点事件
            },
            onEditorFocus(){//获得焦点事件
            },
            onEditorChange(){//内容改变事件
            }
		}, // $(function()) 加载完毕后执行
		mounted() {
			this.getProducts();
			this.getBrands();
			this.getProductTypes();
		}
	}

</script>

<style scoped>

</style>