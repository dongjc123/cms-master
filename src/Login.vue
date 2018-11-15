<template>
	<div class="login">
		<div class="bg">
			<h3>热点新闻资讯</h3>
		</div>
		<div class="login-dialog">
			<div class="title">
				<h2>登录页</h2>
			</div>
			<!-- {{login}} -->
			<div class="form">
				<el-form ref='userForm' :rules="rules" :model="form" label-position="left" size='small'>
				  <el-form-item prop='username' label="用户名" label-width="80px">
				    <el-input v-model="form.username"></el-input>
				  </el-form-item>
				  <el-form-item prop='password' label="密码" label-width="80px">
				    <el-input v-model="form.password" type='password'></el-input>
				  </el-form-item>
				</el-form>
			</div>
			<div class='btn'>
				<el-button type="primary" size='mini' @click='login'>登录</el-button>
			</div>
		</div>
		
	</div>
</template>
<script>
	import axios from '@/http/axios'
	export default {
		data(){
			return {
			    form: {
				    name: '',
				    password: ''
			    },
			    rules:{
			    	username:[{
			    		required:true,
			    		message:'请输入用户名',
			    		tirgger:'blur'
			    	}],
			    	password:[{
			    		required:true,
			    		message:'请输入密码',
			    		tirgger:'blur'
			    	}]
			    }
			}
		},
		methods:{
			login(){
				//登录前验证表单
				this.$refs.userForm.validate((valid)=>{
					if(valid){
						axios.post('http://47.107.41.99:8888/login',this.form)
						.then(({data:result})=>{
							if(result.status == 200 && result.message == '登录成功'){
								//1. 跳转到后台管理页面
								window.vm.currentComponent = 'App';
								//2. 将登录成功的用户信息保存到浏览器中
								localStorage.setItem('user',JSON.stringify(result.data));
							}
						})
						.catch((error)=>{
							console.log('error',error);
						});
						// this.$root.currentComponent = 'App';
					} 
				});
			}
		}
	}
</script>
<style scoped>
	/* input:-webkit-autofill {
		-webkit-box-shadow:0 0 0 1000px white inset !important;
	}
	html,body {
		background-color: #f0f0f0;
		margin: 0;
		padding: 0;
	} */
	.login {
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		background-image:url('./assets/login.jpg');
        background-repeat: no-repeat;
        background-size:100%;
	}
	
	.login .bg h3 {
		text-align: left;
		margin-top: 50px;
		margin-left: 100px;
		font-size: 50px;
		color: #fff;
	}
	.login .login-dialog {
		width: 350px;
		margin: 0 auto;
		padding: 2em;
		text-align: center;
		margin-top: 7em;
		border: 1px solid #ccc;
		border-radius: 10px;
		background-image:url('./assets/tx.jpg');
        background-repeat: no-repeat;
        background-size:100%;
		
	}
	.login .login-dialog:hover{
		box-shadow: 0 0 8px 8px #bbb;
	}
	.login .title {
		margin-bottom: 2em;
	}
	.login .btn {
		text-align: center;
		margin-top: 2em;
	}
</style>
