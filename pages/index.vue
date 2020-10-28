<template>
  <div>
    <b-modal
      ref="detailMovie"
      class="detail-movie-modal"
      size="lg"
      hide-footer
      scrollable
      centered>
      <b-row>
        <b-col
          md="4">
          <img
            v-if="detail_movie.Poster !== 'N/A'"
            class="detail-poster"
            :src="detail_movie ? detail_movie.Poster : ''">
          <div
            v-else
            class="detail-no-poster no-poster">
            <p class="no-poster-title">No Poster</p>
          </div>
        </b-col>
        <b-col
          md="8">
          <h4 class="uppercase detail-title">{{ detail_movie ? detail_movie.Title : '' }}</h4>
          <p class="sub-title uppercase">
            {{ detail_movie ? detail_movie.Year : '' }} | 
            {{ detail_movie ? detail_movie.Rated : '' }} | 
            {{ detail_movie ? detail_movie.Released : '' }} | 
            {{ detail_movie ? detail_movie.Runtime : '' }} | 
            {{ detail_movie ? detail_movie.Genre : '' }} | 
            {{ detail_movie ? detail_movie.Language : '' }}
          </p>
          <br>
          <p class="uppercase detail-text">
            {{ detail_movie ? detail_movie.Plot : '' }}
          </p>
          <br>
          <p class="uppercase detail-text">
            Actor: {{ detail_movie ? detail_movie.Actors : '' }}
          </p>
          <p class="uppercase detail-text">
            Director: {{ detail_movie ? detail_movie.Director : '' }}
          </p>
          <p class="uppercase detail-text">
            Writer: {{ detail_movie ? detail_movie.Writer : '' }}
          </p>
        </b-col>
      </b-row>
    </b-modal>
    <div
      class="cover"
      :style="`
      background-image: url('${cover_film ? cover_film.Poster : ''}');
      background-repeat: no-repeat;
      background-size: cover`">
      <div
        class="foreground">
        <div class="movie-container">
          <h3 class="app-name">
            NETPLIK
          </h3>
          <br>
          <br>
          <h4
            v-if="cover_film"
            class="title">
            {{ cover_film ? cover_film.Title : '' }}
          </h4>
          <br>
          <h4
            v-if="cover_film"
            class="detail-movie"
            @click="detailMovie(cover_film.imdbID)">
            DETAIL MOVIE
          </h4>
        </div>
        <div class="movie-controller">
          <div class="float-left">
            <b-row>
              <div style="color: white; padding-top: 4px; margin-left: 15px">
                TRENDING MOVIE
              </div>
              <div style="margin-left: 15px">
                <input
                  v-model="movie_name"
                  class="movie-search-input"
                  type="text"
                  placeholder="search movie here"
                  style="margin-top: 2px">
              </div>
              <div style="margin-left: 5px">
                <input
                  type="button"
                  value="search"
                  @click="searchMovie">
              </div>
            </b-row>
          </div>
          <div
            v-if="length_film > 10"
            class="float-right">
            <div class="nav-buttons">
              <input
                :disabled="page === 1"
                :class="`nav-button${page === 1 ? ' disabled-nav-button' : ''}`"
                type="button"
                value="<"
                @click="previousRow">
              <input
                :disabled="(page * 10) - length_film >= 0"
                :class="`nav-button${(page * 10) - length_film >= 0 ? ' disabled-nav-button' : ''}`"
                type="button"
                value=">"
                @click="nextRow">
            </div>
          </div>
        </div>
        <div
          v-if="length_film > 0"
          class="movie-rows">
          <div class="movie-list-container">
            <b-row>
              <b-col  
                v-for="(item, index) in film_row_1"
                :key="index">
                <center class="movie-poster-container">
                  <img
                    v-if="item.Poster !== 'N/A'"
                    :src="item.Poster"
                    class="movie-poster"
                    @click="setCover(item)"/>
                  <div
                    v-else
                    class="movie-poster no-poster"
                    @click="setCover(item)">
                    <p class="no-poster-title">{{ item.Title }}</p>
                  </div>
                </center>
              </b-col>
            </b-row>
          </div>
          <br>
          <div class="movie-list-container">
            <b-row>
              <b-col  
                v-for="(item, index) in film_row_2"
                :key="index">
                <center class="movie-poster-container">
                  <img
                    v-if="item.Poster !== 'N/A'"
                    :src="item.Poster"
                    :alt="`${item.Title}'s Poster`"
                    class="movie-poster"
                    @click="setCover(item)"/>
                  <div
                    v-else
                    class="movie-poster no-poster"
                    @click="setCover(item)">
                    <p class="no-poster-title">{{ item.Title }}</p>
                  </div>
                </center>
              </b-col>
            </b-row>
          </div>
        </div>
        <div
          v-else
          class="movie-rows">
          <center style="color: gainsboro">{{ message }}</center>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      movie_name: null,
      page: 1,
      film_row_1: [],
      film_row_2: [],
      message: null,
      length_film: 0,
      cover_film: null,
      detail_movie: {},
      base: 'http://www.omdbapi.com/?',
      apikey: '3a73f7bf',
    };
  },
  watch: {
    movie_name(val) {
      if (val === null || val === '') {
        this.searchMovie();
      }
    },
  },
  created() {
    this.detail_movie = {};
    this.film_row_1 = [];
    this.film_row_2 = [];
    this.cover_film = null;
    this.message = null;
  },
  methods: {
    getCover() {
      const title = 'Toy Story';
      const year = 2019;
      axios.get(`${this.base}t=${title}&y=${year}&apikey=${this.apikey}`)
        .then((res) => {
          this.cover_film = res.data;
        });
    },
    getFilm() {
      axios.get(`${this.base}s=${this.movie_name}&page=${this.page}&apikey=${this.apikey}`)
        .then((res) => {
          this.film_row_1 = [];
          this.film_row_2 = [];
          if (res.data.Error) {
            this.message = this.movie_name === '' || this.movie_name === null ? null : res.data.Error;
          } else {
            res.data.Search.forEach((element, index) => {
              if (index < 5) {
                this.film_row_1.push(element);
              } else {
                this.film_row_2.push(element);
              }
            });
            this.length_film = res.data.totalResults;
          }
        });
    },
    detailMovie(id) {
      axios.get(`${this.base}i=${id}&apikey=${this.apikey}`)
        .then((res) => {
          this.detail_movie = res.data;
          this.$nextTick(() => {
            this.$refs.detailMovie.show();
          });
        });
    },
    searchMovie() {
      this.length_film = 0;
      this.film_row_1 = [];
      this.film_row_2 = [];
      this.message = 'Loading....';
      this.page = 1;
      this.getFilm();
    },
    previousRow() {
      if (this.page > 1) {
        this.page -= 1;
        this.getFilm();
      }
    },
    nextRow() {
      if ((this.page * 10) - this.length_film < 0) {
        this.page += 1;
        this.getFilm();
      }
    },
    setCover(data) {
      this.cover_film = data;
    },
  },
}
</script>

