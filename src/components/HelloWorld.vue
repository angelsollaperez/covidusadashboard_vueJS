<template>
  <div>
    <select v-model="selectedState">
      <option v-bind:value="null"></option>
      <option
        v-for="state in states"
        v-bind:key="state.id"
        v-bind:value="state.id"
      >
        {{ state.name }}
      </option>
    </select>

    <div v-if="data[selectedState]">
      <ul>
        <li>
          Total deaths:
          {{ data[selectedState].death | noInfo }}
        </li>
        <li>
          Current postive people:
          {{ data[selectedState].positive | noInfo }}
        </li>
        <li>
          Total hospitalized people:
          {{ data[selectedState].hospitalized | noInfo }}
        </li>
        <li>
          Currently hospitalized people:
          {{ data[selectedState].hospitalizedCurrently | noInfo }}
        </li>
        <li>
          Date of the information:
          {{ data[selectedState].date | noInfo }}
        </li>
      </ul>
    </div>
    <div v-else>Please select a state</div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      selectedState: null,
      states: [],
      data: [],
    };
  },
  methods: {
    async fetchUrl(url) {
      let res = await fetch(url);

      if (res.status !== 200) {
        return null;
      }

      return await res.json();
    },
    async fetchStates() {
      const res = await this.fetchUrl(
        "https://api.covidtracking.com/v1/states/info.json"
      );

      if (res === null) {
        this.states = null;
        return;
      }

      this.states = res.map((s) => {
        return { name: s.name, id: s.state };
      });
    },
    async fetchData() {
      let res = await this.fetchUrl(
        "https://api.covidtracking.com/v1/states/current.json"
      );

      if (res === null) {
        this.data = null;
        return;
      }

      let tmpData = {};

      res.forEach((d) => {
        tmpData[d.state] = {
          death: d.death,
          positive: d.positive,
          hospitalized: d.hospitalized,
          hospitalizedCurrently: d.hospitalizedCurrently,
          date: d.lastUpdateEt,
        };

        this.data = tmpData;
      });
    },
  },
  filters: {
    noInfo(value) {
      return value || "No information available";
    },
  },
  created() {
    this.fetchStates();
    this.fetchData();
  },
};
</script>

<style scoped>
</style>
