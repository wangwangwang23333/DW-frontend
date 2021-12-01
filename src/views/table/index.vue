
<template>
  <div class="app-container">
    <el-row>
      <el-col :span="14">
        <el-form ref="form" :model="form" label-width="120px">
          <el-form-item label="电影名称">
            <el-input v-model="form.name" />
          </el-form-item>
    
          <el-form-item label="电影类别">
            <el-select v-model="form.region" placeholder="please select your zone">
              <el-option label="Zone one" value="shanghai" />
              <el-option label="Zone two" value="beijing" />
            </el-select>
          </el-form-item>
          <el-form-item label="上映时间">
            <el-date-picker v-model="form.movieDate" type="daterange" align="right" unlink-panels range-separator="至"
                start-placeholder="开始日期" end-placeholder="结束日期" 
                :picker-options="pickerOptions">
              </el-date-picker>
          </el-form-item>
          <el-form-item label="导演">
            <el-tag
              :key="tag"
              v-for="(tag,index) in form.movieDirectors"
              closable
              effect="dark"
              :disable-transitions="false"
              :color="labelColor[index%labelColor.length]"
              style="color: white;"
              :hit="true"
              @close="handleDirectorTagClose(tag)">
              {{tag}}
            </el-tag>
            <el-input
                class="input-new-tag"
                v-if="directorInputVisible"
                v-model="directorInputValue"
                ref="saveDirectorTagInput"
                size="small"
                @keyup.enter.native="handleDirectorInputConfirm"
                @blur="handleDirectorInputConfirm"
              >
              </el-input>
              <el-button v-if="!directorInputVisible && form.movieDirectors.length<5" class="button-new-tag" size="small" 
              @click="showDirectorInput()">
                添加导演
              </el-button>
          </el-form-item>
          <el-form-item label="主演">
            <el-tag
              :key="tag"
              v-for="(tag,index) in form.movieMainActors"
              closable
              effect="dark"
              :disable-transitions="false"
              :color="labelColor[index%labelColor.length]"
              style="color: white;"
              :hit="true"
              @close="handleMainActorTagClose(tag)">
              {{tag}}
            </el-tag>
            <el-input
                class="input-new-tag"
                v-if="mainActorInputVisible"
                v-model="mainActorInputValue"
                ref="saveMainActorTagInput"
                size="small"
                @keyup.enter.native="handleMainActorInputConfirm"
                @blur="handleMainActorInputConfirm"
              >
              </el-input>
              <el-button v-if="!mainActorInputVisible && form.movieMainActors.length<5" class="button-new-tag" size="small" 
              @click="showMainActorInput()">
                添加主演
              </el-button>
          </el-form-item>
          
          <el-form-item label="评分">
            <el-input-number v-model="form.movieMinScore" :precision="2" :step="0.01" :max="form.movieMaxScore" :min="0" />
            <el-input-number v-model="form.movieMaxScore" :precision="2" :step="0.01" :max="5" :min="form.movieMinScore" />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" >Create</el-button>
            <el-button @click="onCancel">Cancel</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="10">
        测试
      </el-col>
    </el-row>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  filters: {
  
  },
  data() {
    return {
      form: {
        name: '',
        region: '',
        delivery: false,
        type: [],
        resource: '',
        desc: '',
        movieDirectors:[],
        movieMainActors:[],
        movieActors:[],
        actorName:'',
        movieMinScore:2.7,
        movieMaxScore:5.0,
        movieDate:'',
      },
      labelColor:["#77C9D4","#57BC90","#015249"],
      pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          },{
            text: '最近半年',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 182);
              picker.$emit('pick', [start, end]);
            }
          },
          {
            text: '最近一年',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 365);
              picker.$emit('pick', [start, end]);
            }
          },
        ]
      },
      directorInputVisible:false,
      directorInputValue:'',
      mainActorInputVisible:false,
      mainActorInputValue:'',
      
      }
  },
  created() {
    
  },
  methods: {
    onCancel() {
      this.$message({
        message: 'cancel!',
        type: 'warning'
      })
    },
    showDirectorInput() {
      this.directorInputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveDirectorTagInput.$refs.input.focus();
      });
    },
    handleDirectorInputConfirm() {
      let inputValue = this.directorInputValue
      // 有效性判断
      if (!inputValue || inputValue.replace(/\s*/g,"").length==0) {
        if(!this.directorInputVisible){
          return;
        }
        this.$message({
          message: '请输入有效的导演名称！',
          type: 'warning'
        })
        return;
      }
      this.form.movieDirectors.push(inputValue.replace(/^\s*|\s*$/g,""));
      this.directorInputVisible = false;
      this.directorInputValue = '';
    },
    handleDirectorTagClose(tag) {
      this.form.movieDirectors.splice(this.form.movieDirectors.indexOf(tag), 1);
    },
    showMainActorInput() {
      this.mainActorInputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveMainActorTagInput.$refs.input.focus();
      });
    },
    handleMainActorInputConfirm() {
      let inputValue = this.mainActorInputValue
      // 有效性判断
      if (!inputValue || inputValue.replace(/\s*/g,"").length==0) {
        if(!this.mainActorInputVisible){
          return;
        }
        this.$message({
          message: '请输入有效的主演名称！',
          type: 'warning'
        })
        return;
      }
      this.form.movieMainActors.push(inputValue.replace(/^\s*|\s*$/g,""));
      this.mainActorInputVisible = false;
      this.mainActorInputValue = '';
    },
    handleMainActorTagClose(tag) {
      this.form.movieMainActors.splice(this.form.movieMainActors.indexOf(tag), 1);
    },
  }
}
</script>

<style scoped>
   .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>