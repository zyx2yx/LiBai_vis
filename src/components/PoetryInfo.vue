<script>
    import PoetryList from './PoetryList.vue'
    import PoetryContent from './PoetryContent.vue'
    import PoetryEmotion from './PoetryEmotion.vue'
    import WordCloud from './WordCloud.vue'

    export default{
        components:{
            PoetryList,
            PoetryContent,
            PoetryEmotion,
            WordCloud
        }
    }
</script>

<template>
    <div class="poetry-info">
        <PoetryList/>
        <PoetryContent/>
        <PoetryEmotion/>
        <WordCloud/>
    </div>
</template>

<style scoped>
    .poetry-info{
        height: 100%;
        display: flex;
        flex-direction: column;
        margin: 1rem 0;
    }
</style>