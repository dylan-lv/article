1、安装依赖

```bash
 npm i leaflet@1.7.1 @supermap/iclient-leaflet@10.1.0 -S
```



2、引入

```js
// main.js
import 'leaflet/dist/leaflet.css';
import '@supermap/iclient-leaflet';
```



3、初始化地图

Map.vue

```vue
<template>
  <div id="map" ref="map" />
</template>

<script>
import L from 'leaflet';
export default {
  data() {
    return {
      map: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      const res = [
        0.0012576413977673302, 
        7.266547736220494E-4,
        3.633273868110247E-4, 
        1.8166369340551236E-4, 
        9.083184670275618E-5, 
        4.5415923351378204E-5, 
        2.2707961675688977E-5, 
        1.1353980837844615E-5, 
        5.676990418922308E-6, 
        2.8384952094612792E-6, 
        1.4192476047306396E-6
      ];
      this.map = L.map('map', {
        crs: L.Proj.CRS('EPSG:4326', {
          origin: L.point(116.20934993335362, 39.83089881098218),
          resolutions: res
        }),
        center: [39.73, 116.33],
        maxZoom: 17,
        zoom: 1,
        zoomControl: false,
        attributionControl: false
      });

      var host = window.isLocal ? window.server : 'https://iserver.supermap.io';
      var url = host + '/iserver/services/map-world/rest/maps/World';
      if (process.env.NODE_ENV === 'production') {
        L.supermap.wmtsLayer(`http://172.25.117.10:8081/geoesb/proxy/db28d8d25a5b4ef2b6b4e2c44bed0e6f/1eb57180e8344ae9b12ac4f9a361363b`,
          {
            layer: 'DX_DLG_2020',
            style: 'default',
            tilematrixSet: 'Custom_DX_DLG_2020',
            format: 'image/png',
            requestEncoding: 'REST',
            attribution: ''
          }
        ).addTo(this.map);
      } else {
        L.supermap.tiledMapLayer(url).addTo(this.map);
      }
    }
  }
};
</script>
```

