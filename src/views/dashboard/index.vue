<template>
  <div class="dashboard-container">
    <!-- <div class="dashboard-text">name: {{ name }}</div>
    <div>通过爬虫获取数据</div> -->
    <mavon-editor style="z-index:0" 
    v-model="value" 
    :subfield="false" 
    :defaultOpen="'preview'" 
    :editable='false'
    :toolbarsFlag='false' 
    :navigation='false' />
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import {mavonEditor} from 'mavon-editor'
import 'mavon-editor/dist/css/index.css'
export default {
  name: 'Dashboard',
  computed: {
    ...mapGetters([
      'name'
    ])
  },
  data(){
    return{
      value:"## 你好\n你好"
    }
  },
  components: { mavonEditor},
  created(){
    var axios = require('axios');

    var config = {
      method: 'get',
      url: 'https://tongjigohome.oss-cn-shanghai.aliyuncs.com/docx/docx.md',
      headers: { }
    };

    axios(config)
    .then((response)=>{
      this.value=response.data;
    })
    .catch(function (error) {
      console.log(error);
    });
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
