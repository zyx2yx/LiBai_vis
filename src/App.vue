<script setup>
import PoetryChart from './components/PoetryChart.vue'
import PoetryInfo from './components/PoetryInfo.vue'
</script>

<template>
  <div class="main">
    <div class="main-content">
      <div class="head-warpper">
        <img alt="Vue logo" class="logo" src="./assets/jpg/libai.webp" width="150" height="150" />
        <div class="info">
          <div class="title">
            <div>本名</div>
            <div>别名</div>
            <div>字</div>
            <div>号</div>
            <div>主要作品</div>
          </div>
          <div class="content">
            <div>李白（701年—762年12月）</div>
            <div>李十二、李翰林、李供奉、李拾遗、诗仙</div>
            <div>太白</div>
            <div>青莲居士</div>
            <div>静夜思、蜀道难、明堂赋、梦游天姥吟留别、行路难、将进酒、早发白帝城、望庐山瀑布</div>
          </div>
        </div>
      </div>
      <PoetryChart/>
    </div>
    <div class="poetry-content">
      <PoetryInfo />
    </div>
  </div>
</template>

<style scoped>
.main{
  width: 100%;
  height: 100%;
  display: flex;
}
.main-content{
  width: 70%;
  height: 100%;
}
.poetry-content{
  width: 30%;
  height: 100%;
}
header {
  line-height: 1.5;
  margin: 1rem 0;
  height: 100%;
}

.head-warpper{
  display: flex;
  height: 25%;
  width: 100%;
}
.logo {
  display: block;
}

.info {
  margin-left: 1rem;
  display: flex;
}

.title {
  width: 6rem;
  /* margin-right: 1rem; */
}

.title>div {
  font-weight: bold;
}
</style>
