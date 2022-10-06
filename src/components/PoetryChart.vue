<script>
    import PoetryPieChart from './PoetryPieChart.vue'
    import PoetryLineChart from './PoetryLineChart.vue'
    import ConnectionChart from './ConnectionChart.vue'
    
    export default{
        components:{
            PoetryPieChart,
            PoetryLineChart,
            ConnectionChart
        }
    }
</script>

<template>
    <div class="chart-wrapper">
        <PoetryPieChart />
        <PoetryLineChart />
        <ConnectionChart />
    </div>
</template>

<style scoped>
    .chart-wrapper{
        display: flex;
        height: 60%;
    }
</style>