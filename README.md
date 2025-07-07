<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Upload M3U Playlist Player</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <style>
    body { font-family: sans-serif; margin: 20px; }
    #playlist { max-height: 200px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; margin-top: 10px; }
    #playlist li { cursor: pointer; padding: 6px; }
    #playlist li.active { background-color: #d0f0ff; font-weight: bold; }
  </style>
</head>
<body>

<h2>Upload M3U File to Play M3U8 Streams</h2>
<input type="file" id="m3uFile" accept=".m3u,.m3u8">
<video id="video" width="640" height="360" controls autoplay muted></video>
<ul id="playlist"></ul>

<script>
  const fileInput = document.getElementById('m3uFile');
  const video = document.getElementById('video');
  const playlistUI = document.getElementById('playlist');
  let hls;
  let streams = [];
  let current = 0;

  fileInput.addEventListener('change', async (event) => {
    const file = event.target.files[0];
    if (!file) return;
    const text = await file.text();
    streams = text
      .split('\\n')
      .map(line => line.trim())
      .filter(line => line && !line.startsWith('#') && line.endsWith('.m3u8'));

    if (streams.length === 0) {
      alert('No valid .m3u8 links found.');
      return;
    }

    renderPlaylist();
    playStream(0);
  });

  function renderPlaylist() {
    playlistUI.innerHTML = '';
    streams.forEach((url, i) => {
      const li = document.createElement('li');
      li.textContent = url;
      li.onclick = () => playStream(i);
      if (i === current) li.classList.add('active');
      playlistUI.appendChild(li);
    });
  }

  function playStream(i) {
    current = i;
    const url = streams[i];

    if (hls) hls.destroy();

    if (Hls.isSupported()) {
      hls = new Hls();
      hls.loadSource(url);
      hls.attachMedia(video);
      hls.on(Hls.Events.MANIFEST_PARSED, () => video.play());
    } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
      video.src = url;
      video.play();
    } else {
      alert('This browser does not support HLS.');
    }

    updateActive();
  }

  function updateActive() {
    Array.from(playlistUI.children).forEach((li, i) => {
      li.classList.toggle('active', i === current);
    });
  }

  video.addEventListener('ended', () => {
    if (current + 1 < streams.length) {
      playStream(current + 1);
    }
  });
</script>

</body>
</html>
