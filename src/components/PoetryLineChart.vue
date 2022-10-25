<script>
import * as d3 from 'd3'

export default {
    props: {
        yearPoetryNum: {
            type: Array,
            required: true
        },
        selectedYear:{
            type:Number,
            required:true
        }
    },
    emits:['changeYear'],
    data() {
        return {
            lineChartData: []
        }
    },
    watch:{
        yearPoetryNum(){
            this.handleData()
            console.log('line watch yearPoetryNum 执行...');
        }
    },
    mounted() {
        this.createSvg()
        this.handleData()
        // this.drawLineChart()
        console.log('line mounted 执行...');
    },
    beforeUpdate() {
        this.clearSvg()
        this.drawLineChart()
        console.log('line beforeUpdate 执行...');
    },
    methods: {
        
        createSvg() {
            // 1.创建画布
            const width = document.getElementById('line').offsetWidth
            const height = document.getElementById('line').offsetHeight

            const svg = d3.select('#line')
                .append('svg')
                .attr('width', width)
                .attr('height', height)
                .attr('id', 'svg_line')
                .style('background-color', '#fff')
            // 创建图
        },
        drawLineChart() {
            const _this = this
            // console.log(this.lineChartData)
            const width = d3.select('#svg_line').attr('width')
            const height = d3.select('#svg_line').attr('height')
            const margin = 30

            const chart = d3.select('#svg_line').append('g').attr('id','svg_line_chart')
            // 2.创建比例尺
            const scaleX = d3.scaleBand()
                .domain(this.lineChartData.map(e => e.year))
                .range([0, width - margin * 2])
            const scaleY = d3.scaleLinear()
                .domain([0, d3.max(this.lineChartData.map(d => d.count))])
                .range([height - 2 * margin, 0])
            // 画坐标轴
            const axisX = d3.axisBottom(scaleX).tickFormat((d, i) => {
                // console.log(d) // 701~762
                // console.log(i) // index
                if (parseInt(d) % 5 === 0 || i === 0 || i === 61)
                    return d
                else
                    return ''
            })
            const axisY = d3.axisLeft(scaleY)
            chart.append('g').attr('transform', `translate(${margin},${height - margin})`).attr('class', 'axisX').call(axisX)
            chart.append('g').attr('transform', `translate(${margin},${margin})`).call(axisY)
            // 创建线生成器 生成点的坐标，由数据的结构决定
            const line = d3.line().x(d => scaleX(d.year)).y(d => scaleY(d.count))//.curve(d3.curveCardinal)
            const lines = chart.append('g')
                .append('path')
                .attr('class', 'lineG1')
                .attr('transform', `translate(${1.25 * margin},${margin})`)
                .attr('d', line(this.lineChartData))
                .attr('stroke', 'green')
                .attr('stroke-width', 1)
                .attr('fill', 'none')
            // 生成点
            const pointsGroup = chart.append('g')
                .attr('class', 'circleG1')
                .selectAll()
                .data(this.lineChartData)
                .join('g')
            pointsGroup
                .append('circle')
                .attr('r', 2)
                .attr('cx', d => scaleX(d.year) + 1.25 * margin)
                .attr('cy', d => scaleY(d.count) + margin)
                .attr('fill', 'green')
            // 标签
            chart.append('text').attr('x', width / 2).attr('y', height - 8).text('年份')
            chart.append('text').attr('x', 0).attr('y', margin - 10).text('数量')
            // 设置一个简单动画 （与饼图和折线图的交互冲突了，影响视觉效果）
            // d3.select('.lineG1')
            //     .style('stroke-dasharray', function () {// 设置虚线长度为本身长度
            //         return d3.select(this).node().getTotalLength()
            //     })
            //     .style('stroke-dashoffset', function () {// 设置虚线偏移量为本身长度
            //         return d3.select(this).node().getTotalLength()
            //     })
            //     .transition()
            //     .duration(5000)
            //     .ease(d3.easeLinear)
            //     .style('stroke-dashoffset', 0)
            // d3.selectAll('.circleG1>g>circle')
            //     .attr('r', 0)
            //     // .attr('stroke-width',1)
            //     .transition()
            //     .duration(0)
            //     .delay((d, i) => i * 5000 / 60)
            //     .ease(d3.easeLinear)
            //     .attr('r', 2)
            // .attr('stroke-width',1)

            // 文本框
            // chart.append('p').attr('width',40).attr('height',20).attr('x',0).attr('y',0).text('1212121') //.text(`${d.year}年${d.count}首`)
            // 添加坐标竖轴，垂直于横轴的各个刻度
            let selectedRect = null
            pointsGroup.append('rect')
                .attr('width', scaleX.bandwidth())
                .attr('height', height - 2 * margin)
                .attr('x', (d, i) => margin + scaleX.bandwidth() * i)
                .attr('y', margin)
                .attr('fill-opacity',0)
                .classed('select-rect',function(d){
                    // console.log(_this.selectedYear,d.year)
                    if(_this.selectedYear === d.year) {
                        selectedRect = d3.select(this)
                        return true
                    }
                    else {return false}
                })
                .on('mouseover',function(e,d){
                    // 给选定的rect增加class
                    d3.select(this).classed('rect-hover',true)
                })
                .on('mouseout',function(){
                    // 目标rect删去class
                    d3.select(this).classed('rect-hover',false)
                })
                .on('click',function(e,d){
                    // d3.selectAll('.select-rect').classed('select-rect',false)
                    selectedRect && selectedRect.classed('select-rect',false)
                    selectedRect = d3.select(this).classed('select-rect',true)
                    
                    // console.log(d);
                    _this.$emit('changeYear',d.year)
                })
                .on('mousemove',function(e,d){

                })
        },
        clearSvg(){
            d3.select('#svg_line_chart').remove()
        },
        handleData() {
            // 清理无用数据，将[[],[],...]，处理成[{},{}...]备用
            const lineData = []
            for (let year = 701; year < 763; year++) {
                let obj = {}
                obj.year = year
                obj.count = 0
                lineData.push(obj)
            }
            // console.log(lineData)
            this.yearPoetryNum.forEach((el) => {
                if (!isNaN(parseFloat(el[0])) && isFinite(el[0])) {// 若第一个数组元素是具体年份
                    lineData[parseInt(el[0]) - 701].count = el[1]
                }
            })
            this.lineChartData = lineData
        },
    }

}
</script>

<template>
    <div class="line" id="line">

    </div>

</template>

<style scoped>
.line {
    flex: none;
    width: 100%;
    height: 50%;
}
:deep(.rect-hover){
    fill:'grey';
    fill-opacity: 0.1;
}
:deep(.select-rect){
    fill: orange;
    fill-opacity: 0.3;
    /* stroke-opacity: 0.2; */
}
</style>