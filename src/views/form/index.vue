<template>
  <div class="app-container">
    <div id="myChart" style="width: 95vw;height: 50vh;"></div>
    <el-divider></el-divider>
  </div>
</template>

<script>

export default {
  data() {
    return {

    }
  },
  methods: {
   
  },
  created(){

  },
  mounted(){
    let myChart = this.$echarts.init(document.getElementById('myChart'))

    let option = {
      tooltip: {
        trigger: 'item',
        triggerOn: 'mousemove'
      },
      
      series: {
        type: 'sankey',
        layout: 'none',
        emphasis: {
          focus: 'adjacency'
        },
        selectedMode:"single",
        select:{
          show:true,
          position:'top',
        },
        lineStyle: {
          color: 'source',
          curveness: 0.5
        },
        data: [
          {
            name: '评论数据'
          },
          {
            name: 'Amazon数据'
          },
          {
            name: '404数据'
          },
          {
            name: '电影数据'
          },
          {
            name: '非电影数据'
          },
          {
            name:"合并冲突后的数据"
          }

        ],
        links: [
          {
            source: '评论数据',
            target: 'Amazon数据',
            value: 5000
          },
          {
            source: '评论数据',
            target: '404数据',
            value: 5
          },
          {
            source: 'Amazon数据',
            target: '电影数据',
            value: 3500
          },
          {
            source: 'Amazon数据',
            target: '非电影数据',
            value: 500
          },
          {
            source: '电影数据',
            target: '合并冲突后的数据',
            value: 3000
          },
        ]
      }
    }
    // 绘制图表
    myChart.setOption(option);
    myChart.on('click',param=>{
      console.log(param.data)
    })
  },
}
</script>

<style scoped>
.line{
  text-align: center;
}
</style>

