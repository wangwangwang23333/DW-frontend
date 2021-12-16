<template>
  <div class="app-container">
    <el-dialog
      title="电影详情"
      :visible.sync="dialogVisible"
      width="25%"
      center
      @close="dialogVisible=false;"
    >
      <div>
        <p>asin:{{dialogData.asin}}</p>
        
        <p>电影名:{{dialogData.name}}</p>
        <p v-if="dialogData.director.length!==0" >
          导演：
          <span v-for="i in dialogData.director">{{ i }}, </span>
        </p>
        <p v-if="dialogData.mainActor.length!==0" >
          主演：
          <span v-for="i in dialogData.mainActor">{{ i }}, </span>
        </p>
        <p v-if="dialogData.actor.length!==0" >
          演员：
          <span v-for="i in dialogData.actor">{{ i }}, </span>
        </p>
        <p>评分:{{dialogData.score}}</p>
        <p>评论总数:{{dialogData.commentNum}}</p>
        
      </div>
      <div slot="footer">
        <el-button type="primary" @click="viewOriginWeb()">查看原始网页</el-button>
        <br><br>
        <el-button @click="dialogVisible = false">关闭</el-button>
      </div>
    </el-dialog>

    <el-row style="height: 50vh;">
      <el-col :span="12">
        <el-form>
          <el-form-item>
            <el-button @click="mostCooperateActorsButton">合作次数最多的演员</el-button>
          </el-form-item>
          <el-form-item>
            <el-button @click="mostCooperateActorAndDirectorButton">合作次数最多的演员和导演</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="1">
        <el-divider direction="vertical"/>
      </el-col>
      <el-col :span="10">
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="查询结果" name="first" v-loading="loading">
            {{ searchResult }}

          </el-tab-pane>
          <el-tab-pane label="速度对比" name="third" :disabled="!hasResult">

            <ve-histogram
              class="myve"
              :data="chartData"
              :settings="vchartsConfig.setting"
              :extend="vchartsConfig.extend"
              width="38vw"
              height = "50vh"
              v-loading="loading"
            />
          </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
    <el-divider />

    <div
      id="network_id"
      slot="reference"
      v-loading="loading"
      class="network"
      element-loading-text="正在搜索中"
      element-loading-background="rgba(0, 0, 0, 0.1)"
      style="height:80vh;
      background: linear-gradient(to bottom, #536976, #292E49);z-index:10;"
      :span="24"
    >
      <el-image
        v-if="nodes.length == 0 && !loading"
        :src="require('@/assets/finderBlank.png')"
        style="
    height:40vh;margin-left: 20%;
    margin-top:-80%;z-index: 999;"
      />
    </div>
  </div>
</template>

<script>
import Vis from 'vis'

const BASE_URL = 'http://localhost:8101'

export default {

  data() {
    return {
      hasResult: false,
      loading: false,
      searchResult: '暂无查询',
      // 速度比较图
      vchartsConfig: {
        setting: {
          // 别称
          labelMap: {
            'type': '数据库',
            'software': '软件',
            'speed': '速度'
          }
        },
        extend: {
          title: {
            show: true,
            text: '',
            subtext: '通过关系型数据库MySql、分布式数据库Hive和图数据库Neo4j进行检索'
            // textAlign:'center',
          },
          // 图标顶部的标题及按钮
          legend: {
            show: false
          },
          // backgroundColor:'red',//整个组件的背景颜色
          // X轴线
          xAxis: {
            // name: "地区",
            type: 'category',
            show: true,
            // 坐标轴轴线
            axisLine: {
              show: false
            },
            // 坐标轴每项的文字
            axisLabel: {
              showMaxLabel: true,
              showMinLabel: true,
              color: '#3a3a3a',
              rotate: 0, // 刻度文字旋转，防止文字过多不显示
              margin: 8, // 文字离x轴的距离
              boundaryGap: true,
              // backgroundColor:'#0f0',
              formatter: (v) => {
                // console.log('x--v',v)
                return v
              }
            },
            // X轴下面的刻度小竖线
            axisTick: {
              show: false,
              alignWithLabel: true, // axisLabel.boundaryGap=true时有效
              interval: 0,
              length: 4// 长度
            },
            // x轴对应的竖线
            splitLine: {
              show: false,
              interval: 0,
              lineStyle: {
                color: 'red',
                backgroundColor: 'red'
              }
            }
          },
          yAxis: {
            show: true,
            offset: 0,
            // 坐标轴轴线
            axisLine: {
              show: false
            },
            // x轴对应的竖线
            splitLine: {
              show: true
            },
            // 坐标轴刻度
            axisTick: {
              show: false
            },
            boundaryGap: true,
            axisLabel: {
              color: '#3a3a3a',
              formatter: (v) => {
                if (v === 0) {
                  return v
                }
                return v + ' ms'
              }
            }

          },

          // 滚动组件参数
          dataZoom: [
            {
              type: 'inside',
              show: true,
              xAxisIndex: [0],
              startValue: 0,
              endValue: 4,
              zoomLock: true// 阻止区域缩放
            }
          ],

          // 每个柱子
          series(v) {
            // console.log("v", v);
            // 设置柱子的样式
            v.forEach(i => {
              console.log('series', i)
              i.barWidth = 30
              i.itemStyle = {
                barBorderRadius: [15, 15, 0, 0],
                borderWidth: 20
              }
              i.label = {
                color: '#666',
                show: true,
                position: 'top'
                // backgroundColor:'yellow',
              }
            })
            return v
          }
        }
      },
      // v-chats列表数据
      chartData: {
        columns: ['type', 'speed'],
        rows: [
          { 'type': '关系型数据库', 'software': 'mysql', 'speed': 0 },
          { 'type': '分布式数据库', 'software': 'hive', 'speed': 0 },
          { 'type': '图数据库', 'software': 'neo4j', 'speed': 0 }
        ]
      },
      activeName: 'first',

      nodeMap: null,
      nodes: [],
      nodesArray: [],
      edges: [],
      edgesArray: [],
      container: null,
      options: {
        autoResize: true, // 网络将自动检测其容器的大小调整，并相应地重绘自身
        locale: 'cn', // 语言设置：工具栏显示中文
        // 设置语言
        locales: {
          cn: {
            // 工具栏中文翻译
            edit: '编辑',
            del: '删除当前节点或关系',
            back: '返回',
            addNode: '添加节点',
            addEdge: '添加连线',
            editNode: '编辑节点',
            editEdge: '编辑连线',
            addDescription: '点击空白处可添加节点',
            edgeDescription: '点击某个节点拖拽连线可连接另一个节点',
            editEdgeDescription: '可拖拽连线改变关系',
            createEdgeError: '无法将边连接到集群',
            deleteClusterError: '无法删除集群.',
            editClusterError: "无法编辑群集'"
          }
        },
        // 设置节点样式
        nodes: {
          shape: 'circle',
          size: 15,
          font: {
            // 字体配置
            size: 15
          },
          color: {
            border: '#f1e7ea', // 节点边框颜色
            background: '#97C2FC', // 节点背景颜色
            highlight: {
              // 节点选中时状态颜色
              border: '#f1e7ea',
              background: '#D2E5FF'
            },
            hover: {
              // 节点鼠标滑过时状态颜色
              border: '#f1e7ea',
              background: '#D2E5FF'
            }
          },
          borderWidth: 3, // 节点边框宽度，单位为px
          borderWidthSelected: 5 // 节点被选中时边框的宽度，单位为px
        },
        // 边线配置
        edges: {
          width: 3,
          length: 300,
          color: {
            color: '#f1e7ea',
            highlight: '#f1f7fa',
            hover: '#f1f7fa',
            inherit: 'from',
            opacity: 1.0
          },
          shadow: true,
          smooth: {
            // 设置两个节点之前的连线的状态
            enabled: true // 默认是true，设置为false之后，两个节点之前的连线始终为直线，不会出现贝塞尔曲线
          },
          arrows: { to: true } // 箭头指向to
        },
        // 计算节点之前斥力，进行自动排列的属性
        physics: {
          enabled: true, // 默认是true，设置为false后，节点将不会自动改变，拖动谁谁动。不影响其他的节点
          barnesHut: {
            gravitationalConstant: -4000,
            centralGravity: 0.3,
            springLength: 120,
            springConstant: 0.04,
            damping: 0.09,
            avoidOverlap: 0
          }
        },
        // 用于所有用户与网络的交互。处理鼠标和触摸事件以及导航按钮和弹出窗口
        interaction: {
          dragNodes: true, // 是否能拖动节点
          dragView: true, // 是否能拖动画布
          hover: true, // 鼠标移过后加粗该节点和连接线
          multiselect: true, // 按 ctrl 多选
          selectable: true, // 是否可以点击选择
          selectConnectedEdges: true, // 选择节点后是否显示连接线
          hoverConnectedEdges: true, // 鼠标滑动节点后是否显示连接线
          zoomView: true // 是否能缩放画布
        },
        // 操作模块:包括 添加、删除、获取选中点、设置选中点、拖拽系列、点击等等
        manipulation: {
          enabled: false, // 该属性表示可以编辑，出现编辑操作按钮
          addNode: true,
          addEdge: true,
          editEdge: true,
          deleteNode: true,
          deleteEdge: true
        },
        layout: {
          improvedLayout: false
        }
      },
      visData: {},

      dialogVisible: false,
      dialogData:{
        asin:'',
        name:'',
        director:[],
        actor:[],
        mainActor:[],
        score:'',
        commentNum:0,
      },

      personColor: {
        background: '#f57797',
        highlight: '#fbc7d4',
        hover: '#fbc7d4'
      },
      movieColor: {
        background: '#7574eb',
        highlight: '#b9b8f5',
        hover: '#b9b8f5'
      }
    }
  },
  watch: {

  },

  created() {
    this.nodeMap = new Map()
  },

  mounted() {
    this.initializeOptions()
  },

  methods: {
    viewOriginWeb() {

    },

    handleClick(tab, event) {
      console.log(tab, event)
    },

    clickSearch() {
      // 清空上一轮的搜索结果

      this.initializeOptions()
    },
    mostCooperateActorAndDirectorButton() {
      this.initializeOptions()

      var axios = require('axios')
      this.loading = true
      var config = {
        method: 'get',
        url: BASE_URL + '/neo4j/relation/actorAndDirector',
        headers: { }
      }
      this.vchartsConfig.extend.title.text="演员和导演之间的合作关系检索结果"
      this.hasResult = true
      // neo4j 查询
      axios(config)
        .then(response => {
          console.log(response.data)
          this.chartData.rows[2].speed = response.data.time
          this.searchResult = "名为"+response.data.actor+"的演员和名为"+response.data.director+"的导演,"+"共合作"+response.data.number+"次."
          axios({
            method: 'get',
            url: BASE_URL + '/mysql/association/actor/director/cooperation',
            headers: {}
          }).then(response=>{
            this.chartData.rows[0].speed = response.data.time
            this.vchartsConfig.extend.title.text="演员和导演之间的合作关系检索结果"
          })

          // 将返回值添加成两个结点
          let newNode = {
            id: 0,
            label: response.data.actor,
            color: this.personColor,
            type: 'actor'
          }
          this.nodes.add(newNode)
          this.nodesArray.push(newNode)

          newNode = {
            id: 1,
            label: response.data.director,
            color: this.personColor,
            type: 'actor'
          }
          this.nodes.add(newNode)
          this.nodesArray.push(newNode)

          // 根据导演和演员获取它们出演过的电影
          axios({
            method: 'get',
            url: BASE_URL + '/mysql/association/movie/actorAndDirector',
            params: { 'actorName': response.data.actor, 'directorName': response.data.director },
            headers: {}
          })
            .then(res => {
              const movieList = res.data
              // 添加电影结点
              for (let i = 0; i < movieList.length; ++i) {
                const newNode = {
                  id: i + 2,
                  label: movieList[i].name.substring(0, 4) + '...',
                  color: this.movieColor,
                  type: 'movie',
                  movieName: movieList[i].name,
                  movieAsin: movieList[i].asin,
                  score:movieList[i].score,
                  commentNum:movieList[i].commentNum,
                }
                this.nodes.add(newNode)
                this.nodesArray.push(newNode)
                // 两个演员和它们之间的关系
                let newEdge = {
                  from: 0,
                  to: i + 2,
                  label: 'Act'
                }
                this.edges.add(newEdge)
                newEdge = {
                  from: 1,
                  to: i + 2,
                  label: 'Direct'
                }
                this.edges.add(newEdge)
              }
              this.loading = false
            })
        })
        .catch(function(error) {
          console.log(error)
        })
    },

    mostCooperateActorsButton() {
      this.initializeOptions()

      var axios = require('axios')

      var config = {
        method: 'get',
        url: BASE_URL + '/neo4j/relation/actors',
        headers: { }
      }
      this.loading = true
      // neo4j 查询
      axios(config)
        .then(response => {
        // 将返回值添加成两个结点
          for (let i = 0; i < response.data.actor.length; ++i) {
            const newNode = {
              id: i,
              label: response.data.actor[i],
              color: this.personColor,
              type: 'actor'
            }
            this.nodes.add(newNode)
            this.nodesArray.push(newNode)
          }
          //获取查询结果
          this.searchResult = "名为"+response.data.actor[0]+"的演员和名为"+response.data.actor[1]+"的演员,"+"共合作"+response.data.number+"次."
          this.hasResult = true
          //防止查询速度结果
          this.chartData.rows[2].speed = response.data.time
          axios({
            method: 'get',
            url: BASE_URL + '/mysql/association/actor/cooperation',
            headers: {}
          }).then(response=>{
            this.chartData.rows[0].speed = response.data.time
            this.vchartsConfig.extend.title.text="演员之间的合作关系检索结果"
          })
          // 根据两个演员获取它们出演过的电影
          axios({
            method: 'get',
            url: BASE_URL + '/mysql/association/movie/actors',
            params: { 'actor1': response.data.actor[0], 'actor2': response.data.actor[1] },
            headers: {}
          })
            .then(res => {
              const movieList = res.data
              // 添加电影结点
              for (let i = 0; i < movieList.length; ++i) {
                const newNode = {
                  id: i + 2,
                  label: movieList[i].name.substring(0, 4) + '...',
                  color: this.movieColor,
                  type: 'movie',
                  movieName: movieList[i].name,
                  movieAsin: movieList[i].asin,
                  score:movieList[i].score,
                  commentNum:movieList[i].commentNum,
                }
                this.nodes.add(newNode)
                this.nodesArray.push(newNode)
                // 两个演员和它们之间的关系
                let newEdge = {
                  from: 0,
                  to: i + 2,
                  label: 'Act'
                }
                this.edges.add(newEdge)
                newEdge = {
                  from: 1,
                  to: i + 2,
                  label: 'Act'
                }
                this.edges.add(newEdge)
              }
              this.loading = false
            })
        })
        .catch(function(error) {
          console.log(error)
        })
    },

    initializeOptions() {
      this.nodes = new Vis.DataSet()

      // 创建边数据数组
      this.edges = new Vis.DataSet()

      // 获取容器
      var container = document.getElementById('network_id')

      // 将数据赋值给vis 数据格式化器
      var data = {
        nodes: this.nodes,
        edges: this.edges
      }

      // 初始化关系图
      var network = new Vis.Network(container, data, this.options)
      network.on('click', params => {
        const nodeId = network.getNodeAt(params.pointer.DOM)
        // eslint-disable-next-line eqeqeq
        if (nodeId != undefined) {
          network.selectNodes([nodeId])
          const selectedIndex = network.getSelectedNodes()[0]
          console.log(this.nodesArray[selectedIndex])
          // eslint-disable-next-line eqeqeq
          if (this.nodesArray[selectedIndex].type === 'actor') {
            return
            // eslint-disable-next-line eqeqeq
          } else if (this.nodesArray[selectedIndex].type === 'movie') {
            this.dialogVisible = true
            this.dialogData.asin=this.nodesArray[selectedIndex].movieAsin
            this.dialogData.name=this.nodesArray[selectedIndex].movieName
            this.dialogData.score=this.nodesArray[selectedIndex].score
            this.dialogData.commentNum=this.nodesArray[selectedIndex].commentNum
            // 根据asin获取详细信息(主演、演员和导演)
            var axios = require('axios');
            axios({
              method: 'get',
              url: BASE_URL + '/mysql/association/movie/director',
              params:{"movieAsin":this.dialogData.asin, "index": 0},
              headers: {}
            })
            .then(response => {
              this.dialogData.director=response.data.director
            })
            axios({
              method: 'get',
              url: BASE_URL + '/mysql/association/movie/mainActor',
              params:{"movieAsin":this.dialogData.asin, "index": 0},
              headers: {}
            })
            .then(response => {
              this.dialogData.mainActor=response.data.mainActor
            })
            axios({
              method: 'get',
              url: BASE_URL + '/mysql/association/movie/actor',
              params:{"movieAsin":this.dialogData.asin, "index": 0},
              headers: {}
            })
            .then(response => {
              this.dialogData.actor=response.data.actor
            })
            return
          }
        }
      })
    }

  }
}
</script>

<style scoped>
.el-divider--vertical{
    height:50vh;
  }
</style>
