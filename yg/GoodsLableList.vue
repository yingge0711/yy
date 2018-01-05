<!-- 标签管理==标签列表-->
<template>
	<div>
		<!-- 添加编辑页面 -->
		<el-dialog title="标签管理" :visible.sync="dialogVisible" width="30%">

		

			<el-form :model="formData" ref="formData" label-width="130px" :rules="rules" style="width:300px;">
				<el-form-item label="标签名称" prop="tagContent">
					<el-input v-model="formData.tagContent" auto-complete="off" style="width:180px;"></el-input>
				</el-form-item>
				<el-form-item label="标签描述" prop="tagRemark">

					<el-input v-model="formData.tagRemark" auto-complete="off" style="width:180px;"></el-input>
				</el-form-item>
				<el-form-item label="标签分类：" label-width="130px">
					<el-select v-model="formData.typeName" style="width:180px;">
						<el-option v-for="item in formData.parseList" :key="item.typeId" :label="item.typeName" :value="item.typeId"></el-option>
					</el-select>
				</el-form-item>

				<el-form-item label="format：" label-width="130px">
					<el-input v-model="formData.format" auto-complete="off" style="width:180px;"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button type="primary" @click="save">确 定</el-button>
			</div>
		</el-dialog>
		<!--模糊查询 -->
		<div class="searchForm">
			<el-form style="width:80%" :model="searchForm" ref="ruleForm" label-width="90px" class="demo-ruleForm">
				<el-row :gutter="30">
					<el-col :span="5">
						<div class="grid-content bg-purple">
							<el-form-item label="标签名称" prop="name">
								<el-input v-model="searchForm.keywords"></el-input>
							</el-form-item>
						</div>
					</el-col>
					<el-col :span="6">
						<el-button type="primary" @click="search">查询</el-button>
					</el-col>
				</el-row>
			</el-form>
		</div>
		<el-button size="small" class="orange" @click="addlable('add')" style="margin-top:10px;"><i class="el-icon-plus"></i>添加标签</el-button>

		<!--  列表  -->
		<div class="tableBox" style="margin-top: 50px;">
			<!-- tableData 数据绑定 -->
			<el-table :data="tableData" style="width: 100%;">
				<el-table-column align="center" prop="tagContent" label="标签名称"></el-table-column>
				<el-table-column align="center" prop="tagId" label="标签ID"></el-table-column>
				<el-table-column align="center" prop="tagRemark" label="备注"></el-table-column>
				<el-table-column align="center" prop="typeName" label="标签分类"></el-table-column>
				<el-table-column align="center" label="操作">
					<template slot-scope="scope">
						<a style="cursor:pointer; color:#1197ff;" @click="editorRow(scope.row.tagId,'edit')">编辑</a>
						<a style="cursor:pointer; color:#1197ff;" @click="deleteInfo(scope.row.tagId)">删除</a>
					</template>
				</el-table-column>
			</el-table>
			<!-- 分页按钮 -->
			<div class="pageArea">
				<el-pagination @current-change="queryTotal" :current-page.sync="currentPage" :page-count="pageCount" layout="prev, pager, next, jumper"></el-pagination>
			</div>
		</div>
	</div>
</template>

