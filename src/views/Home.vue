<script>
/* global mapboxgl*/
import Darkmode from "darkmode-js";

export default {
  data: function () {
    return {
      message: "Mapp App",
      markerMessage: "",
      map: "",
      mapCenter: [-87.6298, 41.8781],
      places: [
        { lat: 41.95032207658536, lng: -87.67559685720875, description: "Trader J!" },
        { lat: 41.951482047297176, lng: -87.66693694670265, description: "Me" },
        { lat: 41.95439629557461, lng: -87.67494352937629, description: "Train" },
        { lat: 41.94515314989085, lng: -87.66377633106227, description: "Pizza lives here" },
      ],
      darkmode: new Darkmode(),
      currentTime: new Date(),
    };
  },
  created: function () {},
  methods: {
    addMarker: function () {
      var popupMessage = new mapboxgl.Popup({ offset: 25 }).setText(this.markerMessage);
      var markerrr = new mapboxgl.Marker({ draggable: true })
        .setLngLat(this.mapCenter)
        .setPopup(popupMessage)
        .addTo(this.map);
      console.log(markerrr);
      this.markerMessage = "";
    },
    toggleDarkmode: function () {
      this.darkmode.toggle();
      console.log(this.currentTime);
    },
  },
  mounted: function () {
    mapboxgl.accessToken = process.env.VUE_APP_MAPPBOX_APPBOX_KEY;
    this.map = new mapboxgl.Map({
      container: "map", // container ID
      style: "mapbox://styles/mapbox/streets-v11", // style URL
      center: this.mapCenter, // starting position [lng, lat]
      zoom: 13, // starting zoom
    });
    var popup = new mapboxgl.Popup({ offset: 25 }).setText("Good ol' Chicago");
    var marker = new mapboxgl.Marker().setLngLat(this.mapCenter).setPopup(popup).addTo(this.map);
    if (marker) {
      console.log(this.map);
    }
    this.places.forEach((place) => {
      popup = new mapboxgl.Popup({ offset: 25 }).setText(place["description"]);
      new mapboxgl.Marker().setLngLat([place["lng"], place["lat"]]).setPopup(popup).addTo(this.map);
    });
    this.map.on("load", () => {
      // Insert the layer beneath any symbol layer.
      const layers = this.map.getStyle().layers;
      const labelLayerId = layers.find((layer) => layer.type === "symbol" && layer.layout["text-field"]).id;

      // The 'building' layer in the Mapbox Streets
      // vector tileset contains building height data
      // from OpenStreetMap.
      this.map.addLayer(
        {
          id: "add-3d-buildings",
          source: "composite",
          "source-layer": "building",
          filter: ["==", "extrude", "true"],
          type: "fill-extrusion",
          minzoom: 15,
          paint: {
            "fill-extrusion-color": "#aaa",

            // Use an 'interpolate' expression to
            // add a smooth transition effect to
            // the buildings as the user zooms in.
            "fill-extrusion-height": ["interpolate", ["linear"], ["zoom"], 15, 0, 15.05, ["get", "height"]],
            "fill-extrusion-base": ["interpolate", ["linear"], ["zoom"], 15, 0, 15.05, ["get", "min_height"]],
            "fill-extrusion-opacity": 0.6,
          },
        },
        labelLayerId
      );
    });
  },
};
</script>

<template>
  <div class="home">
    <button v-on:click="toggleDarkmode()">TOGGLE DARKMODE</button>
    <h1>{{ message }}</h1>
    <input type="text" v-model="markerMessage" />
    <button v-on:click="addMarker()">add marker somewhere</button>
    <div id="map"></div>
  </div>
</template>

<style>
#map {
  height: 500px;
  width: 1000px;
}
</style>
