/* eslint-disable no-console */
<template>
  <div id="app">
    <div class="row no-gutters">
  <!-- 選擇地區 -->
  <div class="toolbox col-sm-3 p-2 bg-white">
    <div class="form-group d-flex">
      <label for="city" class="col-form-label mr-2 text-right">縣市</label>
      <div class="flex-fill">
        <!-- v-model依照選取改變顯示選項 -->
        <select id="city" class="form-control" v-model="select.city">
        <!-- 製作下拉選單 -->
        <!-- 不能直接寫option.value="{{ cityName.districts.name }}" -->
          <option v-bind:value="city.name" v-bind:key="city.name" v-for="city in cityName">
            <!-- v-for做其他縣市 -->
            {{city.name}}
          </option>
        </select>
      </div>
    </div>
    <div class="form-group d-flex">
      <label for="dist" class="col-form-label mr-2 text-right">地區</label>
      <div class="flex-fill">
        <select id="dist" class="form-control" v-model="select.dist">
        <!-- 製作下拉選單 -->
        <!-- v-bind:簡寫為: -->
        <option :value="dist.name" :key="dist.name"
        v-for="dist in cityName.find((city) => city.name === select.city).districts">
          {{dist.name}}
        </option>
        </select>
      </div>
    </div>
  </div>

  <!-- 顯示地圖和 UBike 站點 -->
  <div class="col-sm-9">
    <div id="map"></div>
  </div>
</div>
  </div>
</template>

<script>
import L from 'leaflet'; // OSM不像是google map可以直接標記點位，需要其他套件
import cityName from './assets/cityName.json';

export default {
  name: 'App',
  data: () => ({
    cityName,
    select: {
      city: '臺北市',
      dist: '中正區',
    },
    ubikes: [],
    OSMap: [],
  }),
  computed: { // 重新計算ubikes，避免呈現過多markerLayer
    youbikes() {
      return this.ubikes.filter((bike) => bike.sarea === this.select.dist);
    },
  },
  watch: { // 監聽youbikes是否重新計算過，重新計算時執行addMarkers()方法
    youbikes() {
      this.updateMap();
    },
  },
  methods: {
    updateMap() {
      // remove markers
      this.OSMap.eachLayer((layer) => {
        // 移除先前查詢的標記，避免重複顯示
        if (layer instanceof L.Marker) {
          this.OSMap.removeLayer(layer);
        }
      });

      // add markers
      this.youbikes.forEach((bike) => {
        L.marker([bike.lat, bike.lng])
          .bindPopup(
            `<p><strong style="font-size: 20px;">${bike.sna}</strong></p>
            <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong><br>
            可停空位剩餘: ${bike.bemp}<br>
            <small>最後更新時間: ${bike.mday}</small>`,
          )
          .addTo(this.OSMap); // 新增標記到地圖
      });

      // move to new center
      this.cityName[0].districts.find((dist) => {
        if (dist.name === this.select.dist) {
          // this.OSMap.panTo(new L.LatLng(dist.latitude, dist.longitude)); // 直接平移
          this.OSMap.flyTo(new L.LatLng(dist.latitude, dist.longitude, 14)); // 飛越效果，數字為過程中縮放級數
        }
        return dist.name === this.select.dist;
      });
    },
  },
  created() {
    const url = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
    this.$http.get(url).then((response) => {
      console.log(response.data);
      this.ubikes = Object.keys(response.data.retVal).map((key) => response.data.retVal[key]);
    });
  },
  mounted() {
    // initalize
    this.OSMap = L.map('map', {
      center: [25.041956, 121.508791],
      zoom: 18,
    });

    // add tile to map
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      maxZoom: 18,
    }).addTo(this.OSMap);

    this.setView();
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
  position: relative;
}
</style>
