<template>
  <div class="mapsearch-area bg-blue plr-140 ptb-50">
    <div class="container">
      <div class="row">
        <div class="col-md-3 col-sm-6 col-xs-12 form-group">
          <input
            type="text"
            class="form-control search-field"
            data-default-value
            value
            name="title"
            v-model="mFilterForm.address"
            v-on:keyup="loadHomesOnMap()"
            placeholder="Address, Zip, Neighborhood"
          />
          <a href="#">
            <i class="icon-search2"></i>
          </a>
        </div>
        <div class="col-md-3 col-sm-6 col-xs-12 form-group">
          <select
            name="type"
            title="Property Types"
            class="search-field form-control"
            data-default-value
            v-on:change="loadHomesOnMap()"
            v-model="mFilterForm.type"
          >
            <option value selected>Property Types</option>
            <option v-for="t in mType">{{t}}</option>
          </select>
        </div>

        <div class="col-md-3 col-sm-6 col-xs-12 form-group">
          <select
            name="neighborhoods"
            title="Neighborhoods"
            class="search-field form-control"
            data-default-value
            v-model="mFilterForm.pNeighbour"
            v-on:change="loadHomesOnMap()"
          >
            <option value>Any Neighborhoods</option>
            <option
              v-for="neighbour in neighbours"
              :key="neighbour.id"
              :value="neighbour.id"
            >{{neighbour.title}}</option>
          </select>
        </div>
        <div class="col-md-3 col-sm-6 col-xs-12 form-group">
          <select
            v-model="mFilterForm.price"
            name="price"
            title="Price"
            class="search-field form-control"
            data-default-value
            v-on:change="loadHomesOnMap()"
          >
            <option>Any Price</option>
            <option v-for="p in mPrice" :value="(p[0],p[1])">${{p[0]}} - ${{p[1]}}</option>
          </select>
        </div>
      </div>
    </div>
    <google-map :center="mapICenter" :zoom="14" style="width: 100%; height: 500px">
      <gmap-custom-marker
        v-for="(home, index) in homes"
        :key="index"
        :marker="{ lat: home.lat, lng: home.lng }"
        @click.native="toggleInfoWindow(home, index)"
      >
        <img class="pin-img" src="vue/images/icons/map-marker-2.png" />
        <gmap-info-window
          :options="infoOptions"
          :position="infoWindowPos"
          :opened="infoWinOpen"
          @closeclick="infoWinOpen=false"
        >
          <div v-html="infoContent"></div>
        </gmap-info-window>
      </gmap-custom-marker>
    </google-map>
  </div>
</template>

<script>
export default {
  data() {
    return {
      mapICenter: { lat: 40.7178, lng: -74.0431 },
      homes: "",
      home: "",
      neighbours: {},
      infoContent: "",
      mPrice: "",
      mType: "",
      display_mode: 3,
      infoWindowPos: { lat: 0, lng: 0 },
      infoWinOpen: false,
      currentMidx: null,
      mFilterForm: new Form({
        type: "",
        pNeighbour: "",
        price: "",
        address:""
      }),
      infoOptions: {
        pixelOffset: {
          width: 0,
          height: -70
        }
      }
    };
  },
  methods: {
    toggleInfoWindow: function(home, idx) {
      this.infoWindowPos = {
        lat: Number(home.lat),
        lng: Number(home.lng)
      };
      this.infoContent = this.getInfoWindowContent(home);

      //check if its the same marker that was selected if yes toggle
      if (this.currentMidx == idx) {
        this.infoWinOpen = !this.infoWinOpen;
      }
      //if different marker set infowindow to open and reset current marker index
      else {
        this.infoWinOpen = true;
        this.currentMidx = idx;
      }
    },
    loadHomesOnMap: function() {
      this.mFilterForm
        .post("/api/home-map-house-list-filter")
        .then(({ data }) => {
          this.homes = data;
        });
    },
    getInfoWindowContent: function(home) {
      return `<div class="map-property">
              <a href="#"><img src="/uploads/homes/${home.featured_image}" alt=""></a>
              <h6><a href="#">${home.title}</a></h6>
              <h5>$${home.price}</h5>
              <p></p>
              </div>`;
    },
    getHomes() {
      axios
        .get("/api/home-map-houseList")
        .then(res => {
          this.homes = res.data;
          let price = [];
          let mRange = [];
          let type = [];
          $.each(res.data, function(key, value) {
            price.push(value.price);
            if (type.indexOf(value.type) == -1) type.push(value.type);
          });
          this.mType = type;
          let max = Math.max(...price);
          let min = Math.min(...price);
          const incrementVal = 25000;
          while (min < max) {
            let p = [min, min + incrementVal];
            min = min + incrementVal;
            mRange.push(p);
          }
          this.mPrice = mRange;
          // this.$Progress.finish();
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    loadHomeNeighbour() {
      axios
        .get("api/home-neighbour")
        .then(({ data }) => (this.neighbours = data));
    }
  },
  mounted() {
    this.getHomes();
    this.loadHomeNeighbour();
  },
  created() {}
};
</script>
