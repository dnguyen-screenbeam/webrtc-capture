<script>
  let dom = {
    video: {
      local: null,
      remote: null
    },
    img: null
  };

  let stream = null;
  let peers = {
    local: null,
    remote: null
  };

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

  const createRemotePeer = () => {
    const peer = new RTCPeerConnection();
    peer.onicecandidate = e => {
      if (e.candidate) {
        peers.local.addIceCandidate(e.candidate);
      }
    };

    // display stream on remote video element
    peer.ontrack = e => {
      if (dom.video.remote.srcObject !== e.streams[0]) {
        dom.video.remote.srcObject = e.streams[0];
        dom.video.remote.play();
      }
    };

    return peer;
  };

  const createLocalPeer = () => {
    const peer = new RTCPeerConnection();

    // Add tracks to local peer connection
    stream.getTracks().forEach(track => {
      peer.addTrack(track, stream);
    });

    // Set up event handlers for the ICE negotiation
    peer.onicecandidate = e => {
      if (e.candidate) {
        peers.remote.addIceCandidate(e.candidate);
      }
    };

    return peer;
  };

  const sendStream = async () => {
    peers.local = createLocalPeer();
    peers.remote = createRemotePeer();

    // Create Offer and set local description
    const offer = await peers.local.createOffer();
    await peers.local.setLocalDescription(offer);
    await peers.remote.setRemoteDescription(offer);

    // Create Answer and set remote description
    const answer = await peers.remote.createAnswer();
    await peers.remote.setLocalDescription(answer);
    await peers.local.setRemoteDescription(answer);
  };

  const getImageFromVideo = () => {
    var canvas = document.createElement('canvas');
    canvas.width = 1920;
    canvas.height = 1080;
    var ctx = canvas.getContext('2d');
    //draw image to canvas. scale to target dimensions
    ctx.drawImage(dom.video.remote, 0, 0, canvas.width, canvas.height);

    //convert to desired file format
    var dataURI = canvas.toDataURL('image/jpeg'); // can also use 'image/png'
    return dataURI;
  };

  const takeScreenshot = async () => {
    const img = getImageFromVideo();
    dom.img.src = img;
  };
</script>

<div class="h-full flex flex-col gap-3">
  <div class="flex gap-5">
    <button on:click={getStream}>Display screen</button>
    <button on:click={sendStream}>Send stream</button>
    <button on:click={takeScreenshot}>Take Screenshot</button>
  </div>

  <div class="flex gap-5 h-full flex-1">
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

  <div class="flex flex-col gap-5 h-full flex-1">
    <p class="text-lg font-semibold">Screenshot</p>
    <!-- svelte-ignore a11y-missing-attribute -->
    <img bind:this={dom.img} class="border border-gray-500" />
  </div>
</div>

<style lang="postcss">
  video {
    @apply border-gray-500 border-2;
  }

  button {
    @apply border-gray-500 border rounded-lg px-5 py-2 w-fit hover:bg-indigo-800 hover:text-gray-50;
  }
</style>
