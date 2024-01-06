<template>
	<el-upload
			class="upload-demo"
			action="http://124.221.102.208:8081/senti-strength/fileFolder"
			:data="{
        'para':optionList
        }"
			accept=""
			:auto-upload="false"
			:multiple="true"
			:file-list="fileList"
			:on-change="fileChangeHandle"
			:http-request="httpRequest"
			ref="uploadFile">
<!--		<el-button v-if="this.$refs.uploadFile.length === 0" style="margin-left: 10px;" size="small" type="primary" ref="folder" @click="this.$refs.uploadFile.clearFiles()">-->
<!--			选取文件夹-->
<!--			<i class="el-icon-folder-opened"></i>-->
<!--		</el-button>-->

    <el-button size="small" type="primary" ref="folder">
      <el-icon><FolderAdd /></el-icon>
      &nbsp;选取文件夹
    </el-button>

	</el-upload>
  <div>
  <el-popconfirm title="确认清空已上传的文件吗?" @confirm="this.$refs.uploadFile.clearFiles()"
      width="220"
      confirm-button-text="确认"
      cancel-button-text="取消"
      :icon="InfoFilled"
      icon-color="#626AEF"
  >
    <template #reference>
      <el-button type="danger" style="margin-bottom: 8px;" size="small" >
        <el-icon><Delete /></el-icon>&nbsp; 清空已上传文件</el-button>
    </template>
  </el-popconfirm>
  </div>
  <div>
	<el-button size="small" type="success" @click="submitUpload"><el-icon><ZoomIn /></el-icon>&nbsp;开始分析</el-button>
  </div>
    <p></p>
	<div v-show="false" class="form-check">
		<input class="form-check-input" type="checkbox" value="annotateCol" id="annotateCol" v-model="setAnnotateCol" @change="showAnnotateCol = !showAnnotateCol">
		<label class="form-check-label" for="annotateCol">
			设置需要分析的文本在第几列(默认为1)
		</label>
		<input type="text" class="form-check" v-model="annotateCol" placeholder="正整数" v-if=showAnnotateCol>
	</div>
  <div  v-loading="loading">
    <div v-if="virtualPathList.length !== 0"> 分析结果如下，可点击下载</div>
    <div  v-for="(file,index) in virtualPathList" :key="index" @click="downFile(file)"><i class="el-icon-document"> <el-icon><Document /></el-icon></i> <a href="javascript:void(0);">{{getFileName(file)}}分析结果</a>  <el-icon><Download/></el-icon></div>

  </div>
 <!--	<br/>-->
<!--	<br/>-->
<!--	<br/>-->
</template>
<script>



import axios from "axios";

export default {
	data() {
		return {
      loading : false,
			fileList:[],
			uploadList:[],
			virtualPathList:[],
			optionList: "",
			showAnnotateCol: false,
			setAnnotateCol:[],
			annotateCol:'1',
      uploadState : 'false',
		}
	},
	mounted:function () {
		const upLoadEle = document.querySelectorAll('.el-upload__input');
		upLoadEle[1].webkitdirectory = true;
	},

	methods: {
    getFileName(raw_name){
        let names = raw_name.split('-');
        return names[1];
    },
		submitUpload() {
			this.$refs.uploadFile.submit();
		},
		fileChangeHandle(file, fileList) {
			this.fileList = fileList;
		},
		handleSuccess(result){
			console.log(result);
			this.virtualPathList=result.data;
			console.log(this.virtualPathList);
		},
		downFile(file){
			var url = "http://124.221.102.208:8081/senti-strength/downloadFile?filePath="+encodeURI(file);
			window.open(url);
		},
		httpRequest(param) {
      this.loading = true;
			let fd = new FormData();
			this.uploadList.push(param.file);
			if(this.uploadList.length<this.fileList.length || this.uploadList.length === 0){
				return;
			}
			var i;
			for(i in this.uploadList){
				fd.append("files", this.uploadList[i]);
			}
			this.optionList="fileSubstring\ttxt\t";
			this.optionList += "annotateCol"+'\t' +this.annotateCol;
			this.optionList += "\tinputFolder\t"+this.$parent.parseCmd();
			fd.append("para", this.optionList);
			axios.post('http://124.221.102.208:8081/senti-strength/fileFolder', fd , {
				headers: {'Content-Type': 'multipart/form-data'},//定义内容格式,很重要
				timeout: 20000,
			}).then(response => {
				console.log(response);
				this.virtualPathList = response.data;
				this.optionList = [];
				this.uploadList = [];
        this.$message.success('分析完成!');
			}).catch(err =>{console.log(err);
				this.uploadList = [];
        this.$message.error('分析失败!');});
        this.loading = false;
		}


	}

}
</script>
