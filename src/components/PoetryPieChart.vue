<script>
import * as d3 from 'd3'
// 处理数据，获取每年的诗词数量总和
let poetryList;
// let yearPoetryNum=[['init',75],['test',25]];
// d3.json('src/assets/libai/libai_merge_emotion.json',res => {
//     poetryList = res
//     console.log(poetryList)

// })

// poetryList = getPoetryList()
export default {
    data() {
        return {
            yearPoetryNum: []
        }
    },
    mounted() {
        const _this = this
        const p = new Promise(async function (resolve, reject) {
            try {
                const res = await d3.json('src/assets/libai/libai_merge_emotion.json')
                resolve(res)
            } catch (error) {
                reject(error)
            }
        })
            .then(value => {
                // 数据处理
                let yearPoetryCount = {} // 按年份统计诗词数量，属性值为年份
                let poetrySum = 0 // 诗词总量
                let dataArr = [] // 将yearPoetryCount转化为数组
                value.forEach(element => {
                    // 将element中的time属性值作为属性名添加到对象中
                    if (yearPoetryCount.hasOwnProperty(element.time)) {
                        yearPoetryCount[element.time]++
                    } else {
                        yearPoetryCount[element.time] = 1
                    }
                    poetrySum++
                });
                // 遍历yearPoetryCount每个属性
                Object.keys(yearPoetryCount).forEach(key => {
                    let arr = []
                    if (yearPoetryCount[key] > 10) {// 剔除诗词数量为1的年份
                        if (key === '') {
                            // arr['未知'] = yearPoetryCount[key]
                            arr.push('未知', yearPoetryCount[key])
                        }
                        else {
                            // arr[key] = yearPoetryCount[key]
                            arr.push(key, yearPoetryCount[key])
                        }
                        dataArr.push(arr)
                    }
                })
                // console.log(dataArr)
                // 对dataArr按照数量降序排列
                dataArr.sort((a,b) => b[1]-a[1])
                // 取出前10
                this.yearPoetryNum = dataArr.slice(0,10)
                _this.drawPieChart()
            })
            .catch(reason => { console.log('error', reason) })

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
            // 0.创建饼布局 分别设置排序方式 绑定一个计算扇形区值的函数 处理数据为饼图数据
            // sort(null) 创建布局时，默认按照从小到大，设置为null则按照给定数组顺序
            const pie = d3.pie().sort(null).value(function (d) {
                return d[1]// ?
            })
            const pieData = pie(this.yearPoetryNum)
            console.log(pieData)

            // 1.创建SVG
            const width = document.getElementById('pie').offsetWidth
            const height = document.getElementById('pie').offsetHeight
            // console.log(height,width)
            const svg = d3.select('#pie')
                .append('svg')
                .attr('width', width)
                .attr('height', height)
                .style('background-color','#fff')

            // 2.创建新的弧度生成器，设置内外部半径
            const innerRadius = 0
            const outerRadius = width / 4
            const arc = d3.arc().outerRadius(outerRadius).innerRadius(innerRadius)

            // 3.创建分组并绑定数据
            const arcs = svg.selectAll('g')// arcs是g标签选择集
                .data(pieData)
                .enter()
                .append('g')
                .attr('transform', 'translate(' + width / 2 + ',' + height / 2 + ')')
            
            // 4.调用内置颜色序列 添加交互 添加动画
            const colors = d3.schemeCategory10
            arcs.append('path')// 'path'标签是？ 在g标签中创建path标签
                .attr('fill', function (data, index) {
                    return colors[index]
                })
                .attr('d', data => arc(data))// 设置path标签'd'属性,值为弧度生成器arc处理后的数据
                .on('mouseover',this.arcTween(outerRadius,20))
                .on('mouseout',this.arcTween(outerRadius,0))
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
                
        }
    }
}
</script>

<template>
    <div class="pie" id="pie">

    </div>

</template>

<style scoped>
.pie {
    width: 30%;
    height: 100%;
}
</style>