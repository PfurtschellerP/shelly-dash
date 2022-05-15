<template>
  <div>
    <h3>{{ data.name }}</h3>
    <h4>Daten</h4>
    <h4 id="updateBanner" v-if="updateAvailable">Update verfügbar!!!</h4>
    <img :src="imagePath" />
    <ul>
      <li>
        Hostname:
        <a :href="'http://' + data.host" target="_blank">{{ data.host }}</a>
      </li>
      <li>
        IP:
        <a :href="'http://' + data.wifi_ip" target="_blank">{{
          data.wifi_ip
        }}</a>
      </li>
      <li>Firmware Build: {{ data.fw_build }}</li>
      <li>Firmware Version: {{ data.version }}</li>
      <li>Modell: {{ data.model }}</li>
      <li>Device ID: {{ data.device_id }}</li>
      <li>Uptime: {{ uptime }}</li>
    </ul>
    <!-- <h4>Debug</h4>
    <pre><code>{{ data }}</code></pre> -->
  </div>
</template>

<script setup>
  import { computed, ref } from 'vue';
  import axios from 'axios';

  const props = defineProps({
    shellyIP: String,
  });

  const data = ref({});

  const updateAvailable = ref(false);

  // --------------------------------------------------------------------------
  // Websocket Connection
  // --------------------------------------------------------------------------

  // create Websocket
  const socket = new WebSocket(`ws://${props.shellyIP}/rpc`);

  socket.onopen = (e) => {
    // console.log(`[${props.shellyIP}] Connection established`);
    // fetch data
    socket.send(
      JSON.stringify({ id: 3347, method: 'Shelly.GetInfoExt', params: [] })
    );

    // refresh every 60 seconds
    setInterval(() => {
      socket.send(
        JSON.stringify({ id: 3347, method: 'Shelly.GetInfoExt', params: [] })
      );
    }, 60000);
  };

  socket.onmessage = async (event) => {
    // console.log(`[${props.shellyIP}] Data received from server`);
    data.value = JSON.parse(event.data).result;

    await checkForUpdate(data.value.version, data.value.model);
  };

  socket.onclose = (event) => {
    if (event.wasClean) {
      console.log(
        `[${props.shellyIP}] Connection closed cleanly, code=${event.code} reason=${event.reason}`
      );
    } else {
      // e.g. server process killed or network down
      // event.code is usually 1006 in this case
      console.log('[${props.shellyIP}] Connection died');
    }
  };

  socket.onerror = function (error) {
    console.log(`[${props.shellyIP}] ${error.message}`);
  };

  // --------------------------------------------------------------------------
  // Computed Properties
  // --------------------------------------------------------------------------

  const uptime = computed(() => {
    // pretty complicated and bloated time calculation
    const days = data.value.uptime / 60 / 60 / 24;
    const hours = (days - Math.floor(days)) * 24;
    const minutes = (hours - Math.floor(hours)) * 60;

    return `${Math.floor(days)} Tage, ${Math.floor(
      hours
    )} Stunden und ${Math.floor(minutes)} Minuten`;
  });

  const imagePath = computed(() => {
    return data.value.model == 'Shelly25'
      ? '/img/shelly_2-5.png'
      : '/img/shelly_1.png';
  });

  // --------------------------------------------------------------------------
  // Update Checker
  // --------------------------------------------------------------------------

  const checkForUpdate = async (currentVersion, model) => {
    try {
      const response = await axios.get(
        'https://rojer.me/files/shelly/update.json'
      );

      // const data = response.data;

      // dummy data
      const data = [
        [
          '.*',
          {
            version: '2.10.2',
            rel_notes:
              'https://github.com/mongoose-os-apps/shelly-homekit/releases/tag/2.10.1',
            urls: {
              Shelly1:
                'https://rojer.me/files/shelly/2.10.1/shelly-homekit-Shelly1.zip',
              Shelly25:
                'https://rojer.me/files/shelly/2.10.1/shelly-homekit-Shelly25.zip',
            },
          },
        ],
      ];

      let cfg, latestVersion, updateURL, relNotesURL;
      for (let i in data) {
        let re = new RegExp(data[i][0]);
        console.log(currentVersion);
        console.log(re);
        if (currentVersion.match(re)) {
          cfg = data[i][1];
          break;
        }
      }
      if (cfg) {
        latestVersion = cfg.version;
        relNotesURL = cfg.rel_notes;
        if (cfg.urls) updateURL = cfg.urls[model];
      }

      console.log('Version:', latestVersion, 'URL:', updateURL);
      updateAvailable.value = isNewer(latestVersion, currentVersion);
    } catch (err) {
      console.log(err);
    }
  };

  function isNewer(v1, v2) {
    let vi1 = parseVersion(v1),
      vi2 = parseVersion(v2);
    if (vi1.major != vi2.major) return vi1.major > vi2.major;
    if (vi1.minor != vi2.minor) return vi1.minor > vi2.minor;
    if (vi1.patch != vi2.patch) return vi1.patch > vi2.patch;
    if (vi1.variant != vi2.variant) return true;
    if (vi1.varSeq != vi2.varSeq) return vi1.varSeq > vi2.varSeq;
    return false;
  }

  // major.minor.patch-variantN
  function parseVersion(versionString) {
    let version = versionString.match(
      /^(?<major>\d+).(?<minor>\d+).(?<patch>\d+)-?(?<variant>[a-z]*)(?<varSeq>\d*)$/
    ).groups;
    version.major = parseInt(version.major);
    version.minor = parseInt(version.minor);
    version.patch = parseInt(version.patch);
    version.varSeq = parseInt(version.varSeq) || 0;

    return version;
  }
</script>

<style scoped>
  img {
    height: 20vh;
  }

  #updateBanner {
    background: darkred;
    padding: 0.5rem;
    color: white;
    border-radius: 15px;
  }
</style>
