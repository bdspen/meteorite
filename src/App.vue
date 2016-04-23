<template>
    <div id="app">
        <h1>Meteorite Impacts Worldwide</h1>
        <div class="row">
            <div id="map" class="ten columns"></div>
        </div>
    </div>
</template>

<script>
    var Vue = require('vue');
    Vue.use(require('vue-resource'));

    export default {

        ready: function() {
            this.getMapData();
            this.createMap();
        },
        methods: {
            createMap: function() {
                var map;

                map = new google.maps.Map(document.getElementById('map'), {
                    center: {
                        lat: 0,
                        lng: 45
                    },
                    zoom: 2
                });
            },
            getMapData: function() {
                this.$http({
                    url: 'https://data.nasa.gov/resource/gh4g-9sfh.json',
                    method: 'GET'
                }).then(function(response) {
                      console.log(response);
                      this.mapData = response;
                      this.createMarkers(this.mapData);
                }, function(response) {
                    console.log("ERROR: " + response);
                });
            },
            createMarkers: function(data) {
              for (let i = 0; i < data.data.length; i++) {
                var meteorite = data.data[i];
                const marker = new google.maps.Marker({
                  position: new google.maps.LatLng(parseFloat(meteorite.reclat), parseFloat(meteorite.reclong)),
                  map: map,
                  title: meteorite.name
                });
                data.data[i]
              }
            }
        }
    }
</script>

<style>
    #map {
        max-width: 1800px;
        height: 800px;
        width: 100%;
    }
</style>
