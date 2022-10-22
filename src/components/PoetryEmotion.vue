<script>
    import * as d3 from 'd3'
    export default{
        mounted(){
            this.draw()
        },
        methods:{
            draw() {
                // 1.创建SVG
                const width = document.getElementById('emo').offsetWidth
                const height = document.getElementById('emo').offsetHeight
                // console.log('con',height,width)
                const svg = d3.select('#emo')
                    .append('svg')
                    .attr('width', width)
                    .attr('height', height)
                    .style('background-color','#fff')

        }
    }
}
</script>

<template>
    <div id="emo">诗词情感</div>
    
</template>

<style scoped>
    #emo{
        /* flex: none; */
    }
</style>