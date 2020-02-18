<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
              <div class="flex-fill">
                <select id="cityName" class="form-control"
                  v-model="select.city" @change="removeMapMarker(); uodateMap()">
                  <option value="">-- Select One --</option>
                  <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                    {{ c.CityName }}
                  </option>
                </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="area" class="mr-2 col-form-label text-right">地區</label>
              <div class="flex-fill">
                <select id="area" class="form-control">
                  <option value="">-- Select One --</option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">請先選擇區域查看（綠色表示還有口罩）</p>
          </div>
          <ul class="list-group">
            <template>
              <a class="list-group-item text-left">
                <h3>藥局名稱</h3>
                <p class="mb-0">
                  成人口罩：1 | 兒童口罩：2
                </p>
                <p class="mb-0">
                  地址：<a
                    href="https://www.google.com.tw/maps/place/..."
                    target="_blank"
                    title="Google Map"
                  >
                    地址</a
                  >
                </p>
              </a>
            </template>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";
import L from 'leaflet';
import cityName from './assets/api/cityName.json';

let omgMap = {};

export default {
  name: 'app',
  data: () => ({
    data: [],
    cityName,
    select: {
      city: '臺北市',
    },
  }),
  components: {},
  methods: {
    uodateMap() {
      const pharmacies = this.data.filter((pharmacy) => (
        pharmacy.properties.county === this.select.city));
      console.log(pharmacies);
      // L.marker([25.03, 121.55]).addTo(omgMap);

      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;
        L.marker([
          geometry.coordinates[1],
          geometry.coordinates[0],
        ]).addTo(omgMap).bindPopup(`<strong>${properties.name}<strong><br>
        口罩剩餘：<strong>成人 - ${properties.mask_adult ? `${properties.mask_adult} 個` : '未取得資料'} / 兒童 - 
        ${properties.mask_child ? `${properties.mask_child} 個` : '未取得資料'} <br>
        電話：${properties}
        `);
        console.log(properties);
      });
    },
    // 清除
    removeMapMarker() {
      omgMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          omgMap.removeLayer(layer);
        }
      });
    },
  },
  mounted() {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
    this.$http
      .get(url)
      .then((res) => {
        this.data = res.data.features;
        this.uodateMap();
      })
      .catch((err) => {
        console.log(err);
      });
    // initialize the map on the "omgMap" div with a given center and zoom
    omgMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 16,
    });
    // 導入圖專 TileLayer
    // https://leafletjs.com/reference-1.6.0.html#map-example
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png?{foo}', {
      foo: 'bar',
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>',
      maxZoom: 18,
    }).addTo(omgMap);
    // L.marker([25.03, 121.55]).addTo(omgMap);
  },
};
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
</style>
