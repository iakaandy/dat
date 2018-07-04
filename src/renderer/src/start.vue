<template>
    <div>
        <Form ref="formValidate" :model="formValidate">
            <input placeholder="key" v-model="formValidate.datkey"></input>
            <Button type="button" :loading="loading" size="large" @click="download('formValidate')" long>
                Скачать
            </Button>
            <Button type="button" :loading="loading" size="large" @click="share('formValidate')" long>
                Share
            </Button>
        </Form>
        <br>
        <br>
    </div>
</template>
<script>
  export default {
    data () {
      return {
        formValidate: {
          datkey: ''
        }
      }
    },
    computed: {
      loading () {
        //  return this.$store.state.Auth.status === 'loading'
      }
    },
    methods: {
      download (name) {
        //   this.$Message.success('Success!')
        const {datkey} = this.formValidate
        console.log(datkey)
        console.log('run dat')

        // var fs = require('fs')
        var path = require('path')
        var mirror = require('mirror-folder')
        var ram = require('random-access-memory')
        var Dat = require('dat')
        var dest = path.join(__dirname, 'tmp')
        // fs.mkdirSync(dest)
        Dat(ram, {
          key: datkey,
          sparse: true
        }, function (err, dat) {
          if (err) throw err

          var network = dat.joinNetwork()
          network.once('connection', function () {
            console.log('Connected')
          })

          dat.archive.metadata.update(function () {
            console.log('run download')
            var progress = mirror({fs: dat.archive, name: '/'}, dest, function (err) {
              if (err) throw err
              console.log('Done')
            })
            progress.on('put', function (src) {
              console.log('Downloading', src.name)
            })
          })
          console.log(`Downloading: ${dat.key.toString('hex')}\n`)
        })
      },
      share (name) {
        console.log('share')
        var path = require('path')
        var Dat = require('dat')

        var src = path.join(__dirname, '/src')

        console.log('src path:' + src)

        Dat(src, {temp: true}, function (err, dat) {
          if (err) throw err

          var network = dat.joinNetwork()
          network.once('connection', function () {
            console.log('Connected')
          })
          var progress = dat.importFiles(src, {
            ignore: ['**/dat-node/node_modules/**']
          }, function (err) {
            if (err) throw err
            console.log('Done importing')
            console.log('Archive size:', dat.archive.content.byteLength)
          })
          progress.on('put', function (src, dest) {
            console.log('Added', dest.name)
          })

          console.log(`Sharing: ${dat.key.toString('hex')}\n`)
        })
      }
    }
  }
</script>
