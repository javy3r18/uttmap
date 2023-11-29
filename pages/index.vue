<template>
  <b-container fluid class="m-0 p-0">
    <div id="map"></div>

    <div v-if="docenciasFeatures">
      <b-sidebar
        v-model="openSidebar"
        id="sidebar-right"
        :title="docenciasFeatures.parent"
        right
        shadow
      >
        <div class="px-3 py-2">
          <h2>{{ this.docenciasFeatures.name }}</h2>
          <hr>
          <p><b>Director(a): </b>{{ this.docenciasFeatures.principal }}</p>
          <p><b>Carreras: </b>{{ this.docenciasFeatures.careers }}</p>
          <p><b>Salones: </b>{{ this.docenciasFeatures.classrooms }}</p>
          <p><b>Laboratorios: </b>{{ this.docenciasFeatures.laboratories }}</p>
          <br>
          <hr>
          <b-img src="https://uttijuana.edu.mx/wp-content/uploads/2021/10/Logo-UTT-1.png" fluid></b-img>
        </div>

      </b-sidebar>
    </div>

    <div class="img-container" v-if="openSidebar">
      <div class="images">
        <b-carousel
          id="carousel-1"
          v-model="slide"
          :interval="4000"
          controls
          indicators
          background="#ababab"
          img-width="1024"
          img-height="480"
          style="text-shadow: 1px 1px 2px #333"
          @sliding-start="onSlideStart"
          @sliding-end="onSlideEnd"
        >
          <!-- Text slides with image -->
          <b-carousel-slide
            caption="First slide"
            text="Nulla vitae elit libero, a pharetra augue mollis interdum."
            img-src="https://picsum.photos/1024/480/?image=52"
          ></b-carousel-slide>

          <!-- Slides with custom text -->
          <b-carousel-slide img-src="https://picsum.photos/1024/480/?image=54">
            <h1>Hello world!</h1>
          </b-carousel-slide>

          <!-- Slides with image only -->
          <b-carousel-slide
            img-src="https://picsum.photos/1024/480/?image=58"
          ></b-carousel-slide>

          <!-- Slides with img slot -->
          <!-- Note the classes .d-block and .img-fluid to prevent browser default image alignment -->
          <b-carousel-slide>
            <template #img>
              <img
                class="d-block img-fluid w-100"
                width="1024"
                height="480"
                src="https://picsum.photos/1024/480/?image=55"
                alt="image slot"
              />
            </template>
          </b-carousel-slide>

          <!-- Slide with blank fluid image to maintain slide aspect ratio -->
          <b-carousel-slide
            caption="Blank Image"
            img-blank
            img-alt="Blank image"
          >
            <p>
              Lorem ipsum dolor sit amet, consectetur adipiscing elit.
              Suspendisse eros felis, tincidunt a tincidunt eget, convallis vel
              est. Ut pellentesque ut lacus vel interdum.
            </p>
          </b-carousel-slide>
        </b-carousel>

        <p class="mt-4">
          Slide #: {{ slide }}<br />
          Sliding: {{ sliding }}
        </p>
      </div>
    </div>
  </b-container>
</template>

<script>
export default {
  data() {
    return {
      access_token:
        "pk.eyJ1IjoiamF2eTNyMTgiLCJhIjoiY2w4b3Q3aTdmMDB6djNvbzhycGtmYXF1MSJ9.55Zc-lWR2Oc7YzjBt9S5ow",
      map: {},
      openSidebar: false,
      docenciasFeatures: null,
      popup: null,
    };
  },

  mounted() {
    this.createMap();
    this.isMapLoaded();
  },

  methods: {
    createMap() {
      this.$mapboxgl.accessToken = this.access_token;
      this.map = new this.$mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/javy3r18/clm4cz03900gl01of1ffrhdm2",
        zoom: 17,
        pitch: 65, //inclination
        bearing: -100, // rotation
        center: [-116.824989, 32.461261],
        // maxBounds: [[-116.826585, 32.457475], [-116.824043, 32.464319]],
        antialias: true,
      });
    },

    isMapLoaded() {
      this.map.on("load", () => {
        //agrega controles de zoom y rotacion
        this.map.addControl(
          new this.$mapboxgl.NavigationControl(),
          "bottom-right"
        );

        //pone icono de mano apuntando en el mouse al hacer hover
        this.map.on("mouseenter", "docencias", (e) => {
          console.log(e)
          this.map.getCanvas().style.cursor = "pointer";

          this.map.setLayoutProperty("docencias3d", "visibility", "visible");

          this.createPopup(e);
        });

        //pone icono de mano en el mouse al dejar de hacer hover
        this.map.on("mouseleave", "docencias", () => {
          this.map.getCanvas().style.cursor = "grab";

          this.map.setLayoutProperty("docencias3d", "visibility", "none");

          this.popup.remove();
        });

        //funcionalidad para hacer click en los poligonos de docencias
        this.map.on("click", "docencias", (e) => {
          
          let features = this.map.queryRenderedFeatures(e.point, {
            layers: ["docencias"],
          });
          this.openSidebar = true;
          this.docenciasFeatures = features[0].properties;
          console.log(e);
        });
      });
    },

    createPopup(e) {

      let polygon = this.$turf.polygon([e.features[0].geometry.coordinates[0]]);
      let center = this.$turf.centroid(polygon);

      this.popup = new this.$mapboxgl.Popup({ closeButton: false })
        .setLngLat(center.geometry.coordinates)
        .setHTML(e.features[0].properties.name)
        .addTo(this.map);
    },
  },
};
</script>

<style>
#map {
  width: 100%;
  position: absolute;
  top: 0;
  bottom: 0;
}

.img-container {
  position: absolute;
  width: calc(100% - 320px);
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.6);
}

.images {
  width: 100%;
  margin-left: 100px;
  margin-right: 100px;
}
</style>
