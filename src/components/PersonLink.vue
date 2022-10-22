<script>
import * as d3 from 'd3'
export default {
    props: {
        currentDate: {
            type: [Object,String],
            required: true,
        }
    }
    ,
    data() {
        return {
            nodes: [],
            edges: []
        }
    }
    ,
    mounted() {
        this.createSvg()
        this.handleData()
        this.draw()
    },
    beforeUpdate() {
        this.clearSvg()
        this.draw()
        // this.handleData()
        console.log("PersonLink更新")
    }
    ,
    watch:{
        currentDate(){
            // 监听props currentData 变化，更新state，千万不能在beforeUpdata中更新state
            console.log('currentData Update',this.currentDate)
            this.handleData()
        }
    },
    methods: {
        createSvg() {
            // 1.创建SVG
            const width = document.getElementById('link').offsetWidth
            const height = document.getElementById('link').offsetHeight
            
            const svg = d3.select('#link')
                .append('svg')
                .attr('width', width)
                .attr('height', height)
                .attr('id', 'svg_link')
                .style('background-color', '#fff')
        }
        ,
        ticked(links,linksText,nodesChart) {
                links
                    .attr('x1', function (d) {
                        return d.source.x
                        // console.log('line',d)
                    })
                    .attr('y1', function (d) {
                        return d.source.y
                    })
                    .attr('x2', function (d) {
                        return d.target.x
                    })
                    .attr('y2', function (d) {
                        return d.target.y
                    })

                linksText
                    .attr('x', function (d) {
                        return (d.source.x + d.target.x) / 2
                    })
                    .attr('y', function (d) {
                        return (d.source.y + d.target.y) / 2
                    })

                nodesChart.attr('transform', function (d) {
                    return 'translate(' + d.x + ',' + d.y + ')'
                })
            },
        draw() {
            const width = d3.select('#svg_link').attr('width')
            const height = d3.select('#svg_link').attr('height')

            const chart = d3.select('#svg_link').append('g').attr('id','svg_link_chart')
            // 创建颜色比例尺
            const colorScale = d3.scaleOrdinal(d3.quantize(d3.interpolateRainbow, this.nodes.length + 1))
            // 创建力模拟
            const force = d3.forceSimulation()
                .force('link', d3.forceLink())
                .force('charge', d3.forceManyBody())
                .force('collide',d3.forceCollide(20))
                .force('center', d3.forceCenter(width / 2, height / 2))
            // d3.simulation 内置的internal timer在模拟的过程中不断触发的 'tick'事件
            // 利用点数组初始化一个力Simulation，为nodes添加 index x y vx vy 5个属性(fx fy属性自行指定)，监听'tick'事件
            const forceNodes = force.nodes(this.nodes).on('tick', () => {this.ticked(links,linksText,nodesChart)})
            // console.log(this.nodes)

            // 利用边数组生成链接力，为edges添加 index 3个属性
            force.force('link')
                .links(this.edges)
                // 没有调用id方法，则每一个node的id(非属性)默认为自身的index值，后续向edges数组中添加的source target属性必须采用与node的id相对应的的值
                .distance(function (d) {
                    //每一边的长度
                    return d.value * 50
                })
            // console.log(this.nodes) // 没有id属性
            
            // 将线以及线上的文本放入共同的各自的g标签中，便于力模拟时管理
            const line = chart.append('g').selectAll().data(this.edges).enter().append('g')
            const links = line.append('line').attr('stroke', '#ccc').attr('stroke-width', 1)
            const linksText = line
                .append('text')
                .text(function (d) {
                    return d.value
                })
                .attr('fill', '#ccc')
            
            // 创建节点绘制组
            const nodesChart = chart
                .append('g')
                .selectAll()
                .data(this.nodes)
                .join('g')
                .attr('transform', function (d, i) {
                    // console.log('g-circle',d)
                    return 'translate(' + d.x + ',' + d.y + ')'
                })

            nodesChart
                .append('circle')
                // .on('mouseover',function(e,d){
                //     // 显示李白与人物发生的事
                //     d3.select(this)
                // })
                .attr('r', function (d, i) {
                    // 半径
                    return d.radius * 10
                })
                .attr('fill', function (d, i) {
                    return colorScale(d.name)
                })

            nodesChart
                .append('text')
                .attr('x', -20)
                .attr('y', -5)
                .attr('dy', 10)
                .attr('font-size', 12)
                .text(function (d) {
                    return d.name
                })
                .attr('fill', '#ccc')
                .attr('pointer-events', 'none')
                .style('user-select', 'none')
            
            nodesChart
                .call(this.drag(force))
        },
        handleData() {
            // 将currentDate处理成绘制力导向图所需的编辑和点集
            // console.log(this.currentDate)
            if (JSON.stringify(this.currentDate) === '{}' || JSON.stringify(this.currentDate['人物']) === '{}') {// 若是空对象或者 人物 属性为空对象
                // 点集长度为1 ，边集长度为0
                this.nodes = [{ name: '李白', radius: 4 }]
                this.edges = []
            } else {// 存在人物
                const nodes = [{ name: '李白', radius: 4 }]
                const edges = []
                const person = this.currentDate['人物']
                Object.keys(person).forEach((key,i) => {
                    nodes.push({ name:key,radius:2,text:person[key] })
                    edges.push({ source: 0, target: i+1, value: 2, text: person[key] })
                })
                // 更新state
                this.nodes = nodes
                this.edges = edges
            }
        },
        clearSvg(){
            d3.select('#svg_link_chart').remove()
        },
        drag(simulation){
            function dragStart(e,d){
                console.log(e.active)
                simulation.alphaTarget(0.8).restart()
                d.fx=d.x
                d.fy=d.y
            }
            function dragging(e,d){
                d.fx=e.x
                d.fy=e.y
            }
            function dragEnd(e,d){
                simulation.alphaTarget(0).restart()
                d.fx=null
                d.fy=null
            }
            return d3.drag().on('start',dragStart).on('drag',dragging).on('end',dragEnd)
        },
    }
}
</script>

<template>
    <div id="link"></div>

</template>

<style scoped>
#link {
    /* flex: none; */
    width: 70%;
    height: 100%;
}
</style>