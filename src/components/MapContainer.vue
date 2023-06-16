<template>
    <div class="box">
        <h3>高德地图</h3>
        <div>
            <span>城市：</span>
            <select v-model="currentCity_id">
                <option value="0">选择城市</option>
                <option v-for="(item, index) in cities" :key="index" :value="item.code">{{item.name}}</option>
            </select>
            &nbsp;
            <select v-model="area_id">
                <option value="0">选择区县</option>
                <option v-for="(item, index) in areas" :key="index" :value="item.area_id">{{item.area_name}}</option>
            </select>
        </div>
    </div>
  <div id="container">
  </div>
</template>

<script setup>
import AMapLoader from '@amap/amap-jsapi-loader';
import { shallowRef, onMounted, ref, watch } from 'vue'
import {$get} from '../utils/request.js'

// 备份AMap对象
let currentAMap = null
// 定义一个map对象
const map = shallowRef(null);
// 热门城市
let cities = ref([])
// 当前选择的城市编号
let currentCity_id = ref('0')
// 定义区县数组
let areas = ref([])
// 当前选择的区县编号
let area_id = ref('0')
// 初始化地图方法
const initMap = ()=>{
    AMapLoader.load({
        key:"d09336596eb4b793f08c9805003af069",             // 申请好的Web端开发者Key，首次调用 load 时必填
        version:"2.0",      // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins:[''],       // 需要使用的的插件列表，如比例尺'AMap.Scale'等
    }).then((AMap)=>{
        currentAMap = AMap
        map.value = new AMap.Map("container",{  //设置地图容器id
            viewMode:"3D",    //是否为3D地图模式
            pitch: 45,
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
    cities.value = ret.result.hotcity
}

// 获取城市的区县
const loadAreas = async (citycode)=>{
    let {result} = await $get('/api/aj/get_area', {citycode})
    let arr = Object.keys(result).map(k =>{
        return {
            area_id: k,
            area_name: result[k]
        }
    })
    // console.log(arr)
    areas.value = arr
}

// 获取商城信息
const loadShops = async ()=>{
    // 获取市 + 区县的信息
    let address = cities.value.find(c=>c.code==currentCity_id.value).name
        + areas.value.find(a=>a.area_id==area_id.value).area_name

    // 获取该地址的坐标
    let {geocodes} = await $get('https://restapi.amap.com/v3/geocode/geo', {
        key: 'a6883936d454c262eaf33d9e0064c98b',
        address
    })

    // 将坐标转换为数组格式
    let position = geocodes[0].location.split(',')
    // 设置地图中心点和缩放级别
    map.value.setZoomAndCenter(12, position);
    // 获取商城的信息
    let {shop_data} = await $get('/api/at/shop', {
        currentCity_id: currentCity_id.value,
        area_id: area_id.value
    })
    // 清楚地图上的所有覆盖物
    map.value.clearMap();
    // 循环遍历所有的商城
    shop_data.forEach(s=>{
        // 创建marker
        const marker = new currentAMap.Marker({
            // 位置
            position: [s.map_longitude, s.map_latitude],
            // 标题
            title: s.shop_name
        });
        console.log(s.map_longitude, s.map_latitude, s.shop_name)
        // 将marker添加到地图
        map.value.add(marker)
        // 给每个marker注册点击事件
        map.on('click', function (e){
            console.log(e.target.getPosition());
            // 创建并打开一个信息框
            var infoWindow = new currentAMap.InfoWindow({
                // 信息框内容
                content: "Hello World!",  //传入 dom 对象，或者 html 字符串
                // 位置偏移量
                offset: new AMap.Pixel(0, -30)
            });
            infoWindow.open(map.value, e.target.getPosition());
        })
    })
}

// 监听城市编号
watch(currentCity_id, (nval)=>{
    // 加载区县信息
    loadAreas(nval)
    area_id.value = '0'
})

// 监听区县编号
watch(area_id, (nval)=>{
    if (nval !== '0'){
        // 加载商城信息
        loadShops()
    }
})

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