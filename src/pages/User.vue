<template>
	<div class="user">
		<!-- 按钮区 -->
		<div class="btns">
			<el-button type="primary" size='small' @click='toAddUser'>添加</el-button>
		</div>
		<!-- 按钮区结束 -->
		<!-- 数据区 -->
		<div class="user-tbl" v-loading='loading'>
  			<el-table
  			:border='true'
  			size='small'
	    	:data='users'
	    	style="width: 100%">
	    		<el-table-column
	        	prop="username"
	        	label="用户名"
	        	width="180">
	    		</el-table-column>
	    		<el-table-column
	        	prop="password"
	        	label="密码"
	        	width="180">
	    		</el-table-column>
	    		<el-table-column
	        	prop="nickname"
	        	label="真实姓名"
	        	width="180">
	    		</el-table-column>
	    		<el-table-column
	        	prop="email"
	        	label="email">
	    		</el-table-column>
	    		<el-table-column
	        	prop="regTime"
	        	label="注册时间">
	    		</el-table-column>
	    		<el-table-column label="操作" width='100'>
	      			<template slot-scope='{row}'>
	      				<i class="fa fa-trash" @click='deleteUser(row.id)'></i>
	      				<i class="fa fa-pencil" @click='toUpdateUser(row)'></i>
	      			</template>
	      		</el-table-column>
	    	</el-table>
  		</div>
		<!-- 数据区结束 -->
		<!-- 模态框区 -->
		<el-dialog :title="userDialog.title" :visible.sync="userDialog.visible">
			<!-- {{userDialog.form}} -->
		  <el-form  :model="userDialog.form">
		    <el-form-item label="用户名称" label-width="100px">
		      <el-input v-model="userDialog.form.username" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="用户密码" label-width="100px" prop='title'>
		      <el-input type='password' v-model="userDialog.form.password" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="真实姓名" label-width="100px" prop='title'>
		      <el-input v-model="userDialog.form.nickname" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="用户邮箱" label-width="100px" prop='title'>
		      <el-input type='email' v-model="userDialog.form.email" autocomplete="off"></el-input>
		    </el-form-item>
		  </el-form>
		  <div slot="footer" class="dialog-footer">
		    <el-button size='small' @click="closeUserDialog">取 消</el-button>
		    <el-button size='small' type="primary" @click='saveOrUpdateUser'>确 定</el-button>
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
				users:[],
				loading:false,
				userDialog:{
					title:'',
					visible:false,
					form:{}
				}
			}
		},
		created(){
			this.findAllUsers();
		},
		methods:{
			//修改
			toUpdateUser(row){
				this.userDialog.title = '修改用户';
				this.userDialog.visible = true;
				let u = _.cloneDeep(row);
				this.userDialog.form = u;
				for(let key in u){
					let val = u[key]
					if(!val){
						delete u[key];
					}
				}
			},
			//删除
			deleteUser(id){
				this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        })
		        .then(()=>{
		        	let url = '/manager/user/deleteUserById?id='+id;
		        	axios.get(url)
		        	.then(({data:result})=>{
		        		this.$notify({
					        title: '删除成功',
					        message: result.message,
					        type: 'success'
					    });
					    this.findAllUsers();
		        	})
		        	.catch(()=>{
		        		alert(1);
		        		this.$notify.error({
				          title: '错误',
				          message: '删除异常'
				        });
		        	});
		        })
		        .catch(()=>{});
			},
			//保存或更新用户
			saveOrUpdateUser(){
				let url = '/manager/user/saveOrUpdateUser'
				axios.post(url,this.userDialog.form)
				.then(({data:result})=>{
					if(result.status == 200){
						this.closeUserDialog();
						this.$notify({
					        title: '成功',
					        message: result.message,
					        type: 'success'
					    });
					    //3.刷新
					    this.findAllUsers();
		  			} else {
		  				this.$notify.error({
				          title: '错误1',
				          message: result.message
				        });
		  			}
		  		})
		  		.catch((error)=>{
		  			this.$notify.error({
				          title: '错误2',
				          message: '网络异常'
				        });
		  		});
			},
			//关闭模态框
			closeUserDialog(){
				this.userDialog.visible = false;
			},
			//添加用户
			toAddUser(){
				this.userDialog.form = {};
				this.userDialog.title = '添加用户';
				this.userDialog.visible = true;
			},
			//查找所有用户
			findAllUsers(){
				this.loading = true;
				let url = '/manager/user/findAllUser';
				axios.get(url).then(({data:result})=>{
					// 将查询到的数据绑定到模型中
					this.users = result.data;
				}).catch((error)=>{
		  			console.log('error',error);
		  		}).finally(()=>{
		  			this.loading = false;
		  		});
			},
		}
	}
</script>

<style scoped>
	.btns {
		margin-bottom: .5em;
	}
	i.fa {
		margin: 0 .8em;
		cursor: pointer;;
	}
	i.fa.fa-trash {
		color: #F56C6C;
	}
	i.fa.fa-pencil {
		color: #409eff;
	}
	.user-tbl:hover{
		
	}
</style>



