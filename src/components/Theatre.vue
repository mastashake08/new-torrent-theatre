<template>
  <v-container fluid grid-list-md>
      <!-- xs = 600px full screen (12) -->
      <!-- md = 600px or more. half of the screen (6) -->
      <v-layout row wrap>
          <v-flex xs12 md6 v-for="m in movies" :key="m.id">
              <v-card v-if="m.description_full" :loading="loading">
                  <v-card-title>{{m.title}}</v-card-title>
                  <v-card-subtitle>{{m.genres[0]}} | Runtime - {{m.runtime}} minutes</v-card-subtitle>
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

                      <div class="grey--text ml-4">
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
                    <v-btn color="green">Download</v-btn>
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
      movie: {},
      client: {},
      loading: true
    }),
    computed: {
      magnetUrl : function () {
        const title = encodeURIComponent(this.movie.title)
        return `magnet:?xt=urn:btih:${this.movie.torrents[0].hash}&dn=${title}&tr=wss://tracker.btorrent.xyz&tr=wss://tracker.openwebtorrent.com&tr=wss://tracker.webtorrent.io&tr=wss://tracker.fastcast.nz`
      }
    },
    created () {
      var WebTorrent = require('webtorrent')
      this.client = new WebTorrent()
      fetch('https://yts.mx/api/v2/list_movies.json')
      .then(response => response.json())
      .then(data => {
        this.movies = data.data.movies
        this.movie = this.movies[0]
        this.loading = false
      });

    },
    mounted () {},
    methods: {
      play: function (movie){
        this.movie = movie
        this.getTorrent()
      },
      getTorrent: function () {
        this.client.add(this.magnetUrl, function (torrent) {
          torrent.on('metadata', function () {
            console.log(torrent)
          })
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
