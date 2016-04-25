<template>
    <div id="app">
      <div class="row">
        <div class="nine columns">
          <h1>Meteorite Impacts Worldwide</h1>
        </div>
        <div class="three columns">
          <label>Sort by:</label>
          <select class="u-full-width">
            <option>Name</option>
            <option>Mass</option>
            <option>Year</option>
          </select>
        </div>
      </div>
      <div class="row">
          <div id="map" class="nine columns"></div>
          <div class="three columns">
            <ul  id="meteorite-ul">
              <li v-for="item in meteorites" track-by="$index" class="list-item" v-on:click="goTo($index)">
                <h4 class="list-item-header">{{item.name}}</h4>
                <ul class="list-item-ul">
                  <li>Mass: {{item.mass}}</li>
                  <li>Year: {{item.year}}</li>
                </ul>
              </li>
            </ul>
          </div>
      </div>
  </div>
</template>

<script>
    var Vue = require('vue');
    Vue.use(require('vue-resource'));

    export default {

        data: function(){
          return {
            meteorites: {},
            markers: []
          }
        },

        ready: function() {
            this.initialize();
        },

        methods: {
            initialize: function(){
                this.getMeteoriteData();
            },

            createMap: function() {

                var map;
                map = new google.maps.Map(document.getElementById('map'), {
                    center: {
                        lat: 35,
                        lng: -98
                    },
                    zoom: 4,
                    mapTypeId: google.maps.MapTypeId.SATELLITE
                });
                this.map = map;

            },

            getMeteoriteData: function() {

                this.$http({
                    url: 'https://data.nasa.gov/resource/gh4g-9sfh.json',
                    method: 'GET'
                }).then(function(response) {
                    this.meteoriteData = response;
                }, function(response) {
                    console.log("ERROR: " + response);
                });

            },

            createMarkers: function(data) {

                for (let i = 0; i < data.data.length; i++) {
                    var meteorite = data.data[i];
                    var marker = new google.maps.Marker({
                        position: new google.maps.LatLng(parseFloat(meteorite.reclat), parseFloat(meteorite.reclong)),
                        map: this.map,
                        title: meteorite.name
                    });

                    var dateObject = new Date(Date.parse(meteorite.year));
                    var date = dateObject.toDateString();

                    var infowindow = new google.maps.InfoWindow();
                    var infoWindowContent = '<h2 class="info-text">' + meteorite.name + '</h2>' + '<h4 class="info-text">' + meteorite.fall + ': ' + date + '</h4>' + '<h5 class="info-text">' + 'Mass: ' + meteorite.mass + ' grams' + '</h5>';

                    this.bindInfoWindow(marker, this.map, infowindow, infoWindowContent);
                    this.markers.push(marker);
                }

            },

            bindInfoWindow: function(marker, map, infowindow, content) {

                infowindow.setContent(content);
                infowindow.opened = false;
                marker.addListener('click', function() {
                  if (infowindow.opened === true){
                    infowindow.close();
                    infowindow.opened = false;
                  } else {
                    infowindow.opened = true;
                    map.setZoom(8);
                    map.panTo(marker.getPosition());
                    infowindow.open(map, marker);
                  }
                });

            },

            goTo: function(i){
              new google.maps.event.trigger( this.markers[i], 'click' );
            }
        }, //end methods
        computed: {
          meteoriteData: {
            set: function(newValue){
              this.meteorites = newValue.data;
              this.createMap();
              this.createMarkers(newValue);
            }
          }
        }
    } //end export default
</script>

<style>
    #map {
        max-width: 1800px;
        height: 800px;
    }
    #meteorite-ul{
        height: 800px;
        overflow: scroll;
    }
    .list-item-header{
      margin-bottom: 0px;
    }
    .list-item-ul{
      margin-top: 0px;
      margin-bottom: 0px;
    }
    .list-item{
      border-top: 1px solid black;
      margin-right: 10px;
      margin-left: 10px;
      cursor: pointer;
    }
    .list-item :hover{
      transition: opacity .75s ease;
      opacity: .4
    }
    .info-text{
      margin: 0px;
    }
    li{
      list-style: none
    }
</style>
