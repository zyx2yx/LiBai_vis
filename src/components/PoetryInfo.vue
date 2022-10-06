<script>
    import PoetryList from './PoetryList.vue'
    import PoetryContent from './PoetryContent.vue'
    import WordCloud from './WordCloud.vue'

    export default{
        components:{
            PoetryList,
            PoetryContent,
            WordCloud
        }
    }
</script>

<template>
    <div class="poetry-info">
        <PoetryList/>
        <PoetryContent/>
        <WordCloud/>
    </div>
</template>

<style scoped>
    .poetry-info{
        height: 600px;
        display: flex;
        flex-direction: column;
        margin: 1rem 0;
    }
</style>