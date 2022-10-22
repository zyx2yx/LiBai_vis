<script>
import * as d3 from 'd3'

import PoetryPieChart from './PoetryPieChart.vue'
import PoetryLineChart from './PoetryLineChart.vue'
import ConnectionChart from './ConnectionChart.vue'

export default {
    components: {
        PoetryPieChart,
        PoetryLineChart,
        ConnectionChart,
    },
    data() {
        return {
            yearPoetryNum: [],
            selectedYear:705
        }
    },
    mounted() {
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
                    if (key === '') {
                        // arr['未知'] = yearPoetryCount[key]
                        arr.push('未知', yearPoetryCount[key])
                    }
                    else {
                        // arr[key] = yearPoetryCount[key]
                        arr.push(key, yearPoetryCount[key])
                    }
                    dataArr.push(arr)
                })
                // console.log(dataArr)
                // 对dataArr按照数量降序排列
                // dataArr.sort((a, b) => b[1] - a[1])
                // 取出前10
                // this.yearPoetryNum = dataArr.slice(0, 10)
                this.yearPoetryNum = dataArr
            })
            .catch(reason => { console.log('error', reason) })

    },
    methods:{
        changeYear(curYear){
            this.selectedYear = curYear
        },
    }
}
</script>

<template>
    <div class="chart-wrapper">
        <PoetryPieChart :yearPoetryNum="yearPoetryNum" @change-year="changeYear"/>
        <ConnectionChart v-bind:selectedYear="selectedYear"/>
        <PoetryLineChart :yearPoetryNum="yearPoetryNum" />
    </div>
</template>

<style scoped>
.chart-wrapper {
    
    display: flex;
    flex-wrap: wrap;
    height: 75%;
    width: 100%;
}
</style>