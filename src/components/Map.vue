<template>
  <div>
    <h1> Visualize your data more precisely</h1>
    <div v-if="!mapLoaded" class="margin-top-12">
      <p >Choose csv file to upload <input type="file" class="dropbox" @change="processFile($event)"  width="50%"></p>
      <p> Please upload the csv in the desired format to get deep insights of your data. &#128523;</p>
    </div>
    <div v-if="mapLoaded">
      <p >Apply the filters to get more insights about the your data. &#128522;</p>
      <p>Month : 
      <select name="Months" v-model="monthSelected" @change="trafficChanged()">
        <option value="0">January</option>
        <option value="1">February</option>
        <option value="2">March</option>
        <option value="3">April</option>
        <option value="4">May</option>
        <option value="5">June</option>
        <option value="6">July</option>
        <option value="7">August</option>
        <option value="8">September</option>
        <option value="9">October</option>
        <option value="10">Novembor</option>
        <option value="11">December</option>
        <option value="-1">-</option>
      </select>
      Day:
      <select name="WeekDays" v-model="daySelected" @change="trafficChanged()">
        <option value="0">Sunday</option>
        <option value="1">Monday</option>
        <option value="2">Tuesday</option>
        <option value="3">Wednesday</option>
        <option value="4">Thrusday</option>
        <option value="5">Friday</option>
        <option value="6">Saturday</option>
        <option value="-1">-</option>
      </select>
      Time:
      <select name="time" v-model="timeSelected" @change="trafficChanged()">
        <option value="0">0:00-4:00 Hrs</option>
        <option value="1">4:00-8:00 Hrs</option>
        <option value="2">8:00-12:00 Hrs</option>
        <option value="3">12:00-16:00 Hrs</option>
        <option value="4">16:00-20:00 Hrs</option>
        <option value="5">20:00-24:00 Hrs</option>
        <option value="-1">-</option>
      </select>
      Package Type:
      <select name="packageType" v-model="packageSelected" @change="trafficChanged()">
        <option value="1">4hrs & 40kms</option>
        <option value="2">8hrs & 80kms</option>
        <option value="3">6hrs & 60kms</option>
        <option value="4">10hrs & 100kms</option>
        <option value="5">5hrs & 50kms</option>
        <option value="6">3hrs & 30kms</option>
        <option value="7">12hrs & 120kms</option>
        <option value="0">-</option>
      </select>
      Travel:
      <select name="travelType" v-model="travelSelected" @change="trafficChanged()">
        <option value="1">Long distance</option>
        <option value="2">Point to Point</option>
        <option value="3">Hourly rental</option>
        <option value="0">-</option>
      </select></p>
    </div>
    
    <div id="map" class="displayMapSize" >
    </div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
