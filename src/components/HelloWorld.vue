<template>
  <b-container fluid>
    <div id="scandit-barcode-picker" style="max-width: 1280px; max-height: 80%;">
    </div>
    <!-- <div>
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
    </ul> -->
    {{results}}
  </b-container>
</template>

<script>
import Quagga from 'quagga'
import axios from 'axios'
import * as ScanditSDK from 'scandit-sdk'

export default {
  name: 'HelloWorld',
  data () {
    return {
      licenseKey: 'ATwrXw9bEw81IyUL7gxS6Q8Db366LWoK1kbtMpBGv+U5KkGCdRw83GET+4UtWSWiw1Xb1JtE/D5MA2Ugf07t/vtZg0p0RQ5e7HViH/hnJp3bbbTrBUSjdo13JoSEcw6fpiFTVUNtbB9eI+Moxx7AlpB9VmPRVVQW0/AXyqv4UXTtlImgW01n6ltseCWEdFpPDk90tyN6Qsf2i0O3e1oibMRR04kKEivWCdBOlufyaNKvQW8PwvJU4lXclfggUrj5yhIXofuIN7TCIaTsCefhwG8BebYsYNEt6ngnTY5LYXemXgq6c6YdveDqfkakR8xFaVdTY0Qi6fZLZgrgjU5NNQ/y9yLwiu/zPaF1Cxc2aNP/Tav4XKd90Ln8Kszw5YOZV3VL0gH8w3hZdRZaP7JMDZEEoMFmTs0zY8EJ/7LLi/Y3I13RrycMEFhpAwdya7In3WWzYiGy/tZmP+xiiEOUZOVWknKfh8Et1Ecljg1heMMe42sF7pYd6MGiinJlbWxSU415xMRNbZ2NWtlr8gxkw5bhdHzrJoHbAnHaiDvr15rTloAJfh97bukyBze5R8gS8osDR+xILAx+2jZATc+D0cJArnNQr145Gb+groXbRY8boTHqO2r26UWXmW1BEvQUDbyu2Um+pzHZT11sgFJTTqSbhlkdcjTYS4Y329a3K/u8SfpDOsGJi7AmpU2SikoUW53fyrUIMc/BaZT+QRVVV9pkIo+kxf+VVplKLLOv0rawti96qEALwL00e9D+DghVw01PSrrSr8i8EHbOf+eP3wDlBDEx/NdgJgLsau6wLQVa8Aje2NK0JJFx2OntDw==',
      captures: [],
      results: [],
      index: 0
    }
  },

  methods: {
    capture() {
      this.canvas = this.$refs.canvas
      // eslint-disable-next-line
      var context = this.canvas.getContext('2d').drawImage(this.video, 0, 0, this.canvas.width, this.canvas.height)
      const frame = {
        id: this.index++,
        // eslint-disable-next-line
        capture: canvas.toDataURL('image/jpg')
      }
      this.captures.push(frame)

      console.log(frame.capture)
      axios
        // .post(`https://barcode-scanner-prototype.herokuapp.com/process/`, frame.capture)
        .post(`http://127.0.0.1:8888/process`, frame.capture)
        .then(response => {
          console.log(response.data)
        })
        .catch(error => {
          console.log(error)
        })
    },

    decode(image) {
      let martelada
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
          if (result === undefined) {
            console.log('result undefined')
            martelada = 'undefined'
            // this.results.push('undefined')
          }
          if (result.codeResult) {
            console.log('result obj', result)
            console.log('result', result.codeResult.code)
            // this.results.push({id: this.index, code: result.codeResult.code})
            martelada = result.codeResult.code
            // this.results.push(result.codeResult.code)
          } else {
            console.log('not detected')
          }
        }
      )
      this.results.push(martelada)
    }
  },

  created() {
    ScanditSDK.configure(this.licenseKey, {
      engineLocation: 'https://unpkg.com/scandit-sdk/build'
      // preloadEngineLibrary: true
    })
  },

  mounted() {
    ScanditSDK.BarcodePicker.create(document.getElementById('scandit-barcode-picker'), {
      playSoundOnScan: true
    }).then(function(barcodePicker) {
      const scanSettings = new ScanditSDK.ScanSettings({
        enabledSymbologies: ['ean13'],
        codeDuplicateFilter: 1000
      })
      barcodePicker.applyScanSettings(scanSettings)

      barcodePicker.onScan(function(scanResult) {
        alert(scanResult.barcodes.reduce(function(string, barcode) {
          return string + ScanditSDK.Barcode.Symbology.toHumanizedName(barcode.symbology) + ': ' + barcode.data + '\n'
        }))
      })
    })

    // this.video = this.$refs.video
    // if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
    //   console.log(navigator.mediaDevices)
    //   navigator.mediaDevices
    //     .getUserMedia({
    //       // video: {
    //       //   facingMode: { exact: 'environment' }
    //       // }
    //       video: true
    //     })
    //     .then(stream => {
    //       this.video.srcObject = stream
    //       this.video.play()
    //     })
    // }
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
