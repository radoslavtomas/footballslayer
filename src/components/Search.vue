<template>
  <div class="search-wrap">
    <h5>Choose a stadium</h5>
    <select v-model="selectedTeam" @change="teamSelected">
      <option value>Please choose</option>
      <option v-for="team in teams" :key="team.id" :value="team.id">{{ team.venue }}</option>
    </select>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "Search",
  data() {
    return {
      teams: [],
      selectedTeam: ""
    };
  },
  methods: {
    getPremierLeague() {
      return axios({
        method: "get",
        url: "https://api.football-data.org/v2/competitions/2021/teams",
        headers: { "X-Auth-Token": process.env.VUE_APP_TOKEN }
      });
    },
    getChampionship() {
      return axios({
        method: "get",
        url: "https://api.football-data.org/v2/competitions/2016/teams",
        headers: { "X-Auth-Token": process.env.VUE_APP_TOKEN }
      });
    },
    teamSelected() {
      const selectedTeam = this.teams.find(team => {
        return team.id === this.selectedTeam;
      });
      this.$emit("teamSelected", selectedTeam);
    }
  },
  beforeMount() {
    axios
      .all([this.getPremierLeague(), this.getChampionship()])
      .then(
        axios.spread((p, ch) => {
          const premier = p.data.teams;
          const champion = ch.data.teams;

          let merged = premier.concat(champion).sort((a, b) => {
            let nameA = a.venue.toUpperCase();
            let nameB = b.venue.toUpperCase();

            if (nameA < nameB) {
              return -1;
            }
            if (nameA > nameB) {
              return 1;
            }

            return 0;
          });

          this.teams = merged;
        })
      )
      .catch(error => {
        console.log(error);
      });
  }
};
</script>

<style lang="scss" scoped>
.search-wrap {
  max-width: 250px;
  padding: 10px;
  margin: 0 auto;

  select {
    width: 100%;
    padding: 10px;
    border: 1px solid #999999;
    border-radius: 0.25em;
    background-color: #f3f3f3;
    outline: none;
  }
}
</style>