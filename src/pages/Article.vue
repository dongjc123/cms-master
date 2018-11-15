<template>
	<div class="article">
		<!-- 按钮区 -->
		<div class="btns">
			<el-select clearable style='width: 120px;' size='small' v-model="params.categoryId" placeholder="所有栏目">
			    <el-option v-for='c in categories' :label='c.name' :value='c.id' :key='c.id'></el-option>
			</el-select>
			<el-input
			  style="width: 140px;"
			  size='small'
			  placeholder="请输入关键字"
			  v-model="params.keywords"
			  clearable>
			</el-input>
	  		<el-button type="primary" size='small' @click='toAddArticle'>添加</el-button>
	  		<el-button type="danger" size='small' @click='batchDeleteArticles'>批量删除</el-button>
	  	</div>
		<!-- 按钮区结束 -->
		<!-- 数据区 -->
		<!-- {{page}} -->
		<div class="article-tbl" v-loading='loading'>
	  		<el-table
	  		:border='true'
	  		size='small'
		    :data="articles"
		    style="width: 100%"
		    @selection-change="handleSelectionChange">
		      <el-table-column
			    type="selection"
			    width="55">
			  </el-table-column>
		      <el-table-column
		        prop="title"
		        label="文章标题"
		        width="180">
		      </el-table-column>
		      <el-table-column
		        prop="category.name"
		        label="所属栏目"
		        width="120">
		      </el-table-column>
		      <!-- <el-table-column
		        prop="author"
		        label="作者">
		      </el-table-column> -->
		      <el-table-column
		        prop="publishtime"
		        label="发布时间"
		        width="200">
		      </el-table-column>
		      <el-table-column
		        prop="readtimes"
		        label="阅读次数">
		      </el-table-column>
		      <el-table-column label="操作" width='100'>
		      	<template slot-scope='{row}'>
		      		<i class="fa fa-trash" @click='deleteArticle(row.id)'></i>
		      		<i class="fa fa-pencil" @click='toUpdateArticle(row)'></i>
		      	</template>
		      </el-table-column>
		    </el-table>
		</div>
		<!-- 数据区结束 -->
		<!-- 分页 -->
		<div class="page">
			<el-pagination
			  layout="prev, pager, next"
			  @current-change='handleCurrentChange'
			  :page-size='params.pageSize'
			  :total="total">
			</el-pagination>
		</div>
		<!-- 分页结束 -->
		<!-- 模态框区 -->
		<el-dialog fullscreen :title="articleDialog.title" :visible.sync="articleDialog.visible">
			<!-- {{articleDialog.form}} -->
		  <el-form :model="articleDialog.form" 
		  :rules="rules" ref="articleDialog.form">
		    <el-form-item label="资讯标题" label-width="120px" prop='title'>
		      <el-input v-model="articleDialog.form.title" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="所属栏目" label-width="120px" prop='categoryId'>
		      <el-select v-model="articleDialog.form.categoryId" placeholder="请选择栏目名称">
		        <el-option :key='c.id' v-for='c in categories' :label="c.name" :value="c.id"></el-option>
		      </el-select>
		    </el-form-item>
			<el-form-item label="列表样式" label-width="120px" prop='liststyle'>
		      <div class="list-style">
		      	<div class="style-one" 
		      	:class='{current:articleDialog.form.liststyle == "style-one"}' 
		      	@click='articleDialog.form.liststyle="style-one"'>
		      		<img src="@/assets/style-one.jpg" alt="图片加载不成功">
		      	</div>
		      	<div class="style-two" 
		      	:class='{current:articleDialog.form.liststyle == "style-two"}'
		      	@click='articleDialog.form.liststyle="style-two"'>
		      		<img src="@/assets/style-two.jpg" alt="图片加载不成功">
		      	</div>
		      </div>
		    </el-form-item>
		    <el-form-item label="缩略图" label-width="120px">
			<el-upload
			  action="http://39.107.91.212:8099/manager/file/upload"
			  :file-list='articleDialog.fileList'
			  :on-remove='handleUploadRemove'
			  :on-success='handleUploadSuccess'
			  list-type="picture">
			  <el-button size="small" type="primary">点击上传</el-button>
			  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
			</el-upload>
		    </el-form-item>
				<el-form-item label="资讯正文" label-width="120px" prop='content'>
		      		<!-- <el-input
					  type="textarea"
					  :rows="5"
					  placeholder="请输入内容"
					  v-model="articleDialog.form.content">
					</el-input> -->
					<mavon-editor ref=md v-model="articleDialog.form.content"/>
			    </el-form-item>
			</el-form>
		  <div slot="footer" class="dialog-footer">
		    <el-button size='small' @click='closeArticleDialog'>取 消</el-button>
		    <el-button size='small' type="primary" @click="submitForm('articleDialog.form')">确 定</el-button>
		  </div>
		</el-dialog>
		<!-- 模态框区结束 -->
	</div>
