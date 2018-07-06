<template>
    <div>
        <Form ref="formValidate" :model="formValidate">
            <div>
                <input placeholder="key" v-model="formValidate.datkey"></input>
                <Button type="button" :loading="loading" size="large" @click="download('formValidate')" long>
                    Скачать
                </Button>
            </div>
            <div style="font-family: Arial; font-size: 8pt" v-html="formValidate.load_result"></div>
            <br>
            <br>

            <div>
                <Button type="button" size="large" @click="openpath()" long>
                    Выбрать директорию
                </Button>
                <Button type="button" :loading="loading" size="large" @click="share()" long>
                    Share
                </Button>
            </div>
            <div style="font-family: Arial; font-size: 8pt" v-html="formValidate.share_result"></div>
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
          datkey: '',
          srcpath: '',
          load_result: '',
          share_result: ''
        }
      }
    },
    computed: {
      loading () {
        //  return this.$store.state.Auth.status === 'loading'
      }
    },
    methods: {
      openpath () {
        const {dialog} = require('electron').remote
        this.formValidate.srcpath = dialog.showOpenDialog({properties: ['openDirectory']})[0]
        this.formValidate.share_result = 'Выбрана директория: ' + this.formValidate.srcpath + '<br>'
      },
      download () {
        //   this.$Message.success('Success!')
        const {datkey} = this.formValidate
        console.log(datkey)
        console.log('run dat')

        // var fs = require('fs')
        var path = require('path')
        var mirror = require('mirror-folder')
        var ram = require('random-access-memory')
        var Dat = require('dat')
        var dest = path.join('/', 'tmp')
        // fs.mkdirSync(dest)
        const data = this.formValidate
        data.load_result = ''
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
              data.load_result += 'Done'
            })
            progress.on('put', function (src) {
              data.load_result += 'Downloading ' + src.name + '<br>'
            })
          })
          data.load_result += 'Downloading: ' + dat.key.toString('hex') + '<br>'
        })
      },
      share () {
        const {srcpath} = this.formValidate

        console.log('share')
        // var path = require('path')
        var Dat = require('dat')

        console.log(srcpath)
        var src = this.formValidate.srcpath

        console.log('src path:' + src)

        const data = this.formValidate
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
            data.share_result += 'Added' + dest.name + '<br>'
          })

          data.share_result += 'Sharing key: ' + dat.key.toString('hex') + '<br>'
        })
      }
    }
  }
</script>
