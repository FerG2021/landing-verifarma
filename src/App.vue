<template>
  <div class="contenedor-principal">
    <div class="logo-verifarma">
      <img src="verifarmalogo.png" class="logo-mobile" />
      <img src="verifarma.png" class="logo-desktop" />
    </div>

    <div class="flex flex-column gap-2 contendor-input-metros">
      <label for="cantMetros" class="label-input">Cantidad de metros</label>
      <div class="contenedor-input-buscador">
        <InputNumber
          id="cantMetros"
          v-model="cantMetros"
          aria-describedby="username-help"
          placeholder="Cantidad de metros"
        />
        <Button
          label="Buscar"
          class="btn-buscar"
          @click="getPuntos()"
          :loading="loadingBtnBuscar"
        />
      </div>

      <small id="username-help" class="label-input"
        >Ingresá la cantidad máxima de metros en los que buscas una
        farmacia</small
      >
    </div>

    <GMapMap
      :center="center"
      :zoom="15"
      map-type-id="terrain"
      class="g-map"
      @click="addMarker"
    >
      <GMapMarker
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :clickable="true"
        :draggable="true"
        :icon="m.position.img"
        @click="openMarker(m.position.id)"
      >
        <div v-if="m.position.id != 0">
          <GMapInfoWindow
            :closeclick="true"
            @closeclick="openMarker(null)"
            :opened="openedMarkerID === m.position.id"
          >
            <div>{{ m.position.nombre }}</div>
            <div>{{ m.position.direccion }}</div>
          </GMapInfoWindow>
        </div>
      </GMapMarker>
    </GMapMap>

    <Toast />
  </div>
</template>

<script>
export default {
  data() {
    return {
      // form
      cantMetros: null,
      lat: null,
      lng: null,
      openedMarkerID: null,

      loadingBtnBuscar: false,

      // center: { lat: -27.788322, lng: -64.259941 },
      center: { lat: null, lng: null },
      // markers: [
      //   {
      //     position: {
      //       lat: -27.788322,
      //       lng: -64.259941,
      //     },
      //   },
      // ],

      markers: [
        {
          position: {
            id: 0,
            lat: null,
            lng: null,
            img: null,
            nombre: null,
            direccion: null,
          },
        },
      ],

      // markers: [],
    };
  },

  created() {
    this.obtenerUbicacion();
  },

  methods: {
    obtenerUbicacion() {
      if (!"geolocation" in navigator) {
        return alert(
          "Tu navegador no soporta el acceso a la ubicación. Intenta con otro"
        );
      }

      const onUbicacionConcedida = (ubicacion) => {
        console.log("Tengo la ubicación: ", ubicacion);

        console.log("ubicacion.coords.latitude");
        console.log(ubicacion.coords.latitude);

        console.log("ubicacion.coords.longitude");
        console.log(ubicacion.coords.longitude);

        this.center.lat = ubicacion.coords.latitude;
        this.center.lng = ubicacion.coords.longitude;

        this.markers[0].position.id = 0;
        this.markers[0].position.lat = ubicacion.coords.latitude;
        this.markers[0].position.lng = ubicacion.coords.longitude;

        this.lat = ubicacion.coords.latitude;
        this.lng = ubicacion.coords.longitude;
      };

      const onErrorDeUbicacion = (err) => {
        console.log("Error obteniendo ubicación: ", err);
      };

      const opcionesDeSolicitud = {
        enableHighAccuracy: true, // Alta precisión
        maximumAge: 0, // No queremos caché
        timeout: 5000, // Esperar solo 5 segundos
      };

      // Solicitar
      navigator.geolocation.getCurrentPosition(
        onUbicacionConcedida,
        onErrorDeUbicacion,
        opcionesDeSolicitud
      );
    },

    async getPuntos() {
      if (this.cantMetros != null) {
        this.loadingBtnBuscar = true;
        this.markers = [
          {
            position: {
              lat: null,
              lng: null,
            },
          },
        ];
        this.markers[0].position.lat = this.lat;
        this.markers[0].position.lng = this.lng;

        let params = {
          lat: this.lat,
          lng: this.lng,
          cantMetros: this.cantMetros,
        };

        await this.axios
          .post("/api/farmacias-puntos", params)
          .then((response) => {
            if (response.data.code >= 200 && response.data.code < 300) {
              response.data.data.forEach((elemento) => {
                let position = {};
                position.id = elemento.id;
                position.lat = parseFloat(elemento.lat);
                position.lng = parseFloat(elemento.lng);
                position.img = "./farmacia.png";
                position.nombre = elemento.name;
                position.direccion = elemento.address;
                let fila = { position };
                this.markers.push(fila);
              });
            } else {
              this.$toast.add({
                severity: "error",
                summary: "Se ha producido un error",
                detail: response.data.message,
                life: 5000,
              });
            }
          });

        this.loadingBtnBuscar = false;
      } else {
        this.$toast.add({
          severity: "error",
          summary: "Error",
          detail:
            "Debes ingresar la cantidad de metros máximos en los que buscas una farmacia",
          life: 5000,
        });
      }
    },

    openMarker(param) {
      this.openedMarkerID = param;
    },
  },
};
</script>

<style scoped>
/*  */
/* WEB */
/*  */
@media all and (min-width: 961px) {
  .contenedor-principal {
    display: flex;
    flex-direction: column;
  }

  .logo-verifarma {
    margin: auto;
  }

  .logo-mobile {
    display: none;
  }

  .logo-desktop {
    height: 12vh;
    margin: auto;
  }

  .btn-buscar {
    margin-left: 10px !important;
  }

  .contendor-input-metros {
    margin: auto;
    height: 12vh;
    display: flex;
  }

  .contenedor-input-buscador {
    margin: auto;
  }

  .label-input {
    text-align: center;
  }

  .g-map {
    width: 90%;
    height: 65vh;
    margin: auto;
  }
}

/*  */
/* MOBILE */
/*  */
@media all and (max-width: 960px) {
  .contenedor-principal {
    display: flex;
    flex-direction: column;
  }

  .logo-verifarma {
    margin: auto;
  }

  .logo-desktop {
    display: none;
  }

  .logo-mobile {
    height: 12vh;
    margin: auto;
  }

  .btn-buscar {
    margin-left: 10px !important;
  }

  .contendor-input-metros {
    margin: auto;
    height: 12vh;
  }

  .contenedor-input-buscador {
    margin: auto;
  }

  .label-input {
    text-align: center;
  }

  .g-map {
    width: 90%;
    height: 70vh;
    margin: auto;
  }
}
</style>
