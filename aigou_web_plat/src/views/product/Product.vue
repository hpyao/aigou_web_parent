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
				<el-form-item>
					<el-button type="primary" @click="handleConfigViewProperties">配置显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleConfigSkuProperties">配置sku属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOnsale">上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOffSale">下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @row-click="rowClick" @selection-change="selsChange" style="width: 100%;">
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
		<!--显示属性对话框界面-->
		<el-dialog title="配置显示属性" v-model="viewPropertiesVisible" :close-on-click-modal="false">
			<el-card class="box-card">
				<div v-for="viewProperty in viewProperties" :key="viewProperty" class="text item">
					<el-row :gutter="20">
						<el-col :span="4">{{viewProperty.name}}:</el-col>
						<el-col :span="20"><el-input auto-complete="off" v-model="viewProperty.selectValue" style="width: 200px"></el-input></el-col>
					</el-row>
				</div>
			</el-card>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="viewPropertiesVisible = false">取消</el-button>
				<el-button type="primary" @click.native="viewPropertiesSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--sku属性对话框界面-->
		<el-dialog title="配置sku属性" v-model="skuPropertiesVisible" :close-on-click-modal="false">
			<el-card class="box-card">
				<!--属性选择-->
				<div v-for="skuProperty in skuProperties" :key="skuProperty" class="text item">
					{{skuProperty.name}}:
					<!--
					   1 以后skuValue如果有多个需要设置多个input
					   2 输入一行后要要能自动加入下一行.
					     比如 v-for="i in 10"，你会得到1~10   v-for="i in 2" 1~2
					-->
					<div v-for="i in skuProperty.skuValues.length+1" :key="i" class="text item">
						<el-input auto-complete="off" v-model="skuProperty.skuValues[i-1]" style="width: 400px"></el-input>
						<!--删除i-1(i从1开始,而索引是从零开始)这一个-->
						<el-button type="danger" @click="skuProperty.skuValues.splice(i-1,1)" icon="el-icon-delete" plain>删除</el-button>
					</div>
				</div>
			</el-card>
			<!--动态表格-->
			<el-table :data="skuDatas">
				<!--cols一堆列,col哪一列-->
				<template v-for="(col ,index) in cols">
					<el-table-column :prop="col.prop" sortable :label="col.label" v-if="['price','state','stock'].includes(col.prop)">
						<template scope="scope">
							<el-input auto-complete="off" v-model="skuDatas[scope.$index].price"  style="width: 400px" v-if="'price'===col.prop"/>
							<el-input auto-complete="off" v-model="skuDatas[scope.$index].stock" style="width: 400px" v-if="'stock'===col.prop"/>
							<el-checkbox v-model="skuDatas[scope.$index].state" v-if="col.prop==='state'">备选项</el-checkbox>
						</template>
					</el-table-column>
					<!--只做显示-->
					<el-table-column :prop="col.prop" sortable :label="col.label" v-if="!['price','state','stock'].includes(col.prop)">
					</el-table-column>
					<!--<el-table-column :prop="col.prop" sortable :label="col.label">-->
					<!--</el-table-column>-->
				</template>
			</el-table>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="skuPropertiesVisible = false">取消</el-button>
				<el-button type="primary" @click.native="skuPropertiesSubmit" :loading="editLoading">提交</el-button>
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
			    skuIsNull:true,
                skuDatas:[],
                cols:[],
				filters: {
                    keyword: ''
				},
                editorOption:{},
				viewProperties:[],
                skuProperties:[],
				brands:[],
				productTypes:[],
				products: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列
                fileList2: [],
				formVisible: false,//编辑界面是否显示
                viewPropertiesVisible:false,
                skuPropertiesVisible:false,
				curentRow: null,
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
            viewPropertiesSubmit(){
                //Map{"procutId":1,viewPro perties:[]}
                let productId = this.curentRow.id;
                let params = {"productId":productId,"specifications":this.viewProperties};
                this.$http.post("/product/product/addViewProperties",params)
					.then(res=>{
					    let result = res.data;
					    if(result.success){
                            this.$message({
                                message: '设置成功!',
                                type: 'success'
                            });
                            this.viewPropertiesVisible = false;
						}
						else{
                            this.$message({
                                message: result.message,
                                type: 'error'
                            });
						}
					})
			},
            handleConfigViewProperties(){
				//查询显示属性
				//0 是否选中行
				if(!this.curentRow){
                    this.$message({
                        message: '请选择一行后,再操作!',
                        type: 'warning'
                    });
                    return;
				}
                this.viewPropertiesVisible =true;
				//1 获取当前行productId
				let productId = this.curentRow.id;
				//2 发送请求获取显示属性数据
				this.$http.get("/product/specification/product/"+productId)
					.then(res=>{
                        this.viewProperties = res.data;
					})
				//3 界面双向绑定数据
	        },
            skuPropertiesSubmit(){
                //Map{"procutId":1,viewPro perties:[]}
                let productId = this.curentRow.id;
                let params = {"productId":productId,"skuProperties":this.skuProperties,"skuDatas":this.skuDatas};
                this.$http.post("/product/product/addSkus",params)
                    .then(res=>{
                        let result = res.data;
                        if(result.success){
                            this.$message({
                                message: '设置成功!',
                                type: 'success'
                            });
                            this.skuPropertiesVisible = false;
                        }
                        else{
                            this.$message({
                                message: result.message,
                                type: 'error'
                            });
                        }
                    })
			},
            handleConfigSkuProperties(){
				//查询显示属性
                //0 是否选中行
                if(!this.curentRow){
                    this.$message({
                        message: '请选择一行后,再操作!',
                        type: 'warning'
                    });
                    return;
                }
                this.skuPropertiesVisible =true;
                //1 获取当前行productId
                let productId = this.curentRow.id;
                //2 发送请求获取显示属性数据
                this.$http.get("/product/specification/product/skusProperties/"+productId)
                    .then(res=>{
                        this.skuProperties = res.data;
                    })
				//查询sku值做回显
                this.$http.get("/product/product/skus/"+productId)
                    .then(res=>{
                        if( this.skuDatas.length > 0)
						{
						    this.skuIsNull = false;
						}
                        this.skuDatas = res.data;

                    })
            },
            handleOnsale(){

            },
            handleOffSale(){

            },
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
                        //console.log(this);
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
				//console.debug(row);
				this.form = Object.assign({}, row);
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
            rowClick: function (row,event,column) {
                console.log(row);
                console.log(row.productTypeId);
                this.curentRow = row;

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
		},
		mounted() {
			this.getProducts();
			this.getBrands();
			this.getProductTypes();
		},
		//件事属性值变化
		watch:{
            skuProperties:{
                handler(curVal,oldVal){
                    //1 确实是第一次从数据库查询值
					//2 第二次修改属性
                    if(!this.skuIsNull){ //不为null
						console.log("xxx",this.skuIsNull);
                        this.skuIsNull = true;
					}else{
                        console.log("jjjj",this.skuProperties);
                        // 过滤掉用户没有填写数据的规格参数
                        const arr = this.skuProperties.filter(s => s.skuValues.length > 0);
                        // 通过reduce进行累加笛卡尔积
                        var skus =  arr.reduce((last, spec) => {
                            const result = [];
                            last.forEach(o => {
                                spec.skuValues.forEach(option => {
                                    // option //一个一一个值 黄皮肤
                                    const obj = {};
                                    Object.assign(obj, o);
                                    obj[spec.name] = option;
                                    result.push(obj);
                                })
                            })
                            return result
                        }, [{}]);
                        //假数据测试是否能够绑定数据
                        skus.forEach(function (item) {
                            item['price'] = '';
                            item['stock'] = '';
                            item['state'] = 0;
                        })
                        this.skuDatas = skus;
                        console.log(skus);
					}
                    let headers = [];
                    //现在没有一定有字段 库存 价格 是否可用 颜色
                    //skus [{"身高":170,"三维":"xxx",价格:18,库存:18,是否可用:0},{"身高":170,"三维":"xxx",价格:18,库存:18,是否可用:0}]
					Object.keys(this.skuDatas[0]).forEach(sku=>{
					    let value = sku;
					    if(sku=='price'){
                            value = '价格'
						}
					    if(sku=='stock'){
                            value = '库存'
						}
					    if(sku=='state'){
                            value = '启用'
						}
                        let col =  {"label":value,"prop":sku};
                        headers.push(col);
					});
					this.cols = headers;
                },
                deep:true
            }
		}
	}

</script>

<style scoped>

</style>