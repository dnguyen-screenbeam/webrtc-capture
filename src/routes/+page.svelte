<script>
  let dom = {
    video: {
      local: null,
      remote: null
    }
  };

  let stream = null;

  const getStream = async () => {
    try {
      stream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: true
      });

      dom.video.local.srcObject = stream;
      dom.video.local.play();
    } catch (err) {
      throw new Error(`Unable to get stream ${err.stack}`);
    }
  };
</script>

<div class="h-full flex flex-col gap-5">
  <button
    on:click={getStream}
    class="border-gray-500 border rounded-lg px-5 py-2 w-fit hover:bg-indigo-800 hover:text-gray-50"
    >Share screen</button
  >

  <div class="flex gap-5 h-full">
    <div class="flex flex-col gap-3 h-1/2">
      <p class="text-lg font-semibold">Local</p>
      <!-- svelte-ignore a11y-media-has-caption -->
      <video bind:this={dom.video.local} class="h-full" />
    </div>

    <div class="flex flex-col gap-3 h-1/2">
      <p class="text-lg font-semibold">Remote</p>
      <!-- svelte-ignore a11y-media-has-caption -->
      <video bind:this={dom.video.remote} class="h-full" />
    </div>
  </div>
</div>

<style lang="postcss">
  video {
    @apply border-gray-500 border-2;
  }
</style>
