<template>
  <div class="app-container">
    <el-row>
      <el-col :span="12">
        <el-form>
          <el-form-item>
            <el-button>合作次数最多的演员</el-button>
          </el-form-item>
          <el-form-item>
            <el-button>合作次数最多的导演</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="1">
        <el-divider direction="vertical"></el-divider>
      </el-col>
      <el-col :span="10">
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="查询结果" name="first">
            用户管理

          </el-tab-pane>
          <el-tab-pane label="数据血缘" name="second">
            配置管理
          </el-tab-pane>
          <el-tab-pane label="速度对比" name="third" :disabled="!hasResult">

            <ve-histogram
              class="myve"
              :data="chartData"
              :settings="vchartsConfig.setting"
              :extend="vchartsConfig.extend"
              width="38vw"
            ></ve-histogram>  
          </el-tab-pane>
       </el-tabs>
      </el-col>
    </el-row>

  </div>
</template>

<script>
export default {

  data() {
    return {
      hasResult:false,
      // 速度比较图
      vchartsConfig: {
        setting:{
          // 别称
          labelMap: {
            'type': '数据库',
            'software': '软件',
            'speed': '速度',
          },
        },
        extend: {
          title:{
            show:true,
            text:'检索电影',
            subtext:'通过关系型数据库MySql、分布式数据库Hive和图数据库Neo4j分别检索电影的速度对比',
            // textAlign:'center',
          },
          // 图标顶部的标题及按钮
          legend:{
            show:false,
          },
          // backgroundColor:'red',//整个组件的背景颜色
          //X轴线
          xAxis: {
            // name: "地区",
            type:'category',
            show:true,
            // 坐标轴轴线
            axisLine:{
              show:false,
            },
            // 坐标轴刻度
            axisTick:{
              show:false,
            },
            // 坐标轴每项的文字
            axisLabel:{
              showMaxLabel:true,
              showMinLabel:true,
              color:'#3a3a3a',
              rotate:0, //刻度文字旋转，防止文字过多不显示
              margin:8,//文字离x轴的距离
              boundaryGap:true,
              // backgroundColor:'#0f0',
              formatter:(v)=>{
                // console.log('x--v',v)
                return v
              },
            },
            // X轴下面的刻度小竖线
            axisTick:{
              show:false,
              alignWithLabel:true,//axisLabel.boundaryGap=true时有效
              interval:0,
              length:4,//长度
            },
            // x轴对应的竖线
            splitLine: {
              show: false,
              interval: 0,
              lineStyle:{
                color:'red',
                backgroundColor:'red',
              }
            }
          },
          yAxis: {
            show:true,
            offset:0,
            // 坐标轴轴线
            axisLine:{
              show:false,
            },
            // x轴对应的竖线
            splitLine: {
              show: true,
            },
            // 坐标轴刻度
            axisTick:{
              show:false,
            },
            boundaryGap:true,
            axisLabel:{
              color:'#3a3a3a',
              formatter:(v)=>{
                if (v==0){
                  return v;
                }
                return v+' ms'
            },
            },
            
          },
          
          // 滚动组件参数
          dataZoom:[
            {
              type: 'inside',
              show: true,
              xAxisIndex: [0],
              startValue: 0,
              endValue: 4,
              zoomLock:true,//阻止区域缩放
            }
          ],



          // 每个柱子
          series(v) {
            // console.log("v", v);
            // 设置柱子的样式
            v.forEach(i => {
              console.log("series", i);
              i.barWidth = 30;
              i.itemStyle={
                barBorderRadius:[15,15,0,0],
                borderWidth:20,
              };
              i.label={
                color:'#666',
                show:true,
                position:'top',
                // backgroundColor:'yellow',
              };

            });
            return v;
          },
        }
      },
      // v-chats列表数据
      chartData: {
        columns: ["type","speed"],
        rows: [
          { "type":"关系型数据库","software": "mysql", "speed": 0 },
          { "type":"分布式数据库","software": "hive", "speed": 0 },
          { "type":"图数据库","software": "neo4j", "speed": 0 },
        ],
      },
      activeName: 'first',
    }
  },
  watch: {
   
  },

  methods: {
    handleClick(tab, event) {
      console.log(tab, event);
    }
  }
}
</script>

