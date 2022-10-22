<script>
    import * as d3 from 'd3'
    export default{
        mounted(){
            // this.draw()
        },
        methods:{
            draw() {
                // 1.创建SVG
                const width = document.getElementById('pos').offsetWidth
                const height = document.getElementById('pos').offsetHeight
                // console.log('con',height,width)
                const svg = d3.select('#pos')
                    .append('svg')
                    .attr('width', width)
                    .attr('height', height)
                    .style('background-color','#fff')

        }
    }
}
</script>

<template>
    <div id="pos"></div>
    
</template>

<style scoped>
    #pos{
        /* flex: none; */
        width: 30%;
        height: 100%;
    }
</style>