<script type="text/javascript">
	export default {
		//配置的参数
		data() {
				return {
					dialogVisible: false,
					formData: {},
					currentPage: 1, //当前页
					editType: '', //判断当前的类型(编辑、添加)
					pageCount: 1, //总页数
					tableData: [],
					searchForm: {}, //模糊查询 
					rules:{
						tagContent:[],
						tagRemark:[]
					} 
				}
			},
			//生命周期（页面起初加载的）
			mounted() {
				this.queryTotal();
				//标签分类
				this.getSelectList();
			},
			//进行数据交互的方法
			methods: {
				//标签的分类
				getSelectList() {
					this.$http({
						url: this.apiJSON.lable_type,
						method: 'get',
					}).then((data) => {
						if (data) {
							this.formData.parseList = data;
							console.log(this.formData.parseList);
						}
					});
				},
				//模糊查询
				search() {
					if (Object.keys(this.searchForm).length > 0) {
						let searObj = Object.assign({}, this.searchForm);
						for (let i in searObj) {
							if (typeof searObj[i] == 'object' && searObj[i]) {
								searObj[i] = this.pubMethod.FormatDate(searObj[i]);
							}
						}
						this.searObj = searObj;
						this.queryTotal(searObj);
					}
				},
				//查看 分页
				queryTotal(value) {
					let queryData = {
						order: 'asc',
						offset: '0',
						limit: '10',
						pageSize: '10',
						pageNum: this.currentPage,
					}
					queryData = Object.assign(queryData, this.searObj);
					this.$http({
						url: this.apiJSON.lable_list,
						method: 'get',
						params: queryData
					}).then((res) => {
						this.tableData = res.list;
						this.pageCount = res.pages; //把当前页数赋值
					})
				},
				//添加按钮弹框
				addlable(type) {
					this.dialogVisible = true;
					this.editType = type;
				},
				//编辑通过ID查询数据,渲染数据
				editorRow(tagId, type) {
					this.editType = type;
					this.$http({
						url: this.apiJSON.lable_ByIdInfo,
						method: 'get',
						params: {
							tagId: tagId
						}
					}).then((data) => {
						this.formData = Object.assign(this.formData, data);
						this.dialogVisible = true;
						this.formData.tagId = data.tagId;
					});
				},
				//添加标签
				save(type) {
						//编辑
					if (this.editType == 'edit') {
						//编辑封装的参数
						let editformData;
						let {tagId,tagContent,tagRemark,typeName,format} = this.formData;
						editformData = {tagId,tagContent,tagRemark,typeName,format};
						//参数名称替换	
						editformData.type = editformData.typeName;
						delete editformData.typeName;
						this.$http({
							url: this.apiJSON.lable_editor,
							method: 'post',
							data: editformData
						}).then((data) => {
							if (data) {
								this.$message({
									type: 'success',
									message: '修改成功!'
								});
								this.queryTotal();
								this.dialogVisible = false;
							}
						});
						//添加
					} else if (this.editType == 'add') {
						//通过编辑渲染的数据,在二次编剧的时候那些参数全部会传传过去,一些参数不需要,就把参数二次处理                            
						let formDatas;
						// console.log(this.formData);
						// return false;
						let {tagContent,tagRemark,typeName,format} = this.formData;
						formDatas = {tagContent,tagRemark,typeName,format};
						//参数名称替换	
						formDatas.type = formDatas.typeName;
						delete formDatas.typeName;
						this.$http({
							url: this.apiJSON.lable_save,
							method: 'post',
							data: formDatas
						}).then((data) => {
							if (data) {
								this.$message({
									type: 'success',
									message: '添加成功!'
								});
								this.queryTotal();
								this.dialogVisible = false;
							}
						});
					}
				},
				//删除
				deleteInfo(tagId) {
					let url = '',
						queryData = {
							tagId: tagId
						};
					url = this.apiJSON.lable_delete;
					this.$confirm('确认要删除吗？?', '提示', {
						confirmButtonText: '确定',
						cancelButtonText: '取消',
						type: 'warning'
					}).then(() => {
						this.$http({
							url: url,
							method: 'post',
							params: queryData
						}).then((data) => {
							if (data) {
								this.$message({
									type: 'success',
									message: '删除成功!'
								});
								this.queryTotal(); //调取查询分页的方法
							}
						});
					})

				},
				//添加按钮弹框
				addlable(type) {
					if(this.$refs['formData']!==undefined){
						this.$refs['formData'].resetFields();
					}
					this.dialogVisible = true;
					this.editType = type;
				},
				//编辑通过ID查询数据,渲染数据
				editorRow(tagId, type) {
					this.editType = type;
					this.$http({
						url: this.apiJSON.lable_ByIdInfo,
						method: 'get',
						params: {
							tagId: tagId
						}
					}).then((data)=>{
						this.formData = Object.assign(this.formData, data);
						this.dialogVisible = true;
					});
				},
			//添加标签
			save(type) {
				//编辑
				if (this.editType == 'edit') {
					this.$http({
						url: this.apiJSON.lable_editor,
						method: 'post',
						data: this.formData
					}).then((data)=>{
						if (data) {
							this.$message({
								type: 'success',
								message: '修改成功!'
							});
							this.queryTotal();
							this.dialogVisible = false;
						}
					});
				  //添加
				} else if (this.editType == 'add') {
					this.$http({
						url: this.apiJSON.lable_save,
						method: 'post',
						data: this.formData
					}).then((data)=>{
						if (data) {
							this.$message({
								type: 'success',
								message: '添加成功!'
							});
							this.queryTotal();
							this.dialogVisible = false;
						}
					});

				}
			}
	}
</script>