<style>

body {
  background-color: #232323;
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
}

.app-name {
  display: block;
  font-weight: 400;
  font-size: 27px;
  color: #e20000;
  letter-spacing: 2px;
}

.cover {
  height: 100vh;
}

.foreground {
  padding: 10px 30px 10px 30px;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.65);
}

.title {
  font-size: 40px;
  font-weight: 700;
  letter-spacing: 5px;
  color: #efefef;
  width: 50vw;
}

.detail-movie {
  color: red;
  font-weight: 400;
  font-size: 20px;
  font-style: italic;
  transition: 0.2s;
  cursor: pointer;
}

.detail-movie:hover {
  text-decoration: underline;
}

.modal-header {
  padding: 0 5px 0 5px;
  border: 0;
}

.sub-title {
  color: #8a8a8a;
}

.uppercase {
  text-transform: uppercase;
}

.float-left {
  float: left;
}

.float-right {
  float: right;
}

.detail-poster {
  margin-top: 9px;
  width: 100%;
}

.detail-no-poster {
  margin-top: 9px;
  width: 100%;
  height: 225px;
  line-height: 225px;
}

.detail-title {
  font-weight: 800;
  font-size: 30px;
  letter-spacing: 3px;
}

.detail-text {
  font-weight: 500;
}

.movie-container {
  height: 40vh;
}

.movie-controller {
  height: 50px;
}

.movie-rows {
  height: calc(60vh - 60px);
  overflow-y: scroll;
  overflow-x: hidden;
}

.movie-rows::-webkit-scrollbar {
  display: none;
}

.movie-rows {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.movie-search-input {
  background-color: rgba(0, 0, 0, 0);
  border: 0;
  border-bottom: 1px solid white;
  color: white;
}

.movie-poster-container {
  height: 250px;
}

.movie-poster {
  width: 100%;
  height: 225px;
  object-fit: cover;
  cursor: pointer;
}

.no-poster {
  background-color: black;
  line-height: 225px;
  text-align: center;
}

.no-poster-title {
  color: white;
  line-height: 1.5;
  display: inline-block;
  vertical-align: middle;
}

.nav-button {
  background-color: black;
  color: #e20000;
  border: 1px solid black;
  font-weight: 500;
}

.disabled-nav-button {
  cursor: not-allowed;
  background-color: #070707;
  color: gainsboro;
  border: 1px solid #070707;
  font-weight: 500;
}

.links {
  padding-top: 15px;
}
</style>
