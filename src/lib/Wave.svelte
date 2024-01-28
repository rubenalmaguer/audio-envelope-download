<script type="module">
  import { onMount } from "svelte";
  import WaveSurfer from "wavesurfer.js"; // OR from  "https://unpkg.com/wavesurfer.js@7/dist/wavesurfer.esm.js";
  import EnvelopePlugin from "wavesurfer.js/dist/plugins/envelope.js"; // OR from "https://unpkg.com/wavesurfer.js@7/dist/plugins/envelope.js";

  import {
    downloadAudio,
    gradualVolumeChange,
    applyEnevelopePoints,
  } from "./AudioDownloader.js";

  let wavesurfer;
  let envelope;
  let currentVolume;
  let playButton;

  onMount(() => {
    wavesurfer = WaveSurfer.create({
      container: "#container",
      waveColor: "#4F4A85",
      progressColor: "#383351",
      url: "https://upload.wikimedia.org/wikipedia/commons/f/f7/1AX_California_A_Express_Inbound_Route_Announcement.wav",
      sampleRate: 44100, // Default is only 8000!, consider 48000Hz
    });

    const isMobile = top.matchMedia("(max-width: 900px)").matches;

    // Initialize the Envelope plugin
    envelope = wavesurfer.registerPlugin(
      EnvelopePlugin.create({
        volume: 0.8,
        lineColor: "rgba(255, 0, 0, 0.5)",
        lineWidth: "4px",
        dragPointSize: isMobile ? 20 : 12,
        dragLine: !isMobile,
        dragPointFill: "rgba(0, 255, 255, 0.8)",
        dragPointStroke: "rgba(0, 0, 0, 0.5)",

        points: [
          { time: 0.2, volume: 0.7 },
          { time: 0.4, volume: 0.1 },
        ],
      })
    );

    envelope.on("points-change", (points) => {
      //console.log("Envelope points changed", points);
    });

    envelope.addPoint({ time: 0.7, volume: 0.9 });

    // Show the current volume
    const showVolume = () => {
      currentVolume = envelope.getCurrentVolume().toFixed(2);
    };
    envelope.on("volume-change", showVolume);
    wavesurfer.on("ready", showVolume);

    // Play/pause button
    wavesurfer.once("ready", () => {
      playButton.onclick = () => {
        wavesurfer.playPause();
      };
    });
    wavesurfer.on("play", () => {
      playButton.textContent = "Pause";
    });
    wavesurfer.on("pause", () => {
      playButton.textContent = "Play";
    });
  });

  function randomizePoints() {
    const points = [];
    const len = 5 * Math.random();
    for (let i = 0; i < len; i++) {
      points.push({
        time: Math.random() * wavesurfer.getDuration(),
        volume: Math.random(),
      });
    }
    envelope.setPoints(points);
  }

  function download() {
    const decoded = wavesurfer.getDecodedData();
    //downloadAudio(decoded); // Download original
    const ramped = gradualVolumeChange(decoded);
    //downloadAudio(ramped); // Download constant fade-in
    const enveloped = applyEnevelopePoints(decoded, envelope.getPoints());
    downloadAudio(enveloped); // Download result from envelope UI
  }
</script>

<button bind:this={playButton} style="min-width: 5em" id="play">Play</button>
<button on:click={randomizePoints} style="margin: 0 1em 2em" id="randomize"
  >Randomize points</button
>

Volume: <span>{currentVolume}</span>

<button on:click={download} style="margin: 0 1em 2em" id="randomize"
  >Download</button
>

<div id="container" style="border: 1px solid #ddd;"></div>
