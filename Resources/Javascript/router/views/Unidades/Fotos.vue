<template>
  <section class="bg-light m-4">
    <div class="columns">
      <div class="column is-6 is-offset-3">
        <div class="box">
          <h3 class="title mb-5 is-3">
            {{ title }}
          </h3>

          <b-field label="Foto destacada">
            <vue-dropzone
              id="dzFeaturedPhoto"
              ref="dzFeaturedPhoto"
              :options="dzFeaturedPhotoOptions"
              @vdropzone-error="dzFeaturedPhotoError"
              @vdropzone-removed-file="dzFeaturedPhotoRemovedFile"
              @vdropzone-processing="dzFeaturedPhotoProcessing"
              @vdropzone-success-multiple="dzFeaturedPhotoSuccess"
              @vdropzone-upload-progress="dzFeaturedPhotoUploadProgress"
            />
          </b-field>
          <p class="mb-4">Puedes subir una foto de hasta 8Mb.</p>

          <b-field label="Galería de fotos">
            <vue-dropzone
              id="dzUnfeaturedPhotos"
              ref="dzUnfeaturedPhotos"
              :options="dzUnfeaturedPhotosOptions"
              @vdropzone-error="dzUnfeaturedPhotosError"
              @vdropzone-removed-file="dzUnfeaturedPhotosRemovedFile"
              @vdropzone-processing="dzUnfeaturedPhotosProcessing"
              @vdropzone-success-multiple="dzUnfeaturedPhotosSuccess"
              @vdropzone-upload-progress="dzUnfeaturedPhotosUploadProgress"
            />
          </b-field>
          <p class="mb-4">Puedes subir hasta 8 fotos de hasta 8Mb cada una.</p>

          <b-button
            type="is-primary"
            expanded="true"
            size="is-large"
            @click="$router.push({ name: 'Mis Inmuebles' })"
          >
            &larr; Ir a "Mis Inmuebles"
          </b-button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import app from "@/app"
import { alertErrorMessage, getSavedState } from "@/utilities/helpers"
import { unidadesMethods } from "@mvp/store/helpers"
import { dz } from "@/utilities/mixins/dz"
import { photosMethods } from "@/store/helpers"

import vue2Dropzone from "vue2-dropzone"
import "vue2-dropzone/dist/vue2Dropzone.min.css"

var csrfToken = document.head.querySelector("meta[name=\"csrf-token\"]").content
var token = getSavedState("auth.token")

export default {
    components: {
        VueDropzone: vue2Dropzone
    },

    mixins: [dz],

    data() {
        return {
            action: app.apiUrl + "/unidades",
            dzFeaturedPhotoOptions: {
                addRemoveLinks: true,
                autoProcessQueue: true,
                dictDefaultMessage: "<i class='fa fa-cloud-upload'></i><br/>Hacé click o arrastrá una foto hacía acá",
                headers: {
                    "X-CSRF-TOKEN": csrfToken,
                    "Authorization": "Bearer " + token
                },
                maxFiles: 1,
                maxFilesize: 8,
                method: "put",
                parallelUploads: 1,
                paramName: "featured_photo",
                uploadMultiple: true,
                url: app.apiUrl + "/unidades"
            },
            dzUnfeaturedPhotosOptions: {
                addRemoveLinks: true,
                autoProcessQueue: true,
                dictDefaultMessage: "<i class='fa fa-cloud-upload'></i><br/>Hacé click o arrastrá una o más fotos hacía acá",
                headers: {
                    "X-CSRF-TOKEN": csrfToken,
                    "Authorization": "Bearer " + token
                },
                maxFiles: 8,
                maxFilesize: 8,
                method: "put",
                parallelUploads: 8,
                paramName: "unfeatured_photos",
                uploadMultiple: true,
                url: app.apiUrl + "/unidades"
            },
            isDestroying: false,
            method: "put",
            prepared: false,
            title: "Agrega o actualiza las fotos de tu Departamento",
            url: app.apiUrl + "/unidades"
        }
    },

    watch: {
        "$route" (value) {
            if (value.name === "Fotos Unidad" && this.prepared) {
                this.prepare()
            }
        }
    },

    beforeDestroy() {
        this.isDestroying = true
    },

    created() {
        return this.prepare().then(this.prepared = true)
    },

    methods: {
        ...photosMethods,
        ...unidadesMethods,

        dzFeaturedPhotoSuccess() {
            this.$buefy.toast.open({
                message: "La foto destacada fue actualizada.",
                type: "is-success"
            })
        },

        dzUnfeaturedPhotosSuccess() {
            this.$buefy.toast.open({
                message: "La galería de fotos fue actualizada.",
                type: "is-success"
            })
        },

        prepare() {
            var unidad = this.fetchOneUnidad(this.$route.params.id)

            this.url = app.apiUrl + "/unidades/" + this.$route.params.id

            return Promise.all([unidad])
                .then(values => {
                    if (values[0].id_inmueble.featured_photo) {
                        this.dzFeaturedPhotoMounted(values[0].id_inmueble.featured_photo)
                    }

                    if (values[0].id_inmueble.unfeatured_photos) {
                        this.dzUnfeaturedPhotosMounted(values[0].id_inmueble.unfeatured_photos)
                    }

                    return values
                })
        }
    }
}
</script>
