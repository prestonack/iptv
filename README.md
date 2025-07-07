<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <title>Working M3U8 Playlist</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <style>
    body { font-family: sans-serif; margin: 20px; }
    #playlist { max-height: 120px; overflow-y: auto; border: 1px solid #ccc; padding: 8px; }
    #playlist li { padding: 6px; cursor: pointer; }
    #playlist li.active { background: #cde; font-weight: bold; }
  </style>
</head>
<body>

<h2>Working M3U8 Playlist Player</h2>

<video id="video" width="640" height="360" controls autoplay muted></video>

<ul id="playlist"></ul>

<script>
  const streams = [
    "https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8",
    "https://test-streams.mux.dev/tsfiles/hls/playlist.m3u8"
  ];

  const video = document.getElementById('video');
  const listEl = document.getElementById('playlist');
  let current = 0;
  let hls = null;

  function createList() {
    streams.forEach((url, i) => {
      const li = document.createElement('li');
      li.textContent = `Stream ${i + 1}`;
      li.onclick = () => play(i);
      if (i === current) li.classList.add('active');
      listEl.appendChild(li);
    });
  }

  function highlight() {
    Array.from(listEl.children).forEach((li, i) => {
      li.classList.toggle('active', i === current);
    });
  }

  function play(i) {
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
      alert("HLS not supported");
    }

    highlight();
  }

  video.addEventListener('ended', () => {
    const next = current + 1;
    if (streams[next]) play(next);
  });

  // Initialize
  createList();
  play(0);
</script>

</body>
</html>
