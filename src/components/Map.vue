<template>
  <div class="ss">
    <button @click="active">加载地图</button>
    <div id="map" class="map"></div>
  </div>
</template>

<script>
import ol from "openlayers"
import demojson from "../../static/toursim.json"

export default {
  name: "Map",
  data() {
    return {
      map: null,
      pointCount: 0,
      line: [],
      lineCount: 0
    }
  },
  methods:{
    active: function() {
      this.map = new ol.Map({
        target: "map",
        layers: [
          new ol.layer.Tile({
            source: new ol.source.XYZ({
              url: '../../static/googlemaps/roadmap/{z}/{x}/{y}.png'
            })
          })
        ],
        view: new ol.View({
          center: ol.proj.fromLonLat([116.28, 39.54]),
          zoom: 4,
          minZoom: 4,
          maxZoom: 10
        })
      })
      this.lineCount = demojson.moveLines.length
      for (var i = 0; i < this.lineCount; i++) {
        var lineFeature = new ol.Feature(
          new ol.geom.LineString([
            new ol.proj.fromLonLat(demojson.moveLines[i].coords[0]),
            new ol.proj.fromLonLat(demojson.moveLines[i].coords[1])
          ])
        )
        this.line[i] = lineFeature
      }
      var lineSource = new ol.source.Vector({
        features: this.line
      })
      var lineLayer = new ol.layer.Vector({
        source: lineSource,
        style: new ol.style.Style({
          stroke: new ol.style.Stroke({
            color: "black",
            width: 0.5,
            opacity: 0.2
          })
        })
      })
      this.pointCount = demojson.citys.length
      var features = []
      for (var i = 0; i < this.pointCount; i++) {
        features[i] = new ol.Feature(
          new ol.geom.Point(
            new ol.proj.fromLonLat([
              demojson.citys[i].value[0],
              demojson.citys[i].value[1]
            ])
          )
        )
      }
      var pointSource = new ol.source.Vector({
        features: features
      })
      var clusterSource = new ol.source.Cluster({
        distance: 20,
        source: pointSource
      })
      var clusters = new ol.layer.Vector({
        source: clusterSource,
        style: new ol.style.Style({
                  image: new ol.style.Circle({
                    radius: 3,
                    fill: new ol.style.Fill({
                            color: "black"
                          })
                    })
                })
      })
      this.map.addLayer(lineLayer)
      this.map.addLayer(clusters)
    }
  }
}
</script>
<style>
.map {
  width: 755px;
  height: 500px;
  margin-left: 600px;
}
</style>
