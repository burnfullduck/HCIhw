<template>
	<el-upload
			class="upload-demo"
			ref="upload"
			action="http://124.221.102.208:8081/senti-strength/file"
			:data="{ 'para':optionList }"
			:limit="1"
			:on-exceed="handleExceed"
			:file-list="fileList"
			:auto-upload="false"
			:before-upload="beforeAvatarUpload"
			:http-request="httpRequest"
			:on-success="handleSuccess">
		<template #trigger>
			<el-button size="small" type="primary" @click="this.$refs.upload.clearFiles();"><el-icon><DocumentAdd /></el-icon>&nbsp;选取单个文件</el-button>

		</template>


	</el-upload>
  <el-button size="small" type="success" @click="submitUpload"><el-icon><ZoomIn /></el-icon>&nbsp;开始分析</el-button>



	<!--tip-->
	<p></p>
	<i>注：只能上传txt文件</i>
	<p></p>
  <div  v-loading="loading">
    <div v-if="fileinfo.virtualPath.length !== 0"> 分析结果如下，可点击下载</div>
	<p></p>
    <div  v-if="fileinfo.virtualPath.length !== 0" @click="downFile"><i class="el-icon-document"> <el-icon><Document /></el-icon></i> <a href="javascript:void(0);">{{getFileName(fileinfo.virtualPath)}}分析结果</a>  <el-icon><Download/></el-icon></div>

  </div>
<!--	<div @click="downFile"><i class="el-icon-document"></i>{{fileinfo.virtualPath}}</div>-->
	<br/>
</template>
<script>



import {ElMessageBox} from "element-plus";
import axios from "axios";

export default {
	data() {
		return {
      loading : false,
			fileList: [],
			fileinfo:{
				virtualPath:"",
			},
			optionList: ""
		};
	},
	methods: {
    getFileName(raw_name){
      let names = raw_name.split('-');
      return names[1];
    },
		submitUpload() {
			this.$refs.upload.submit();
		},
		handleExceed(){
			return ElMessageBox.alert(
					`Limit 1 file`
			).then(
					() => true
			)
		},
		beforeAvatarUpload(file){
			const isTxt = file.type === 'text/plain';
			if(!isTxt){
				this.$message.error('上传文件格式只能是txt!')
			}
			return isTxt;
		},
		handleSuccess(result){
			this.fileinfo.virtualPath=result;
		},
		downFile(){
			var url = "http://124.221.102.208:8081/senti-strength/downloadFile?filePath="+this.fileinfo.virtualPath;
			window.open(url);
		},
		httpRequest(param) {
      this.loading = true;
			let fd = new FormData();
			fd.append("file", param.file);
			this.optionList = "input\t"+this.$parent.parseCmd();
			fd.append("para", this.optionList);
			axios.post('http://124.221.102.208:8081/senti-strength/file', fd , {
				headers: {'Content-Type': 'multipart/form-data'},//定义内容格式,很重要
				timeout: 20000,
			}).then(response => {
				this.fileinfo.virtualPath=response.data;
        this.$message.success('分析完成!');
			}).catch(err =>{console.log(err);
				})
      this.loading = false;
		}
	}
}
</script>


