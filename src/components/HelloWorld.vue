<template>
  <div class="hello">
    <div>
      <video ref="video" id="video" width="640" height="480" autoplay></video>
    </div>
    <div>
      <button id="snap" v-on:click="capture()">Snap Photo</button>
    </div>
    <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
    <ul>
      <li v-for="c in captures" :key="c.id">
        <img @click="decode(c)" v-bind:src="c.capture" height="50" />
      </li>
    </ul>
  </div>
</template>

<script>
import Quagga from 'quagga'

export default {
  name: 'HelloWorld',
  data () {
    return {
      captures: [],
      results: [],
      index: 0
    }
  },

  methods: {
    capture() {
      this.canvas = this.$refs.canvas
      // eslint-disable-next-line
      var context = this.canvas.getContext('2d').drawImage(this.video, 0, 0, 640, 480)
      const frame = {
        id: this.index++,
        // eslint-disable-next-line
        capture: canvas.toDataURL('image/jpg')
      }
      this.captures.push(frame)
    },

    decode(image) {
      console.log(image)
      Quagga.decodeSingle(
        {
          decoder: {
            readers: [
              'code_128_reader',
              'code_39_reader',
              'code_39_vin_reader',
              'ean_reader',
              'ean_8_reader',
              'upc_reader',
              'upc_e_reader',
              'codabar_reader'
            ] // List of active readers
          },
          locate: true, // try to locate the barcode in the image
          src: image.capture
          // '/src/assets/big.jpg'
        },
        function(result) {
          if (result.codeResult) {
            console.log('result obj', result)
            console.log('result', result.codeResult.code)
            this.results.push(result.codeResult.code)
          } else {
            console.log('not detected')
          }
        }
      )
    }
  },

  mounted() {
    this.video = this.$refs.video
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
        this.video.srcObject = stream
        this.video.play()
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
