<!-- ================================================= SCRIPT -->
<script lang="ts">
  import { onMount } from "svelte";
  import * as tf from "@tensorflow/tfjs";
  import * as cocoSsd from "@tensorflow-models/coco-ssd";
  import { browser } from "$app/environment";
  import { fade } from 'svelte/transition';

  let webcamAvailable = false;
  let video: any = null;
  let liveView: any = null;
  let model: any = null;

  interface Prediction {
    bbox: Array<number>;
    class: string;
    score: number;
  }

  async function getUserMediaSupported() {
    try {
      const _webcamAvailable = await navigator?.mediaDevices?.getUserMedia({
        video: true,
      });

      if (_webcamAvailable) {
        webcamAvailable = true;
      } else {
        alert("webcam is not available");
      }
    } catch (_) {
      alert("webcam is not available or you denied access");
    }
  }

  async function startWebcamStreaming() {
    if (!webcamAvailable) {
      await getUserMediaSupported();

      const constraints = {
        video: true,
      };

      navigator.mediaDevices.getUserMedia(constraints).then(function (stream) {
        video.srcObject = stream;
        video.addEventListener("loadeddata", predictWebcam);
      });
    }
  }

  let currentPredictions: Array<Prediction> = [];

  async function predictWebcam() {
    if (model === null) return;
    model.detect(video).then(function (predictions: Array<Prediction>) {
      currentPredictions = predictions.filter(
        (prediction) => prediction.score > 0.66
      );

      window.requestAnimationFrame(predictWebcam);
    });
  }

  onMount(async () => {
    if (browser) {
      model = await cocoSsd.load();
    }
  });
</script>

<!-- ================================================= CONTENT -->
<p>
  Supercharge your webcam with AI to make it smart thanks to <a
    href="https://www.tensorflow.org/js"
    ><span class="text-orange-300 font-bold">tensorflow.js</span></a
  >.
  <br class="hidden md:block" />
  The model used is
  <a
    href="https://www.npmjs.com/package/@tensorflow-models/coco-ssd"
    target="_blank"><span class="font-bold">coco-ssd</span></a
  >
  which is a pre-trained model for object detection based on the
  <a href="https://cocodataset.org/" target="_blank"
    ><span class="font-bold">COCO dataset</span>.</a
  >
  <br class="hidden md:block" />
  The model is capable of detecting
  <a
    href="https://github.com/tensorflow/tfjs-models/blob/master/coco-ssd/src/classes.ts"
    target="_blank"><span class="font-bold">80 classes</span></a
  > of objects.
</p>

<div
  id="liveView"
  class="relative border border-solid border-black bg-neutral-100 my-10"
  bind:this={liveView}
>
  <video
    id="webcam"
    bind:this={video}
    autoplay
    muted
    width="100"
    height="480"
  />
  {#if !webcamAvailable}
    <div
      class="absolute w-full top-44 flex flex-col items-center justify-center gap-4"
    >
      <p class="opacity-40">
        Please click the button below to activate your webcam.
        <br class="hidden md:block" />
        Don't forget to allow the access to your webcam.
      </p>
      <button on:click={startWebcamStreaming} disabled={model === null}>Activate</button>
      {#if model === null}
        <p out:fade={{duration: 300}}>please wait the model to load...</p>
      {/if}
    </div>
  {/if}
  {#each currentPredictions as currentPrediction}
    {@const scorePercentage = Math.round(currentPrediction.score * 100)}
    <div
      class="absolute z-20 overflow-hidden border border-solid border-black"
      style="left: {currentPrediction.bbox[0]}px; top: {currentPrediction
        .bbox[1]}px; width: {currentPrediction
        .bbox[2]}px; height: {currentPrediction.bbox[3]}px;"
    >
      <div
        class="relative bg-white w-full border-b border-solid border-black px-2"
      >
        <div
          class="absolute top-0 left-0 z-10 bg-green-300 h-full"
          style="width: {scorePercentage}%;"
        ></div>
        <div
          class="px-2 w-full absolute top-0 left-0 z-20 flex justify-between"
        >
          <p class="w-fit">{currentPrediction.class}</p>
          <p class="w-fit">{scorePercentage}%</p>
        </div>
        <p class="invisible">nop</p>
      </div>
      <div class="w-full h-full bg-green-300 opacity-30"></div>
    </div>
  {/each}
</div>

<!-- ================================================= CSS -->
<style lang="postcss">
  p {
    @apply text-center;
  }

  video {
    @apply block -z-10 w-full md:w-[640px];
    height: 480px;
  }

  .invisible {
    opacity: 0.2;
  }

  .camView {
    @apply border border-solid border-black;
    position: relative;
    float: left;
    width: calc(100% - 20px);
    margin: 10px;
    cursor: pointer;
  }

  .camView p {
    position: absolute;
    top: 0;
    padding: 5px;
    background-color: rgba(255, 111, 0, 0.85);
    color: #fff;
    border: 1px dashed rgba(255, 255, 255, 0.7);
    z-index: 2;
    font-size: 12px;
  }

  .highlighter {
    top: 0;
    background: rgba(0, 255, 0, 0.25);
    border: 1px dashed #fff;
    z-index: 10;
    position: absolute;
  }
</style>
