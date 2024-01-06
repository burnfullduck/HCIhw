<script setup>
import { ref, defineProps } from 'vue'
import {ElMessage} from 'element-plus'
import axios from 'axios'

// reactive state
const text = ref('')
const tableData = ref([
])
const props = defineProps({
	get_cmd :  Function ,
});
const responseText = ref('');
function show_result(result) {
  let results = result.split(' ');
    tableData.value=[{
      positive: results[0],
      negative: results[1],
      result: (results.length ===2)?'无':results[2]
    }]
}

function sendText() {
	let fd = new FormData();
	let myOptionList = props.get_cmd();
	fd.append("text", text.value.toString());
	fd.append("para", "text\t" + myOptionList);
	console.log(myOptionList);
	axios.post('http://124.221.102.208/controller/text', fd, {
		headers: {'Content-Type': 'multipart/form-data'},//定义内容格式,很重要
		timeout: 20000,
	}).then((res) => {
		responseText.value = res.data;
    console.log('2222222222222'+ responseText.value);
    show_result(responseText.value);
    ElMessage.success('分析完成!');
	}).catch(err => {
		console.log(err);
	})
}

function translate() {
  if (text.value === ''){
    ElMessage.error('输入不能为空!');
    return ;
  }
	const crypto = require('crypto');
	console.log(text);
	let api = 'https://fanyi-api.baidu.com/api/trans/vip/translate';
	let raw = '';
	let q = text.value;
	raw = raw + '?q=' + q;
	raw = raw + '&from=auto';
	raw = raw + '&to=en';
	let app_id = '20230405001629785';
	raw = raw + '&appid=' + app_id;
	let salt = '';
	for (let i = 0; i < 10; i++) {
		salt = salt + Math.round(Math.random() * 9);
	}
	console.log(salt);
	raw = raw + '&salt=' + salt;
	console.log(salt);
	api = api + raw;
	let secret_key = '12gScEzaWA3k5rMfHemT';
	let before_md5 = app_id + q + salt + secret_key;
	const hash = crypto.createHash('md5');
	hash.update(before_md5);
	let sign = hash.digest('hex');
	api = api + '&sign=' + sign;
	console.log(api);
	axios.get('http://124.221.102.208/controller/translate',{
    params:{
      url : api
    }
  }).then((response) => {
		let result = response.data.trans_result[0].dst;
		text.value = result;
		sendText();
		console.log(result);
	})
}
</script>

<template>
	<div style="z-index: 100;">
        <div>
            <input v-model="text" type="text" placeholder="输入需要分析情绪的中文文本" style="width: 100%;"/>
        </div>
        <p></p>
        <div>
          <el-button size="small" type="success" @click="translate"><el-icon><ZoomIn /></el-icon>&nbsp;翻译并分析</el-button>

        </div>
        <p></p>

      <div v-show="responseText!==''" >分析结果为:
        <el-table :data="tableData" style="width: 100%">
          <el-table-column prop="positive" label="积极情绪"  />
          <el-table-column prop="negative" label="消极情绪"  />
          <el-table-column prop="result" label="总体结果" />
        </el-table>
		<p></p>

      </div>
    </div>
  
</template>

<style scoped></style>