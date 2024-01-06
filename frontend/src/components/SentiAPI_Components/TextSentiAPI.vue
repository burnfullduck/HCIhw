<template>
    <div style="z-index: 100;">
        <div>
            <input v-model="inputText" type="text" placeholder="输入需要分析情绪的英文文本" style="width: 100%;"/>
        </div>
        <p></p>
        <div>
          <el-button size="small" type="success" @click="sendText"><el-icon><ZoomIn /></el-icon>&nbsp;开始分析</el-button>

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

<script>
import axios from "axios";

export default {
    name: 'SentiAPI',
    data() {
        return {
            tableData : [],
            inputText: '',
            responseText: '',
            myOptionList:""
        };
    },
    methods: {
       show_result(result) {
         let results = result.split(' ');
         this.tableData = [{
           positive: results[0],
           negative: results[1],
           result: (results.length === 2) ? '无' : results[2]
         }];
},
        sendText() {
            if (this.inputText === ''){
              this.$message.error('输入不能为空!');
              return ;
            }
            let fd = new FormData();
            this.myOptionList = this.$parent.parseCmd();
            console.log(this.myOptionList);
            fd.append("text", this.inputText.toString());
            fd.append("para", "text\t"+this.myOptionList);
            axios.post('http://124.221.102.208/controller/text', fd , {
                headers: {'Content-Type': 'multipart/form-data'},//定义内容格式,很重要
                timeout: 20000,
            }).then((res) =>
                {
                    this.responseText = res.data;
                    this.myOptionList = [];
                    this.show_result(this.responseText);
                  this.$message.success('分析完成!');
                }).catch(err =>{console.log(err);
                    this.myOptionList=[];
                })
        },
    },
};
</script>
