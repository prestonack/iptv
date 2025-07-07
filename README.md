<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>M3U8 Playlist Player</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    #playlist { max-height: 200px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; }
    #playlist li { cursor: pointer; padding: 5px; }
    #playlist li.active { background-color: #d0f0ff; font-weight: bold; }
  </style>
</head>
<body>
  <h1>M3U8 Playlist Player</h1>
  
  <input type="text" id="playlistUrl" placeholder="Enter .m3u URL (list of .m3u8 streams)" size="60" />
  <button onclick="loadPlaylist()">Load Playlist</button>

  <video id="video" width="640" height="360" controls autoplay></video>

  <h3>Playlist</h3>
  <ul id="playlist"></ul>

  <script>
    let streams = [];
    let currentIndex = 0;
    let hls;

    const video = document.getElementById('video');
    const playlistElement = document.getElementById('playlist');

    async function loadPlaylist() {
      const url = document.getElementById('playlistUrl').value.trim();
      if (!url) return alert('Please enter a valid .m3u URL');

      try {
        const res = await fetch(url);
        const text = await res.text();

        streams = text.split('\n')
          .map(line => line.trim())
          .filter(line => line && !line.startsWith('#') && line.endsWith('.m3u8'));

        if (streams.length === 0) {
          alert('No .m3u8 URLs found.');
          return;
        }

        renderPlaylist();
        playStream(0);
      } catch (err) {
        alert('Failed to load playlist: ' + err.message);
      }
    }

    function renderPlaylist() {
      playlistElement.innerHTML = '';
      streams.forEach((url, i) => {
        const li = document.createElement('li');
        li.textContent = url;
        li.onclick = () => playStream(i);
        if (i === currentIndex) li.classList.add('active');
        playlistElement.appendChild(li);
      });
    }

    function playStream(index) {
      if (index < 0 || index >= streams.length) return;
      currentIndex = index;

      const url = streams[index];
      const items = playlistElement.querySelectorAll('li');
      items.forEach((li, i) => li.classList.toggle('active', i === index));

      if (hls) {
        hls.destroy();
      }

      if (Hls.isSupported()) {
        hls = new Hls();
        hls.loadSource(url);
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED, () => video.play());
      } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
        video.src = url;
        video.play();
      } else {
        alert('This browser does not support HLS playback.');
      }
    }

    video.addEventListener('ended', () => {
      if (currentIndex + 1 < streams.length) {
        playStream(currentIndex + 1);
      }
    });
  </script>
</body>
</html>
