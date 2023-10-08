<template>
  <div class="wrapper">
    <h1>
      {{ paper }} out of {{ bukkit }} servers on
      <select name="version" v-model="selectedVersions" multiple>
        <option v-for="ver in allowedVersions" :value="ver" :key="ver">{{
          ver
        }}</option>
      </select>
      run Paper!<br />
      That's {{ percent }} percent!
    </h1>
    <h2>
      Additionally, {{ combinedForks }} servers run forks of Paper, making up an addittional {{ forkPercent }} percent!
    </h2>
    <h3>
      This leaves Spigot with less than {{ bukkit - paper - combinedForks }} servers and {{ 100 - percent - forkPercent }} percent!
    </h3>
  </div>
</template>

<script>
const paperUrl =
  "https://bstats.org/api/v1/plugins/580/charts/minecraft_version/data";
const bukkitUrl =
  "https://bstats.org/api/v1/plugins/1/charts/minecraftVersion/data";
const forkUrls = ["https://bstats.org/api/v1/plugins/5103/charts/minecraft_version/data"];  

export default {
  name: "App",
  data() {
    return {
      paper: -1,
      bukkit: -1,
      forks: [],
      selectedVersions: ["1.20.2"],
      allowedVersions: [
        "1.20",
        "1.19",
        "1.18",
        "1.17",
        "1.16",
        "1.15",
        "1.14",
        "1.13",
        "1.12",
        "1.20.2",
        "1.20.1",
        "1.19.4",
        "1.19.3",
        "1.19.2",
        "1.19.1",
        "1.18.2",
        "1.18.1",
        "1.17.1",
        "1.16.5",
        "1.16.4",
        "1.16.3",
        "1.16.2",
        "1.16.1",
        "1.15.2",
        "1.15.1",
        "1.14.4",
        "1.14.3",
        "1.14.2",
        "1.14.1",
        "1.13.2",
        "1.13.1",
        "1.12.2",
        "1.12.1"
      ]
    };
  },
  computed: {
    percent() {
      if (this.paper === -1 || this.bukkit === -1) {
        return -1;
      } else {
        return ((this.paper / this.bukkit) * 100).toFixed(2);
      }
    },
    combinedForks() {
        if (this.forks.length === 0) {
          return -1;
        }
        let result = 0;
        this.forks.forEach(f => result += f);
        return result;
    },
    forkPercent() {
      if (this.forks.length === 0 || this.bukkit === -1) {
        return -1;
      } else {
        return ((this.combinedForks / this.bukkit) * 100).toFixed(2);
      }
    }
  },
  watch: {
    selectedVersions() {
      window.history.replaceState(
        null,
        "PaperStats",
        "#" + this.selectedVersions.join(",")
      );
      this.update();
    }
  },
  mounted() {
    if (window.location.hash) {
      this.selectedVersions = window.location.hash.replace("#", "").split(",");
    }
    this.update();
  },
  methods: {
    update() {
      this.getJSON(paperUrl, data => (this.paper = data));
      this.getJSON(bukkitUrl, data => (this.bukkit = data));
      for (let idx = 0; idx < forkUrls.length; idx++) {
        this.getJSON(forkUrls[idx], data => (this.forks[idx] = data));
      }
    },
    getJSON(url, callback) {
      const xhr = new XMLHttpRequest();
      xhr.open("GET", url, true);
      xhr.responseType = "json";
      xhr.onload = () => {
        const status = xhr.status;
        if (status === 200) {
          const data = xhr.response;
          let y = 0;
          for (const key in data) {
            if (Object.prototype.hasOwnProperty.call(data, key)) {
              const foundVersion = data[key].name;
              for (const v of this.selectedVersions) {
                if (foundVersion.startsWith(v)) {
                  y = y + data[key].y;
                  break;
                }
              }
            }
          }
          if (y === 0) {
            console.log("No " + this.selectedVersions + " found?");
            callback(-1);
          } else {
            callback(y);
          }
        } else {
          console.log("Error: ", status);
          console.log(xhr.response);
          callback(-1);
        }
      };
      xhr.send();
    }
  }
};
</script>

<style lang="scss">
html {
  color-scheme: dark light;
}
.wrapper {
  position: fixed;
  top: 50%;
  transform: translate(5%, -70%);
}
h1 {
  font-size: 3.5vw;
}
h2 {
  margin-top: 2em;
  font-size: 2vw;
}
h3 {
  font-size: 2vw;
}
select {
  font-size: 3.5vw;
}
</style>