</template>
<script>
	import axios from '@/http/axios'
	export default {
		data(){
			return {
				loading:false,
				categories:[],
				articles:[],
				total:0,
				multipleSelection:[],
				params:{
					page:0,
					pageSize:5,
				},
				articleDialog:{
					title:'',
					visible:false,
					form:{
						liststyle:'style-one',
						fileIds:[],
					},
					fileList:[]
				},
				rules: {
		          title: [
		            { required: true, message: '请输入文章名称', trigger: 'blur' },
		            { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
		          ],
		          categoryId: [
		            { required: true, message: '请选择栏目名称', trigger: 'change' }
		          ],
		          liststyle: [
		            { required: true, message: '请选择列表样式', trigger: 'change' }
		          ],
		          content: [
		            { required: true, message: '请填写文章正文', trigger: 'blur' }
		          ]
		        }
			}
		},
		watch:{
			//只要params中的任意参数改变，就要刷新页面
			params:{
				handler:function(){
					this.findAllArticles();
				},
				deep:true
			}
		},
		created(){
			this.findAllCategories();
			this.findAllArticles();
		},
		methods:{
			//表单验证
			submitForm(formName) {
		        this.$refs[formName].validate((valid) => {
		          if (valid) {
		          	//通过表单验证之后执行保存或修改操作
		            this.saveOrUpdateArticle();
		          } else {
		            console.log('error submit!!');
		            return false;
		          }
		        });
		      },
			//删除附件
			handleUploadRemove(file){
				//1. 调用接口去删除图片
				axios.get('/manager/file/delete',{
					params:{
						id:file.name
					}
				})
				.then(()=>{
					this.$notify({title:'成功',message:'操作成功'});
					//2. 从fileIds中移除掉
					_.remove(this.articleDialog.form.fileIds,(id)=>{
						return id == file.name;
					});
					this.articleDialog.form.fileIds.push(1);
					this.articleDialog.form.fileIds.pop();
				})
				.catch(()=>{
					this.$notify.error({title:'错误',message:'网络异常'});
				});
			},
			//修改操作
			toUpdateArticle(row){
				//1. 显示模态框
				this.articleDialog.title = '修改资讯';
				this.articleDialog.visible = true;
				//2. 克隆当前行数据（避免错误修改）
				let article = _.cloneDeep(row);
				//3. 处理附件默认显示
				this.articleDialog.fileList = article.articleFileVMs.map((item)=>{
					return {
						name:item.cmsFile.id,
						url:'http://39.108.81.60:8888/group1/'+item.cmsFile.id
					}
				});
				//4. 处理表单数据
				//4.1 栏目依赖 category -> categoryId
				article.categoryId = article.category.id;
				delete article.category;
				//4.2 文件依赖 articleFileVMs -> fileIds
				article.fileIds = article.articleFileVMs.map(item=>item.cmsFile.id);
				delete article.articleFileVMs;
				//4.3 取消默认值
				for(let key in article){
					let val = article[key]
					if(!val){
						delete article[key];
					}
				}
				//5. 强处理后的结果与表单双向绑定
				this.articleDialog.form = article;
			},
			//批量删除
			batchDeleteArticles(){
				let ids  = this.multipleSelection.map((item)=>{
					return item.id;
				});
				let obj = {
					ids:ids
				};
				this.$confirm('此操作将批量删除这些文章, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        })
		        .then(()=>{
		        	let url = '/manager/article/batchDeleteArticle';
					let obj = {
						ids:ids
					};
		        	axios.post(url,obj)
		        	.then(({data:result})=>{
		        		this.$notify({
					        title: '删除成功',
					        message: result.message,
					        type: 'success'
					    });
					    this.findAllArticles();
		        	})
		        	.catch(()=>{
		        		this.$notify.error({
				          title: '删除错误',
				          message: '删除异常'
				        });
		        	});
		        })
		        .catch(()=>{});
			},
			handleSelectionChange(val){
				this.multipleSelection = val;
			},
			//关闭模态框
			closeArticleDialog(){
				this.articleDialog.visible = false;
			},
			//保存或更新资讯
			saveOrUpdateArticle(){
				this.articleDialog.form.source = this.$refs.md.d_render;
				let url = '/manager/article/saveOrUpdateArticle'
		  		axios.post(url,this.articleDialog.form)
		  		.then(({data:result})=>{
		  			if(result.status == 200){
		  				//1.关闭模态框
		  				this.closeArticleDialog();
		  				//2.提示成功
		  				this.$notify({
					        title: '成功',
					        message: result.message,
					        type: 'success'
					    });
					    //3.刷新
					    this.findAllArticles();
		  				} else {
		  				this.$notify.error({
				          title: '错误',
				          message: result.message
				        });
		  			}
		  		})
		  		.catch((error)=>{
		  			this.$notify.error({
				          title: '错误',
				          message: '网络异常'
				    });
		  		});
			},
			//获取成功上传文件的id
			handleUploadSuccess(response,file,fileList){
				this.articleDialog.form.fileIds.push(response.data.id);
			},

			//添加按钮
			toAddArticle(){
				this.articleDialog.form={
					liststyle:'style-one',
					fileIds:[],
					fileList:'',
				};
				this.articleDialog.fileList = [];
				this.articleDialog.title = '发布资讯';
				this.articleDialog.visible = true;
			},
			//查询所有栏目
		  	findAllCategories(){
		  		let url = '/manager/category/findAllCategory';
		  		axios.get(url).then(({data:result})=>{
		  			//将查询到的数据绑定到模型中，进而渲染到页面
		  			this.categories = result.data;
		  		}).catch((error)=>{
		  			this.$notify.error({
		  				title:'错误',
		  				massage:'网络异常'
		  			})
		  		});
		  	},
		  	//分页的页码改变
			handleCurrentChange(page){
				this.params.page = page-1;
			},
			//查询所有栏目
		  	findAllArticles(){
		  		this.loading = true;
		  		let url = '/manager/article/findArticle';
		  		axios.get(url,{
		  			params:this.params
		  		})
		  		.then(({data:result})=>{
		  			//将查询到的数据绑定到模型中，进而渲染到页面
		  			console.log(result);
		  			this.total = result.data.total;
		  			this.articles = result.data.list;
		  		}).catch((error)=>{
		  			this.$notify.error({
		  				title:'错误',
		  				massage:'网络异常'
		  			})
		  		}).finally(()=>{
		  			this.loading = false;
		  		});
		  	},
		  	//删除
			deleteArticle(id){
				this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        })
		        .then(()=>{
		        	let url = '/manager/article/deleteArticleById?id='+id;
		        	axios.get(url)
		        	.then(({data:result})=>{
		        		this.$notify({
					        title: '成功',
					        message: result.message,
					        type: 'success'
					    });
					    this.findAllArticles();
		        	})
		        	.catch(()=>{
		        		this.$notify.error({
				          title: '错误',
				          message: '删除异常'
				        });
		        	});
		        })
		        .catch(()=>{});
			}
		}
	}
</script>

<style scoped>
	.list-style {

	}
	.list-style>div {
		width: 200px;
		border: 1px solid #fff;
		border-radius: 8px;
		padding: 2px;
	}
	.list-style>div:hover{
		cursor: pointer;
	}
	.list-style>div.current {
		border-color: #419fff;
	}
	.list-style img {
		width: 100%;
	}
	.list-style .style-one {
		float: left;
	}
	.list-style .style-two {
		margin-left: 220px;
	}
	.btns {
		margin-bottom: .5em;
	}
	i.fa {
		margin: 0 .8em;
		cursor: pointer;
	}
	i.fa.fa-trash {
		color: #F56C6C;
	} 
	i.fa.fa-pencil {
		color: #409eff;
	} 
</style>