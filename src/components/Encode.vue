<template>
  <div class="encode">
    <el-button
      class="encode__btn"
      @click="openCropDialog"
    >Encode image</el-button>
    <el-dialog
      title="Encode and crop image"
      :visible.sync="showCropDialog"
    >
      <div class="crop-preview-wrapper">
        <div
          v-show="cropPreview"
          class="crop-preview"
        >
          <img
            ref="cropper"
            :src="cropPreview"
            alt="crop-preview"
          >
        </div>
      </div>
      <el-upload
        ref="encode"
        action=""
        :on-change="onChange"
        :http-request="onEncode"
        :before-encode="onBeforeEncode"
        :multiple="false"
        :show-file-list="false"
        :auto-encode="false"
      >
        <div slot="trigger">
          <button
            ref="encodeButton"
            style="display: none;"
          />
        </div>
        <div
          v-if="cropPreview"
          class="aspect-ratio-buttons"
        >
          <el-radio-group
            v-model="aspectRatio"
            size="mini"
            @change="changeAspectRation"
          >
            <el-radio-button :label="1">1:1</el-radio-button>
            <el-radio-button :label="4 / 3">4:3</el-radio-button>
            <el-radio-button :label="2 / 3">2:3</el-radio-button>
            <el-radio-button :label="16 / 9">16:9</el-radio-button>
            <el-radio-button :label="NaN">Free</el-radio-button>
          </el-radio-group>
        </div>
        <div class="encode-action">
          <el-button @click="$refs.encodeButton.click()">Select image</el-button>
          <el-button
            v-if="cropPreview"
            type="success"
            @click="onEncode"
          >Save</el-button>
        </div>
      </el-upload>
    </el-dialog>
  </div>
</template>

<script>
// import S3 from 'aws-sdk/clients/s3'
import { guid } from '@/util/helpers'
import Cropper from 'cropperjs'
import 'cropperjs/dist/cropper.css'

export default {
  name: 'Encode',

  props: {
    cropWidth: {
      type: Number,
      default: null
    },
    cropHeight: {
      type: Number,
      default: null
    },
    quality: {
      type: Number,
      default: 0.9
    }
  },

  data () {
    return {
      fileRaw: '',
      showCropDialog: false,
      cropper: undefined,
      aspectRatio: NaN
    }
  },

  computed: {
    cropPreview () {
      if (this.fileRaw) {
        return URL.createObjectURL(this.fileRaw)
      }
    }
  },

  methods: {
    onBeforeEncode (file) {
      this.checkEncodeedFile(file)
    },
    checkEncodeedFile (file) {
      const isJPG = file.type === 'image/jpeg'
      const isPNG = file.type === 'image/png'

      return new Promise((resolve, reject) => {
        if (!isJPG && !isPNG) {
          const message = 'Uploaded file should be a .jpg or .png.'

          this.$message({ message, type: 'error' })
          reject(new Error(message))
        }

        resolve(true)
      })
    },
    async onChange (file, fileList) {
      try {
        await this.checkEncodeedFile(file.raw)
        this.fileRaw = file.raw
        this.initCropper()
      } catch (err) {
        console.error(err)
      }
    },
    async onEncode (data) {
      const url = await this.encodeTo64(data)
      this.$emit('encode', url)
      this.showCropDialog = false
      this.fileRaw = ''
    },
    async encodeTo64 (data) {
      // const bucket = new S3({
      //   accessKeyId: process.env.VUE_APP_AWS_S3_ID,
      //   secretAccessKey: process.env.VUE_APP_AWS_S3_KEY,
      //   region: process.env.VUE_APP_AWS_S3_REGION
      // })
      console.log('base64data')

      const name = this.fileRaw.name
      const ext = name.match(/.jpg|.jpeg|.png$/i)[0]
      const date = new Date().toJSON().substr(0, 10)
      const file = `${date}-${guid()}${ext}`
      const key = `upload/${file}`
      // const croppedImage = await this.getCroppedImage()
      console.log('crop')
      // console.log(croppedImage)

      // return new Promise((resolve, reject) => {
      //   bucket.putObject({
      //     Bucket: process.env.VUE_APP_AWS_S3_BASKET,
      //     Key: key,
      //     ContentType: this.fileRaw.type,
      //     Body: croppedImage.blob
      //   }, (err, data) => {
      //     if (err) return reject(err)
      //     const res = process.env.VUE_APP_AWS_S3_URL + '/' + key
      //     return resolve(res)
      //   })
      // })
      return new Promise((resolve, reject) => {
        return resolve(
          // this.getCroppedImage()
          key
        )
      })
    },
    changeAspectRation (aspect) {
      this.aspectRatio = aspect
      this.cropper.setAspectRatio(this.aspectRatio)
    },
    getCroppedImage () {
      return new Promise(resolve => {
        this.cropper.getCroppedCanvas({
          width: this.cropWidth,
          height: this.cropHeight,
          imageSmoothingQuality: 'medium'
        }).toBlob(blob => {
          resolve({
            blob: blob,
            url: URL.createObjectURL(blob)
          })
        }, this.fileRaw.type, this.quality)
      })
    },
    openCropDialog () {
      this.showCropDialog = true
      this.$nextTick(() => {
        this.initCropper()
      })
    },
    initCropper () {
      if (typeof this.cropper === 'object') {
        this.cropper.destroy()
      }
      this.$nextTick(() => {
        this.cropper = new Cropper(this.$refs.cropper, {
          aspectRatio: this.aspect,
          viewMode: 1,
          autoCropArea: 1
        })
      })
    }
  }
}
</script>

<style lang="scss">
@import '../assets/scss/variables.scss';

.crop-preview-wrapper {
   overflow: hidden;
   padding-bottom: 5px;
}
.crop-preview {
  padding: 2px 0;
  max-height: 250px;
  padding-bottom: 20px;
  img {
     max-width: 100%;
  }
  &__placeholder {
    width: 100%;
    height: 200px;
    background-color: #eee;
    border: 1px dashed $color-info;
    border-radius: 3px;
  }
}
.encode-action, .aspect-ratio-buttons {
  text-align: center;
}
.aspect-ratio-buttons {
  margin-bottom: 10px;
}
</style>