export default {  
  name: 'HelloWorld',
  data () {
    return {
      maps : null,
      daySelected : -1,
      monthSelected : -1,
      timeSelected : -1,
      packageSelected : 0,
      travelSelected : 0,
      mapLoaded: false
    }
  },
  methods:{
    //method for filtering data based on filters applied
    trafficChanged(){
      var arr =  this.featureArray 

      // Filters on basis of month, if month is applied on the filter
      var month = parseInt(this.monthSelected)
      if( month != null && !(isNaN(month)) && month >=0){
        arr =  arr.filter(a => a.properties.month==month)
      }

      // Filters on basis of day, if day is applied on the filter
      var day = parseInt(this.daySelected)
      if(day !=null && !(isNaN(day)) && day>=0){
        arr = arr.filter(a => a.properties.day==day)
      }

      // Filters on basis of time, if time is applied on the filter
      var time = parseInt(this.timeSelected)
      if(time != null && !(isNaN(time)) && time>=0){
        arr = arr.filter(a => (a.properties.hours>=(time*4) && a.properties.hours<((time*4)+4)))
      }

      // Filters on basis of package_type, if package_type is applied on the filter
      var packageId = this.packageSelected
      if(packageId>0){
        arr = arr.filter(a => a.properties.package_id==packageId)
      }

      // Filters on basis of travel_id, if travel_id is applied on the filter
      var travel = parseInt(this.travelSelected)
      if(travel>0){
        arr = arr.filter(a => a.properties.travel_type_id==travel)
      }

      //set filtered data to map
      this.maps.getSource("pointsource").setData({
          type: 'FeatureCollection',
          features: arr
        });
        this.popup.remove()
    },

    //converts input csv into geojson format
    csvToJson(csv){
      var lines = csv.split('\n')
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
        var currentLine = line.split(',')
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
          geoJsonArray.push(geoObj);
      })
      geoJsonArray.pop()
      this.featureArray = geoJsonArray;
      this.createMap()
    },

    //reads the csv file
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

    //creation of map control and adding layer on top of it
    createMap () {
      mapboxgl.accessToken = 'pk.eyJ1Ijoic2lkZGhhcnRoYTk0IiwiYSI6ImNqdzh2NDNjbTJ0Ym4zeXBnc3R6dTRrNDgifQ.4fXJ1TzgULyVy2w7ndZW-Q'

      // init the map
      this.maps = new mapboxgl.Map({
        container: map,
        style: 'mapbox://styles/mapbox/light-v9',
        minzoom: 1,
        center: [77.63625469,12.98246088],
        zoom: 12
      })
      // filters for classifying earthquakes into five categories based on magnitude
      var cancel1 = ["==", ["get", "Car_Cancellation"], 0];
      var cancel2 = ["==", ["get", "Car_Cancellation"], 1];
     
      
      // colors to use for the categories
      var colors = ['#fed976', '#feb24c'];
 
      var that = this
      this.maps.on('load', function(){
        that.maps.addSource('pointsource',{
          type:'geojson',
          data:{
            "type": "FeatureCollection",
            "features" : that.featureArray
          },
          cluster: true,
          clusterMaxZoom: 50, // Max zoom to cluster points on
          clusterRadius: 50,
          "clusterProperties": { // keep separate counts for each magnitude category in a cluster
            "canceled": ["+", ["case", cancel2, 1, 0]],
            "unavailable": ["+", ["case", cancel1, 1, 0]],
          }
        
        });
        that.mapLoaded = true;

        // adding cluster layer
        that.maps.addLayer({
          id: "clusters",
          type: "circle",
          source: "pointsource",
          filter: ["has", "point_count"],
          paint: {
          // with three steps to implement three types of circles
          "circle-color": [
          "step",
          ["get", "point_count"],
          "#ffd700",
          100,
          "#ffa500",
          500,
          "#ff6347",
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

        //Adding text in the clusters
        that.maps.addLayer({
        id: "cluster-count",
        type: "symbol",
        source: "pointsource",
        filter: ["has", "point_count"],
        layout: {
          "text-field":"{point_count_abbreviated}",
          "text-font": [ "Arial Unicode MS Bold"],
          "text-size": 12
        }
      });

      //unclustered point i.e, only 1 point is there
      that.maps.addLayer({
        id: "unclustered-point",
        type: "circle",
        source: "pointsource",
        filter: ["!", ["has", "point_count"]],
        
        paint: {
        "circle-color": "#11b4da",
        "circle-radius": 5,
        "circle-stroke-width": 1,
        "circle-stroke-color": "#fff"
        }
      });
      var that2=that;
      that.popup = new mapboxgl.Popup({closeButton: false});

      //showing popup on mouse move
      that.maps.on('mousemove', e =>{
        var result= that2.maps.queryRenderedFeatures(e.point, {layers:['cluster-count']})
        if(result.length){
          var unavaliableCount = result[0].properties.unavailable;
          var cancelCount = result[0].properties.canceled;
          that2.popup.setLngLat(e.lngLat).
            setHTML( `<h3>Unavailable:${unavaliableCount}</h3><h3>Canceled:${cancelCount}</h3>`).
            addTo(that2.maps)
        }
        else{
          that2.popup.remove()
        }
      })
    })
      
    }
    
  },
    
}
</script>

