<template>
  <div id="map" v-if="loading == false">
    <ol-map
    :loadTilesWhileAnimating="true"
    :loadTilesWhileInteracting="true"
    style="height: 100%; width: 100%;"
  >
    <ol-view
      :center="center"
      :zoom="zoom"
      :projection="projection"
      :extent="[-20037508.34, -20037508.34, 20037508.34, 20037508.34]"
      :enableRotation="false"
    />
    <ol-tile-layer>
      <ol-source-osm />
    </ol-tile-layer>

    <ol-overlay
      v-if="fires.length > 0"
      v-for="fire in fires"
      :position="convertTo3857(fire.geometry[0].coordinates[0], fire.geometry[0].coordinates[1])"
      :key="fire.id"
      :wrapX="true"
      >
      <template v-slot="slotProps">
        <div class="fire fire-icon" :id="fire.id" @click="openDetails(fire.id)">
          <Icon icon="mdi:fire-alert" color=""/>
        </div>
      </template>
    </ol-overlay>
  </ol-map>
  </div>
  <div id="fire-info" v-if="details_active && fire_details != null">
    <h2>Fire info</h2><hr/>
    <p>{{ fire_details.title }}</p>
  </div>

</template>

<script>
import { defineComponent, ref } from 'vue';
import axios from 'axios';
import { Icon } from '@iconify/vue';

export default defineComponent({
  components: {
		Icon,
	},

  data(){
    return {
      loading: false,

      zoom: 10,
      center: [2.3522219, 48.856614],
      projection: 'EPSG:3857',
      
      fires: [],
      details_active: false,
      opened_details_id: "",
      fire_details: null
    }
  },

  async mounted() {
    // Center map on user position
    navigator.geolocation.getCurrentPosition(position => {
      this.center = this.convertTo3857(position.coords.longitude, position.coords.latitude);
    }, error => {
      this.center = this.convertTo3857(2.3522219, 48.856614);
    });

    // Wildfires stuff
    let uri = "https://eonet.gsfc.nasa.gov/api/v3/events?category=wildfires&status=open";
    const response = await axios.get(uri);
    this.fires = response.data.events;
    //const response_json = await response.json();
    console.log(response.data.events);
  },

  methods : {
    convertTo3857(lon, lat) {
      var x = lon * 20037508.34 / 180;
      var y = Math.log(Math.tan((90 + lat) * Math.PI / 360)) / (Math.PI / 180);
      y = y * 20037508.34 / 180;
      return [x, y];
    },

    openDetails(id) {
      if(this.details_active == true && this.opened_details_id == id){
        this.closeDetails();
        return;
      }
      if(this.details_active == true){
        this.closeDetails();
      }

      this.details_active = true;
      this.opened_details_id = id;
      document.getElementById(id).classList.remove("fire-icon");
      document.getElementById(id).classList.add("fire-icon-active");

      let fire = this.fires.find(fire => fire.id == id);
      this.fire_details = {title: fire.title}
    },

    closeDetails(){
      this.details_active = false;
      document.getElementById(this.opened_details_id).classList.remove("fire-icon-active");
      document.getElementById(this.opened_details_id).classList.add("fire-icon");
      this.opened_details_id = "";
    }
  }

});
</script>