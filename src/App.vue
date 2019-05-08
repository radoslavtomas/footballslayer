<template>
  <div id="app">
    <div class="container">
      <h1 class="main-title">
        Football
        <br>Slayer
      </h1>

      <Search @teamSelected="slayItLikeABrut"/>

      <section v-if="matchesResult">
        <div v-if="team" class="team-card">
          <small>Football Team</small>
          <h3>{{ team.name }}</h3>
          <p>--------------------------------</p>
        </div>

        <div v-if="homeMatches.length !== 0">
          <article v-for="match in homeMatches" :key="match.id">
            <h2>
              <span class="avoid-sign">avoid</span>
              {{ match.utcDate | date }}
            </h2>
            <p>{{ match.homeTeam.name }} vs. {{ match.awayTeam.name }}</p>
          </article>
        </div>
        <div v-else>
          <h2 class="all-clear-sign">YOU SLAYED IT</h2>
          <h2 class="all-clear-sign">-</h2>
          <h2 class="all-clear-sign">the air is clear</h2>
          <h2 class="all-clear-sign">for next few days</h2>
        </div>
      </section>

      <section class="mapWrapper" :class="{ active:mapActive }">
        <div id="map"></div>
      </section>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Search from "./components/Search";
import mapboxgl from "mapbox-gl";
import MapboxTraffic from "@mapbox/mapbox-gl-traffic";

export default {
  name: "app",
  components: {
    Search
  },
  data() {
    return {
      team: null,
      matchesResult: false,
      mapActive: false,
      matches: []
    };
  },
  methods: {
    slayItLikeABrut(team) {
      this.team = team;
      this.getMatches();
      this.getMap();
    },
    getMatches() {
      axios({
        method: "get",
        url: `https://api.football-data.org/v2/teams/${
          this.team.id
        }/matches?status=SCHEDULED`,
        headers: { "X-Auth-Token": process.env.VUE_APP_TOKEN }
      })
        .then(response => {
          this.matchesResult = true;
          this.matches = response.data.matches;
        })
        .catch(error => {
          conosle.log(error);
        });
    },
    getMap() {
      axios
        .get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${
            this.team.address
          }.json?access_token=${process.env.VUE_APP_MAPTOKEN}`
        )
        .then(response => {
          this.mapActive = true;
          mapboxgl.accessToken =
            "pk.eyJ1IjoicmFkb3NsYXZ0b21hcyIsImEiOiJjanZkeGNqY2QwZjlkNDFsYzFqNGlvYW85In0.mc8Nj4zfAoOW_ZBxspAo5g";
          const map = new mapboxgl.Map({
            container: "map", // container id
            style: "mapbox://styles/mapbox/streets-v10", // stylesheet location
            center: response.data.features[0].center, // starting position [lng, lat]
            zoom: 13 // starting zoom
          });
          map.addControl(
            new MapboxTraffic({
              showTraffic: true,
              showTrafficButton: false
            })
          );
        });
    }
  },
  computed: {
    homeMatches() {
      if (this.matches) {
        return this.matches.filter(match => {
          return match.homeTeam.name === this.team.name;
        });
      } else {
        return [];
      }
    }
  },
  filters: {
    date(val) {
      const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "numeric",
        minute: "numeric"
      };
      const date = new Date(val);
      const dateFormat = new Intl.DateTimeFormat("en-GB", options);

      return dateFormat.format(date);
    }
  }
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css?family=Rock+Salt|Rubik:400,700");

body {
  color: #404040;
  font-family: "Rubik", sans-serif;
  background-image: linear-gradient(
    to right bottom,
    #cacdcb,
    #d6d9d9,
    #e4e6e6,
    #f2f2f3,
    #ffffff
  );
  min-height: 100vh;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
  font-family: "Rubik", sans-serif;

  .main-title {
    color: #d40000;
    font-size: calc(2em + 0.4vw);
    font-family: "Rock Salt", cursive;
    margin: 0;
    padding-top: 1em;
  }

  .team-card {
    margin-top: 0.5em;
    h3 {
      margin-top: 0;
    }
  }

  .avoid-sign {
    color: #d40000;
    font-family: "Rock Salt", cursive;
    text-transform: uppercase;
    position: relative;
    margin-right: 10px;
  }

  .all-clear-sign {
    color: #23ab5a;
    font-family: "Rock Salt", cursive;
  }
  .mapWrapper {
    max-width: 800px;
    height: 250px;
    margin: 0 auto;
    position: relative;
    visibility: hidden;

    &.active {
      visibility: initial;
    }

    #map {
      position: relative;
      margin: 0 auto;
      border: 2px solid #d40000;
      border-radius: 0.25em;
      max-width: 97%;
      height: 97%;

      .mapboxgl-canvas-container {
        position: absolute;
        top: 0;
        left: 0;
      }
    }
  }
}
</style>
