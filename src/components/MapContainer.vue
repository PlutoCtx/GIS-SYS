<template>
    <div class="box">
        <h3>高德地图</h3>
        <div>
            <span>城市：</span>
            <select></select>
            &nbsp;
            <select></select>
        </div>
    </div>
  <div id="container">
  </div>
</template>

<script setup>
import AMapLoader from '@amap/amap-jsapi-loader';
import { shallowRef, onMounted } from 'vue'
import {$get} from '../utils/request.js'
// 定义一个map对象
const map = shallowRef(null);
// 初始化地图方法
const initMap = ()=>{
    AMapLoader.load({
        key:"d09336596eb4b793f08c9805003af069",             // 申请好的Web端开发者Key，首次调用 load 时必填
        version:"2.0",      // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins:[''],       // 需要使用的的插件列表，如比例尺'AMap.Scale'等
    }).then((AMap)=>{
        map.value = new AMap.Map("container",{  //设置地图容器id
            viewMode:"3D",    //是否为3D地图模式
            zoom:5,           //初始化地图级别
            center:[105.602725,37.076636], //初始化地图中心点位置
        });
    }).catch(e=>{
        console.log(e);
    })
}

// 获取城市
const loadCities = async ()=>{
    let ret = await $get('/api/aj/getcitycode')
    console.log(ret);
}

// 页面挂载完成
onMounted(()=>{
    // 获取城市
    loadCities()
    // 调用initMap()方法
    initMap()
})
</script>

<style scoped>
.box{
    border: 1px solid #ccc;
    padding: 5px;
    margin: 5px;
    color: #333;
}
#container{
    padding:0px;
    margin: 5px 5px 0 5px;
    box-sizing: border-box;
    height: 700px;
}
</style>