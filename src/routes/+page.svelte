<script lang="ts">
  const paperUrl = "https://bstats.org/api/v1/plugins/580/charts/minecraft_version/data";
  const bukkitUrl = "https://bstats.org/api/v1/plugins/1/charts/minecraftVersion/data";
  const forkUrls = [
    "https://bstats.org/api/v1/plugins/5103/charts/minecraft_version/data",
    "https://bstats.org/api/v1/plugins/18084/charts/minecraft_version/data",
  ];

  const defaultVersion = "1.21.7";
  const allowedVersions = [
    "1.21",
    "1.20",
    "1.19",
    "1.18",
    "1.17",
    "1.16",
    "1.15",
    "1.14",
    "1.13",
    "1.12",
    "1.21.7",
    "1.21.6",
    "1.21.5",
    "1.21.4",
    "1.21.3",
    "1.21.2",
    "1.21.1",
    "1.20.6",
    "1.20.5",
    "1.20.4",
    "1.20.3",
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
    "1.12.1",
  ];

  let paper = $state(-1);
  let bukkit = $state(-1);
  let forks = $state([] as number[]);
  let selectedVersions = $state([typeof window !== "undefined" && window.location.hash ? window.location.hash.replace("#", "") : defaultVersion]);

  let percent = $derived.by(() => {
    if (paper === -1 || bukkit === -1) {
      return -1;
    } else {
      return (paper / bukkit) * 100;
    }
  });

  let combinedForks = $derived.by(() => {
    if (forks.length === 0) {
      return -1;
    } else {
      return forks.reduce((a, b) => a + b, 0);
    }
  });

  let forkPercent = $derived.by(() => {
    if (combinedForks === -1 || bukkit === -1) {
      return -1;
    } else {
      return (combinedForks / bukkit) * 100;
    }
  });

  let legacyCount = $derived(bukkit - paper - combinedForks);
  let legacyPercent = $derived(100 - percent - forkPercent);
  $effect(() => {
    if (typeof window !== "undefined") window.history.replaceState(null, "PaperStats", "#" + selectedVersions.join(","));
    update();
  });

  async function update() {
    await load(paperUrl, (data) => (paper = data));
    await load(bukkitUrl, (data) => (bukkit = data));
    forks = await Promise.all(forkUrls.map((url) => new Promise<number>((resolve) => load(url, resolve))));
  }

  async function load(url: string, callback: (data: number) => void) {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        console.error(`Response not ok: ${response.status}: ${await response.text()}`);
        callback(-1);
        return;
      }

      const data = (await response.json()) as { name: string; y: number }[];
      let y = 0;
      for (const version of data) {
        for (const v of selectedVersions) {
          if (version.name.startsWith(v) && !version.name.includes("v") && !version.name.includes("git")) {
            y += version.y;
            break;
          }
        }
      }
      if (y === 0) {
        console.log("No " + selectedVersions + " found?");
        callback(-1);
      } else {
        callback(y);
      }
    } catch (e) {
      console.error("Error: ", e);
      callback(-1);
    }
  }
</script>

<main class="wrapper">
  <h1>
    {paper} out of {bukkit} servers on
    <select name="version" bind:value={selectedVersions} multiple title="Version Selection">
      {#each allowedVersions as ver (ver)}
        <option value={ver}>{ver}</option>
      {/each}
    </select>
    run Paper!<br />
    That's {percent.toFixed(2)} percent!
  </h1>
  {#if combinedForks === -1 && forkPercent === -1}
    <h2>There are no servers running popular forks of Paper on this version.</h2>
  {:else}
    <h2>
      Additionally, {combinedForks} servers run popular forks of Paper, making up an additional {forkPercent.toFixed(2)}
      percent!
    </h2>
  {/if}
  <h3>
    This leaves Spigot with less than {legacyCount} servers and {legacyPercent.toFixed(2)}
    percent!
  </h3>
</main>

<style>
  :global(html) {
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
