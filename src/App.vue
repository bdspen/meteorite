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
        },

        methods: {

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

            getMapData: function() {

                this.$http({
                    url: 'https://data.nasa.gov/resource/gh4g-9sfh.json',
                    method: 'GET'
                }).then(function(response) {
                    this.createMap();
                    this.createMarkers(response);
                }, function(response) {
                    console.log("ERROR: " + response);
                });

            },

            createMarkers: function(data) {

                for (let i = 0; i < data.data.length; i++) {
                    var meteorite = data.data[i];
                    // console.log(meteorite);
                    var marker = new google.maps.Marker({
                        position: new google.maps.LatLng(parseFloat(meteorite.reclat), parseFloat(meteorite.reclong)),
                        map: this.map,
                        title: meteorite.name
                    });

                    var dateObject = new Date(Date.parse(meteorite.year));
                    var date = dateObject.toDateString();

                    var infowindow = new google.maps.InfoWindow();
                    var infoWindowContent = '<h2>' + meteorite.name + '</h2>' + '<h4>' + meteorite.fall + ': ' + date + '</h4>' + '<h5>' + 'Mass: ' + meteorite.mass + ' grams' + '</h5>';

                    this.bindInfoWindow(marker, this.map, infowindow, infoWindowContent)
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

            }
        } //end methods
    } //end export default
</script>

<style>
    #map {
        max-width: 1800px;
        height: 800px;
        width: 100%;
    }
</style>
