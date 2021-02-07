<template>
  <v-container fluid grid-list-md>
      <!-- xs = 600px full screen (12) -->
      <!-- md = 600px or more. half of the screen (6) -->
      <v-text-field v-model="search" label="Movie search" v-on:keyup="searchMovies"></v-text-field>

      <v-text-field v-model="magnet" label="Magnet URL"></v-text-field>
      <v-btn v-on:click="getManualTorret">Play</v-btn>
      <div id="movie"></div>
      <v-layout row wrap>
          <v-flex xs12 md6 v-for="m in movies" :key="m.id">
              <v-card v-if="m.description_full" :loading="loading">
                  <v-card-title>{{m.title}} - {{m.year}}</v-card-title>
                  <v-card-subtitle>{{m.genres[0]}} | Runtime - {{m.runtime}} minutes  <span v-if="m.mpa_rating !=''">| Rated {{m.mpa_rating}}</span> | {{m.torrents[0].seeds}} Seeds</v-card-subtitle>
                  <v-img :src="m.medium_cover_image" height="300px"></v-img>
                  <v-card-text>
                    <v-row
                      align="center"
                      class="mx-0"
                    >
                      <v-rating
                        :value="m.rating/2.0"
                        color="amber"
                        dense
                        half-increments
                        readonly
                        size="14"
                      ></v-rating>

                      <div class="grey--text ml-4" >
                        {{m.rating}}/10
                      </div>
                    </v-row>
                    <v-row
                      align="center"
                      class="mx-0"
                    >
                    {{m.description_full}}
                  </v-row>
                </v-card-text>
                  <v-card-actions>
                    <v-btn v-on:click="play(m)">Play</v-btn>
                  </v-card-actions>
              </v-card>
          </v-flex>
      </v-layout>
  </v-container>
</template>

<script>
  export default {
    name: 'Theatre',
    data: () => ({
      movies: [],
      movie: null,
      search: '',
      client: {},
      loading: true,
      magnet: '',
      trackers: '&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.dler.org%3A6969%2Fannounce&tr=udp%3A%2F%2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=udp%3A%2F%2F47.ip-51-68-199.eu%3A6969%2Fannounce&tr=wss%3A%2F%2Ftracker.btorrent.xyz&tr=wss%3A%2F%2Ftracker.openwebtorrent.com',
      wttr: '&tr=udp%3A%2F%2Fexplodie.org%3A6969&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.empire-js.us%3A1337&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337&tr=wss%3A%2F%2Ftracker.btorrent.xyz&tr=wss%3A%2F%2Ftracker.fastcast.nz&tr=wss%3A%2F%2Ftracker.openwebtorrent.com&ws=https%3A%2F%2Fwebtorrent.io%2Ftorrents%2F'
    }),
    computed: {
      magnetUrl : function () {
        if (this.movie != null) {
          const title = encodeURIComponent(this.movie.title)
          return `magnet:?xt=urn:btih:${this.movie.torrents[0].hash}&dn=${title}${this.trackers}`
        }
        else {
          return ''
        }
      },
      manualMagnetUrl : function () {
        return this.magnet+this.wttr+encodeURI(this.trackers)
      }
    },
    created () {
      var WebTorrent = require('webtorrent')
      this.client = new WebTorrent()
      this.client.on('error', function (err) {console.log(err)})

      fetch('https://yts.mx/api/v2/list_movies.json?order_by=seeds')
      .then(response => response.json())
      .then(data => {
        this.movies = data.data.movies
        this.loading = false
      });

    },
    mounted () {},
    methods: {
      searchMovies: function () {
        fetch('https://yts.mx/api/v2/list_movies.json?query_term='+encodeURIComponent(this.search))
        .then(response => response.json())
        .then(data => {
          this.movies = data.data.movies
          this.loading = false
        });
      },
      play: function (movie){
        this.movie = movie
        this.getTorrent()
      },
      getManualTorret: function () {
        this.client.add(this.manualMagnetUrl, function (torrent) {
          torrent.on('infoHash', function () {console.log('infohash')})
          console.log(torrent)
          torrent.on('metadata', function () {
            console.log(torrent)
          })
          torrent.on('download', function (bytes) {
            console.log('just downloaded: ' + bytes)
            console.log('total downloaded: ' + torrent.downloaded)
            console.log('download speed: ' + torrent.downloadSpeed)
            console.log('progress: ' + torrent.progress)
          })
          torrent.on('error', function (err) { console.log(err) })
          // Torrents can contain many files. Let's use the .mp4 file
          var file = torrent.files.find(function (file) {
            return file.name.endsWith('.mp4')
          })

          // Display the file by adding it to the DOM. Supports video, audio, image, etc. files
          file.appendTo('#movie')
        })
      },
      getTorrent: function () {
        this.client.add(this.magnetUrl, function (torrent) {
          console.table(torrent)
          torrent.on('infoHash', function () {console.log('infohash')})
          console.log(torrent)
          torrent.on('metadata', function () {
            console.log(torrent)
          })
          torrent.on('download', function (bytes) {
            console.log('just downloaded: ' + bytes)
            console.log('total downloaded: ' + torrent.downloaded)
            console.log('download speed: ' + torrent.downloadSpeed)
            console.log('progress: ' + torrent.progress)
          })
          torrent.on('error', function (err) { console.log(err) })
          // Torrents can contain many files. Let's use the .mp4 file
          var file = torrent.files.find(function (file) {
            return file.name.endsWith('.mp4')
          })

          // Display the file by adding it to the DOM. Supports video, audio, image, etc. files
          file.appendTo('body')
        })
      }
    }
  }
</script>
