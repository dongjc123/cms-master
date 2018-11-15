<template>
  <div class="category">
  	<!-- 按钮区 -->
  	<div class="btns">
  		<el-button type="primary" size='small' @click='toAddCategory'>添加</el-button>
  		<el-button type="danger" size='small' @click='batchDeletecategory'>批量删除</el-button>
  	</div>
  	<!-- 按钮区结束 -->
  	<!-- 表格区 -->
  	<div class="table" v-loading='loading'>
  		 <el-table
  		  :border='true'
  		  size='small'
	      :data="categories"
	      style="width: 100%"
	       @selection-change="handleSelectionChange">
	      <el-table-column
		    type="selection"
		    width="55">
		  </el-table-column>
	      <el-table-column
	        prop="name"
	        label="栏目名称"
	        width="180">
	      </el-table-column>
	      <el-table-column
	        prop="parent.name"
	        label="父栏目">
	      </el-table-column>
	      <el-table-column
	        prop="comment"
	        label="描述">
	      </el-table-column>
	      <el-table-column label="操作" width='100'>
	      	<template slot-scope='{row}'>
	      		<i class="fa fa-trash" @click='deleteCategory(row.id)'></i>
	      		<i class="fa fa-pencil" @click='toUpdateCategory(row)'></i>
	      	</template>
	      </el-table-column>
	    </el-table>
  	</div>
  	<!-- 表格区结束 -->
  	<!-- 模态框 -->
  	<el-dialog :title="categoryDialog.title" :visible.sync="categoryDialog.visible">
  		<!-- {{categoryDialog.form}} -->
	  <el-form :model="categoryDialog.form" :rules="rules" ref="categoryDialog.form">
	    <el-form-item label="栏目名称" label-width="120px" prop="name">
	      <el-input v-model="categoryDialog.form.name" autocomplete="off"></el-input>
	    </el-form-item>
	    <el-form-item label="父栏目" label-width="120px">
	      <el-select v-model="categoryDialog.form.parentId" placeholder="一级栏目">
	        <el-option :key='c.id' v-for='c in categories' :label="c.name" :value="c.id"></el-option>
	      </el-select>
	    </el-form-item>
	    <el-form-item label="描述信息" label-width="120px" prop="comment">
	      <el-input
			  type="textarea"
			  :rows="2"
			  placeholder="请输入内容"
			  v-model="categoryDialog.form.comment">
			</el-input>
	    </el-form-item>
	  </el-form>
	  <div slot="footer" class="dialog-footer">
	    <el-button size='small' @click='closeCategoryDialog'>取 消</el-button>
	    <el-button size='small' type="primary" @click="submitForm('categoryDialog.form')">确 定</el-button>
	  </div>
	</el-dialog>
  	<!-- 模态框结束 -->
  </div>
</template>

<script>
	import axios from '@/http/axios'
	export default {
	  data(){
	    return {
	    	categories:[],
	    	loading:false,
	    	ids:[],
	    	categoryDialog:{
	    		title:'',
	    		visible:false,
	    		form:{
	    			name: '',
		        	comment: ''
	    		},
	    	},
	    	rules: {
	          name: [
	            { required: true, message: '请输入栏目名称', trigger: 'blur' },
	            { min: 2, max: 4, message: '长度在 2 到 4 个字符', trigger: 'blur' }
	          ],
	          comment: [
	            { required: true, message: '请填写描述信息', trigger: 'blur' }
	          ]
	        }
	    }
	  },
	  created(row){
	  	//加载所有栏目信息
	  	this.findAllCategories();
	  },
	  methods:{
	  	//表单验证
	  	submitForm(formName) {
	        this.$refs[formName].validate((valid) => {
	          if (valid) {
	          	//通过表单验证之后执行保存或修改操作
	            this.saveOrUpdateCategory();
	          } else {
	            console.log('error submit!!');
	            return false;
	          }
	        });
	      },
	  	//修改操作
	  	toUpdateCategory(row){
	  		//1. 显示模态框
	  		this.categoryDialog.title = '修改栏目';
	  		this.categoryDialog.visible = true;
	  		//2. 处理表单数据
	  		let cate = _.cloneDeep(row);
	  		this.categoryDialog.form = cate;
	  		cate.parentId = cate.parent.id;
	  		delete cate.parent;
	  	},
	  	//批量删除
	  	batchDeletecategory(){
			let ids  = this.ids.map((item)=>{
				return item.id;
			});
			this.$confirm('此操作将批量删除这些栏目, 是否继续?', '提示', {
	          confirmButtonText: '确定',
	          cancelButtonText: '取消',
	          type: 'warning'
	        })
	        .then(()=>{
	        	let url = '/manager/category/batchDeleteCategory';
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
				    this.findAllCategories();
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
			this.ids = val;
		},
	  	//删除操作
	  	deleteCategory(id){
	  		this.$confirm('此操作将永久删除该栏目以及该栏目的子栏目，以及该栏目下的文章, 是否继续?', '提示', {
	          confirmButtonText: '确定',
	          cancelButtonText: '取消',
	          type: 'warning'
	        })
	        .then(()=>{
	        	let url = '/manager/category/deleteCategoryById?id='+id;
	        	axios.get(url)
	        	.then(({data:result})=>{
	        		this.$notify({
				        title: '删除成功',
				        message: result.message,
				        type: 'success'
				    });
				    this.findAllCategories();
	        	})
	        	.catch(()=>{
	        		this.$notify.error({
			          title: '错误',
			          message: '删除异常'
			        });
	        	});
	        })
	        .catch(()=>{});
	  	},
	  	//保存或更新栏目
	  	saveOrUpdateCategory(){
	  		let url = '/manager/category/saveOrUpdateCategory'
	  		axios.post(url,this.categoryDialog.form)
	  		.then(({data:result})=>{
	  			if(result.status == 200){
	  				//1.关闭模态框
	  				this.closeCategoryDialog();
	  				//2.提示成功
	  				this.$notify({
				        title: '成功',
				        message: result.message,
				        type: 'success'
				    });
				    //3.刷新
				    this.findAllCategories();
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
	  	//关闭模态框
	  	closeCategoryDialog(){
	  		this.categoryDialog.visible = false;
	  	},
	  	toAddCategory(){
	  		this.categoryDialog.form = {};
	  		this.categoryDialog.title = '添加栏目';
	  		this.categoryDialog.visible = true;
	  	},
	  	//查询所有栏目
	  	findAllCategories(){
	  		this.loading = true;
	  		let url = '/manager/category/findAllCategory';
	  		axios.get(url).then(({data:result})=>{
	  			//将查询到的数据绑定到模型中，进而渲染到页面
	  			this.categories = result.data;
	  		}).catch((error)=>{
	  			console.log('error',error);
	  		}).finally(()=>{
	  			this.loading = false;
	  		});
	  	}
	  }
	}
</script>

<style scoped>
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
