<script>
    import * as d3 from 'd3'
    import PersonLink from './PersonLink.vue'
    import PositionChart from './PositionChart.vue'
    export default{
        components:{
            PersonLink,
            PositionChart,
        }
        ,
        props:{
            selectedYear:{
                type:[String,Number],
                required:true,
            }
        },
        data(){
            return {
                currentDate:{},
            }
        },
        mounted(){
            // 获取数据并更新state
            this.getLifeExpByYear()
                .then(async data => {
                    if(data === 0) this.currentDate={} 
                    else this.currentDate = data
                })
                .catch(e => {
                    console.log('getLifeExpByYear:',e)
                })
        },
        beforeUpdate(){
            // 获取数据并更新state
            // 若在beforeUpdate中更新状态，组件会陷入死循环的更新状态
            
            // this.getLifeExpByYear()
            //     .then(async data => {
            //         this.currentDate = data
            //     })
            //     .catch(e => {
            //         console.log('getLifeExpByYear:',e)
            //     })
            console.log('connectionChart更新')
        },
        watch:{
            selectedYear(){
                this.getLifeExpByYear()
                    .then(async data => {
                        if(data === 0) this.currentDate={} 
                        else this.currentDate = data
                    })
                    .catch(e => {
                        console.log('getLifeExpByYear:',e)
                    })
            }
        }
        ,
        methods:{
            async getLifeExpByYear(){
                if(this.selectedYear === 0){
                    return Promise.resolve(0)
                }else{
                    const year =  parseInt(this.selectedYear)
                    
                    // 根据year在libai_life_experience中获取year的数据
                    try {
                        const data = await d3.json('src/assets/libai/libai_life_experience.json')
                        const currentDate = data[year]
                        console.log(data)
                        return Promise.resolve(currentDate)
                    } catch (error) {
                        return Promise.reject(error)
                    }

                }
            },
        },
}
</script>

<template>
    <div class="connect">
        <PersonLink :currentDate="currentDate"/>
        <PositionChart :currentDate="currentDate"/>
    </div>
</template>

<style scoped>
    .connect{
        /* flex: none; */
        width: 75%;
        height: 50%;
        display: flex;
        background-color: #fff;
    }
</style>