<script>
import * as d3 from 'd3'
// 处理数据，获取每年的诗词数量总和

// let topTen=[['init',75],['test',25]];
// d3.json('src/assets/libai/libai_merge_emotion.json',res => {
//     poetryList = res
//     console.log(poetryList)

// })

// poetryList = getPoetryList()
export default {
    props:{
        yearPoetryNum:{
            type:Array,
            required:true
        },
        selectedYear:{
            type:Number,
            required:true
        }
    },
    emits:['changeYear'],
    data() {
        return {
            topTen:[]
        }
    },
    mounted() {
        // 组件首次加载，传入的props或watch不会触发更新，若没有修改data，组件不会进入更新，刷新页面相当于重新挂载组件。
        console.log('piechart mounted executing~')
        this.createSvg()
        this.handleData()
        this.drawPieChart()
    },
    beforeUpdate(){
        console.log('piechart beforupdata executing~')
        this.handleData()// 在这里更新状态是安全的
        this.clearSvg()
        this.drawPieChart()
    },
    methods: {
        // 弧形动画，鼠标移入，扇形放大，鼠标移出，扇形恢复
        arcTween(outerRadius,newOuterRadius){
            return function(){// 事件监听函数
                d3.select(this)// ?
                .transition()
                .duration(500)
                .attrTween('d',function(d){
                    let interpolate = d3.interpolate(outerRadius,outerRadius+newOuterRadius)
                    return function(t){
                        let arcT = d3.arc().innerRadius(0).outerRadius(interpolate(t))//创建放大的弧度生成器
                        return arcT(d)
                    }
                })
            }
        },
        drawPieChart() {
            console.log('piechart drawPieChart executing~')
            const _this = this
            // 0.创建饼布局 分别设置排序方式 绑定一个计算扇形区值的函数 处理数据为饼图数据
            // sort(null) 创建布局时，默认按照从小到大，设置为null则按照给定数组顺序
            const pie = d3.pie().sort(null).value(function (d) {
                return d[1]// ?
            })
            const pieData = pie(this.topTen)
            console.log('piechart topTen',this.topTen)
            console.log('piechart pieData',pieData)

            const svg = d3.select('#svg_pie_1')
            const edge = svg.attr('width')
            
            // 2.创建新的弧度生成器，设置内外部半径
            const innerRadius = 0
            const outerRadius = edge / 4
            const arc = d3.arc().outerRadius(outerRadius).innerRadius(innerRadius)

            // 3.创建分组并绑定数据
            const arcs = svg.append('g').attr('id','svg_pie_1_chart').selectAll('g')// arcs是g标签选择集
                .data(pieData)
                .enter()
                .append('g')
                .attr('transform', 'translate(' + edge / 2 + ',' + edge / 2 + ')')
            
            // 4.调用内置颜色序列 添加交互 添加动画
            const colors = d3.schemeCategory10
            // 保存已选择的扇区
            let selectedSector = null
            arcs.append('path')// 'path'标签是？ 在g标签中创建path标签
                .attr('fill', function (data, index) {
                    return colors[index]
                })
                // .attr('class','pie_1')
                .attr('d', data => arc(data))// 设置path标签'd'属性,值为弧度生成器arc处理后的数据
                .classed('pie_1_s',function(d,i){
                    if(_this.selectedYear == d.data['0']) {
                        selectedSector = d3.select(this)
                        // console.log('selected',this,i)
                        return true
                    }else {
                        // console.log('unselected',this,i)
                        return false
                    }
                })
                .on('mouseover',this.arcTween(outerRadius,20))
                .on('mouseout',this.arcTween(outerRadius,0))
                .on('click',function(e,d){
                    console.log('clicked',this)
                        _this.$emit('changeYear',d.data['0'])// 发射点击事件 输出年份 字符串；类型

                        // 删除其他的选中类 pie_1_s
                        // d3.selectAll('.pie_1').attr('class','pie_1')
                        selectedSector && selectedSector.classed('pie_1_s',false)

                        // 为当前点击的path添加类 pie_1_s
                        // console.log(this)
                        selectedSector = d3.select(this).classed('pie_1_s',true)
                        // console.log(selectedSector);
                    }
                )
                .transition()
                .duration(1500)
                .attrTween('d', d => {// 对d属性使用给定的插值器
                    // 该插值器返回给定两个对象中对应字段之间的值，返回的仍是饼图数据(pie(data))类型
                    let interpolate = d3.interpolate({endAngle:d.startAngle},d)
                    return function(t){// t ???
                        return arc(interpolate(t))// 利用弧度生成器arc处理饼图数据
                    }
                })

            // 5.添加标签
            arcs.append('text')
                .attr('transform',function(d){
                    let x = arc.centroid(d)[0]*3
                    let y = arc.centroid(d)[1]*3
                    return 'translate('+x+','+y+')'
                })
                .attr('text-anchor','middle')
                .text(d=>{
                    return d.data['0']+' '+d.data['1']
                })
            // 6.添加连线
            arcs.append('line')
                .attr('stroke',(d,i) => colors[i])
                .attr('x1',d => arc.centroid(d)[0] * 2)
                .attr('y1',d => arc.centroid(d)[1] * 2)
                .attr('x2',d => arc.centroid(d)[0] * 2.8)
                .attr('y2',d => arc.centroid(d)[1] * 2.8)
            
        },
        createSvg(){
            // 1.创建SVG
            const width = document.getElementById('pie').offsetWidth
            const height = document.getElementById('pie').offsetHeight
            // console.log(height,width)
            const edge = width > height ? height : width// 做成正方形
            const svg = d3.select('#pie')
                .append('svg')
                .attr('id','svg_pie_1')
                .attr('width', edge)
                .attr('height', edge)
                .style('background-color','#fff')
        },
        clearSvg(){
            console.log('piechart clearSvg executing~')
            d3.select('#svg_pie_1_chart').remove()
        },
        handleData(){
            // 对参数进行排序
            console.log('piechart handleData executing~')
            let dataArrOrder = this.yearPoetryNum
            console.log(this.yearPoetryNum,'1515321')
            dataArrOrder.sort((a,b) => b[1]-a[1])
            this.topTen = dataArrOrder.slice(0,10)
            
        }
    }
}
</script>

<template>
    <div class="pie" id="pie">

    </div>

</template>

<style>
.pie {
    /* flex: none; */
    width: 25%;
    height: 50%;
}
.pie_1_s{
    stroke: #000;
    stroke-width: 3;
}
</style>