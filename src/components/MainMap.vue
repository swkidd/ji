<template>
  <div>
    <l-map
      ref="map"
      :center="center"
      :zoom="zoom"
      style="height: 100vh; width: 100%; z-index: 0; user-select: none;"
      @click="mapClick"
    >
      <l-tile-layer :url="url" :attribution="attribution" />
      <l-marker
        v-for="(marker, i) in markers.filter(m => m.latlng)"
        :key="i"
        style="user-select: none;"
        :lat-lng="marker.latlng"
      >
        <l-popup v-if="marker.image">
          <div
            style="cursor: pointer;"
            @click="
              $router.push({ name: 'marker-detail', params: { id: marker.id } })
            "
          >
            <v-img width="200" :src="marker.image" />
          </div>
        </l-popup>
      </l-marker>
    </l-map>
    <v-card
      justify="start"
      class="d-flex flex-row marker-image"
      :style="scrollStyles"
    >
      <v-sheet
        v-for="(marker, i) in markers"
        :key="i"
        style="cursor: pointer"
        @click="center = marker.latlng; zoom = 24"
      >
        <v-img :src="marker.image" width="75" class="ma-3" />
      </v-sheet>
    </v-card>
  </div>
</template>

<script>
import Marker from "@/models/Marker";
import { LMap, LTileLayer, LMarker, LPopup } from "vue2-leaflet";
import L from "leaflet";

export default {
  name: "MainMap",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  props: {
    following: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  data() {
    return {
      center: [35.77, 139.3],
      zoom: 13,
      isFollowing: false,
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
    };
  },
  watch: {
    following (val) {
      if (val && !this.isFollowing) {
        this.initFollow()
      }
    }
  },
  mounted() {
    Marker.fetch();
  },
  computed: {
    scrollStyles() {
      return this.$vuetify.breakpoint.mobile
        ? {
            maxWidth: "225px",
            left: "10px"
          }
        : {
            maxWidth: "500px",
            left: "50%",
            transform: "translateX(-50%)"
          };
    },
    markers() {
      console.log(Marker.all());
      return Marker.all();
    }
  },
  methods: {
    initFollow() {
      this.$nextTick(() => {
        this.$refs.map.mapObject.on("locationfound", this.onLocationFound);
        this.$refs.map.mapObject.on("locationerror", this.onLocationError);
        this.$refs.map.mapObject.locate({ setView: true });
      });
      this.isFollowing = true
    },
    onLocationFound(e) {
      var radius = e.accuracy;

      L.marker(e.latlng).addTo(this.$refs.map.mapObject);

      L.circle(e.latlng, radius).addTo(this.$refs.map.mapObject);
    },
    onLocationError(e) {
      alert(e.message);
    },
    mapClick(e) {
      this.$emit("click", e);
    }
  }
};
</script>
<style scoped>
.marker-image {
  position: fixed;
  bottom: 20px;
  overflow-x: auto;
  overflow-y: none;
}
</style>
