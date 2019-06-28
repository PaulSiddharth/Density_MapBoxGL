<template>
  <div>
    <h1> Visualize your data more precisely</h1>
    <p>Choose csv file to upload <input type="file" class="dropbox" @change="processFile($event)"  width="50%"></p>
    
    <div id="map" class="displayMapSize" >
    </div>
    <div>
      <h3>Different types of filter for better visualization</h3>
    </div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
export default {  
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      maps : null
    }
  },
  methods:{
      csvToJson(csv){
        var lines = csv.split('\n')
        var results = []
        var geoJsonArray = []
        var headers = lines[0].split(',')
        lines.map( function(line, indexLine){
          if(indexLine < 1 )
            return
          var obj={}
          var geoObj={
              "type": "Feature",
              "properties": {},
              "geometry": {
                "type": "Point"
              }
            }
          // var geoObj = {
          //   "type": "Feature",
          //   "properties": {},
          //   "geometry": {
          //     "type": "LineString"
          //   }
          // }

          var currentLine = line.split(',')
          // if(currentLine[17] !=="NULL")
          // { 
          //   headers.map(function(header, indexheader){
          //     obj[header] = currentLine[indexheader]
          //     if(header=='from_lat' )
          //       geoObj.geometry["coordinates"] = [[parseFloat(currentLine[indexheader+1]),parseFloat(currentLine[indexheader])],
          //                                         [parseFloat(currentLine[indexheader+3]),parseFloat(currentLine[indexheader+2])]]
          //   })
          //   results.push(obj);
          //   geoJsonArray.push(geoObj);
          // }
          headers.map(function(header, indexheader){
              obj[header] = currentLine[indexheader]
              if(header=='Car_Cancellation' || header=='package_id' || header=='travel_type_id')
                geoObj.properties[header] = parseInt(currentLine[indexheader])
              if(header=='from_lat' )
                geoObj.geometry["coordinates"] = [parseFloat(currentLine[indexheader+1]),parseFloat(currentLine[indexheader])]
              if(header == 'from_date'){
                var date = new Date(currentLine[indexheader]);
                geoObj.properties['day']= date.getDay()
                geoObj.properties['month']= date.getMonth()
                geoObj.properties['hours']= date.getHours()
              }
            })
            results.push(obj);
            geoJsonArray.push(geoObj);
        })
        results.pop()
        geoJsonArray.pop()
        this.featureArray = geoJsonArray;
        this.featureArray_sunday = geoJsonArray.filter(a => a.properties.day==0)
        this.featureArray_monday = geoJsonArray.filter(a => a.properties.day==1)
        this.featureArray_tuesday = geoJsonArray.filter(a => a.properties.day==2)
        this.featureArray_wednesday = geoJsonArray.filter(a => a.properties.day==3)
        this.featureArray_thrusday = geoJsonArray.filter(a => a.properties.day==4)
        this.featureArray_friday = geoJsonArray.filter(a => a.properties.day==5)
        this.featureArray_saturday = geoJsonArray.filter(a => a.properties.day==6)
        debugger
        console.log(geoJsonArray)
        this.createMap()
        // console.log(results);
      },
      processFile(event) {
        this.uploadFile = event.target.files[0];
        if(this.uploadFile.name.split('.')[1] != 'csv'){
          console.log('Please upload a csv file')
          return
        }
        //load csv
        if(window.FileReader){
          var that = this;
          var reader = new FileReader();
          reader.readAsText(this.uploadFile);
          reader.onload = function(event){
            var csv = event.target.result
            that.csvToJson(csv)
          }
        }
      },
      createPopup(e){
        debugger
      },
      
      createMap () {
      mapboxgl.accessToken = 'pk.eyJ1Ijoic2lkZGhhcnRoYTk0IiwiYSI6ImNqdzh2NDNjbTJ0Ym4zeXBnc3R6dTRrNDgifQ.4fXJ1TzgULyVy2w7ndZW-Q'

      // init the map
      this.maps = new mapboxgl.Map({
        container: map,
        style: 'mapbox://styles/mapbox/dark-v9',
        minzoom: 1,
        center: [77.63625469,12.98246088],
        zoom: 12
      })
      // filters for classifying earthquakes into five categories based on magnitude
      var cancel1 = ["==", ["get", "Car_Cancellation"], 0];
      var cancel2 = ["==", ["get", "Car_Cancellation"], 1];
      var sun = ["==", ["get", "day"], 0];
      var mon = ["==", ["get", "day"], 1];
      var tue = ["==", ["get", "day"], 2];
      var wed = ["==", ["get", "day"], 3];
      var thru = ["==", ["get", "day"], 4];
      var fri = ["==", ["get", "day"], 5];
      var sat = ["==", ["get", "day"], 6];
      
      // colors to use for the categories
      var colors = ['#fed976', '#feb24c'];
 
      var that = this
      this.maps.on('load', function(){
        that.maps.addSource('pointsource',{
        type:'geojson',
        data:{
          "type": "FeatureCollection",
          "features" : that.featureArray_monday
          // "features": [
          //   {
          //     "type": "Feature",
          //     "properties": {},
          //     "geometry": {
          //       "type": "LineString",
          //       "coordinates": [
          //         [
          //           77.50030517578125,
          //           13.408322332907831
          //         ],
          //         [
          //           77.34100341796875,
          //           13.181119724574392
          //         ]
          //       ]
          //     }
          //   },
          //   {
          //     "type": "Feature",
          //     "properties": {},
          //     "geometry": {
          //       "type": "LineString",
          //       "coordinates": [
          //         [
          //           77.35748291015625,
          //           13.309448548494611
          //         ],
          //         [
          //           77.4810791015625,
          //           13.226577250436602
          //         ]
          //       ]
          //     }
          //   }
          // ]
          // "features": [
          //   {
          //     "type": "Feature",
          //     "properties": {
          //       "temperature":30
          //     },
          //     "geometry": {
          //       "type": "Point",
          //       "coordinates": [
          //         77.67229,
          //         12.92415
          //       ]
          //     }
          //   },
          //   {
          //     "type": "Feature",
          //     "properties": {
          //       "temperature":50
          //     },
          //     "geometry": {
          //       "type": "Point",
          //       "coordinates": [
          //         77.74935,12.96691

          //       ]
          //     }
          //   }
          // ]
        },
        cluster: true,
        clusterMaxZoom: 50, // Max zoom to cluster points on
        clusterRadius: 50,
        "clusterProperties": { // keep separate counts for each magnitude category in a cluster
          "canceled": ["+", ["case", cancel2, 1, 0]],
          "unavailable": ["+", ["case", cancel1, 1, 0]],
          "sun": ["+", ["case", sun, 1, 0]],
        }
      
      });

        // maps.addLayer({
        //   id:'points',
        //   type:'circle',
        //   source:'pointsource',
        //   paint:{
        //     // "line-opacity":0.8,
        //     // "line-color": "#B42222",
        //     // "line-dasharray":[32],
        //     // "line-width":0.6,
        //     "circle-radius":{
        //       property : "Car_Cancellation",
        //       stops :[
        //         [{zoom:0,value:0}, 1],[{zoom:0,value:1}, 1],
        //         [{zoom:12,value:0}, 2],[{zoom:12,value:1}, 3],
        //         [{zoom:20,value:0}, 5],[{zoom:20,value:1}, 5]]
        //     },
        //     //  "circle-radius": 2,
        //     // "circle-color": "#B42222"
        //     "circle-color" :{
        //       property : "Car_Cancellation",
        //       stops : [[0, 'blue'],[1,'red']]
        //     }
        //   }
        // })
        that.maps.addLayer({
          id: "clusters",
          type: "circle",
          source: "pointsource",
          filter: ["has", "point_count"],
          paint: {
          // Use step expressions (https://docs.mapbox.com/mapbox-gl-js/style-spec/#expressions-step)
          // with three steps to implement three types of circles:
          //   * Blue, 20px circles when point count is less than 100
          //   * Yellow, 30px circles when point count is between 100 and 750
          //   * Pink, 40px circles when point count is greater than or equal to 750
          "circle-color": [
          "step",
          ["get", "point_count"],
          "#51bbd6",
          100,
          "#f1f075",
          750,
          "#f28cb1"
          ],
          "circle-radius": [
          "step",
          ["get", "point_count"],
          20,
          100,
          30,
          750,
          40
          ]
          }
        });
        that.maps.addLayer({
        id: "cluster-count",
        type: "symbol",
        source: "pointsource",
        // filter: ["has", "point_count"],
        filter: ['all',
        ['==', ['get', 'cluster'], true]],
        // filter: ['sun'],
        layout: {
          "text-field":"{point_count_abbreviated}",
        // "text-field": "u:"+"{unavailable}"+"c:"+"{canceled}",
        "text-font": [ "Arial Unicode MS Bold"],
        "text-size": 12
        }
      });
      that.maps.addLayer({
        id: "unclustered-point",
        type: "circle",
        source: "pointsource",
        filter: ["!", ["has", "point_count"]],
        paint: {
        "circle-color": "#11b4da",
        "circle-radius": 4,
        "circle-stroke-width": 1,
        "circle-stroke-color": "#fff"
        }
      });
      var that2=that;
      that.maps.on('click', e =>{
        var result= that2.maps.queryRenderedFeatures(e.point, {layers:['cluster-count']})
        if(result.length){
          const popup = new mapboxgl.Popup();
          var unavaliableCount = result[0].properties.unavailable;
          var cancelCount = result[0].properties.canceled;
          popup.setLngLat(e.lngLat).
            setHTML( `<h3>Unavailable:${unavaliableCount}</h3><h3>Canceled:${cancelCount}</h3>`).
            addTo(that2.maps)
        }
        console.log(result);
        debugger
      })
      })
      

      // this.map.addControl(new mapboxgl.Navigation())
    }
    
  },
  mounted() {
    // this.createMap();
  }
    
}
</script>

