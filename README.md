<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>M3U8 Playlist Video Player</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
    }
    #playlist {
      max-height: 200px;
      overflow-y: scroll;
      border: 1px solid #ccc;
      padding: 10px;
      margin-top: 10px;
    }
    #playlist li {
      cursor: pointer;
      padding: 5px;
    }
    #playlist li.active {
      background-color: #d1eaff;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h2>M3U8 Playlist Video Player</h2>

  <input type="text" id="playlistUrl" placeholder="Enter URL to .m3u/.m3u8 playlist" size="60" />
  <button onclick="loadPlaylist()">Load Playlist</button>

  <ul id="playlist"></ul>

  <video id="video" width="640" height="360" controls></video>

  <script>
    let playlist = [];
    let currentIndex = 0;
    let hls;
    const video = document.getElementById('video');
    const playlistUI = document.getElementById('playlist');

    async function loadPlaylist() {
      const url = document.getElementById('playlistUrl').value.trim();
      if (!url) return alert("Please enter a playlist URL.");

      try {
        const response = await fetch(url);
        const text = await response.text();

        // Parse M3U list and filter out comments
        playlist = text.split('\n')
          .map(line => line.trim())
          .filter(line => line && !line.startsWith('#') && line.endsWith('.m3u8'));

        if (playlist.length === 0) {
          alert('No valid .m3u8 URLs found in the playlist.');
          return;
        }

        renderPlaylist();
        playStream(0);
      } catch (err) {
        console.error(err);
        alert('Failed to load playlist: ' + err.message);
      }
    }

    function renderPlaylist() {
      playlistUI.innerHTML = '';
      playlist.forEach((url, index) => {
        const li = document.createElement('li');
        li.textContent = url;
        li.onclick = () => playStream(index);
        if (index === currentIndex) {
          li.classList.add('active');
        }
        playlistUI.appendChild(li);
      });
    }

    function playStream(index) {
      if (index < 0 || index >= playlist.length) return;

      currentIndex = index;
      const url = playlist[index];

      // Clean up old HLS instance
      if (hls) {
        hls.destroy();
      }

      if (Hls.isSupported()) {
        hls = new Hls();
        hls.loadSource(url);
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED, () => {
          video.play();
        });
        hls.on(Hls.Events.ERROR, function (event, data) {
          console.error("HLS.js error:", data);
          alert("Error playing stream: " + data.details);
        });
      } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
        video.src = url;
        video.play();
      } else {
        alert('This browser does not support HLS playback.');
      }

      updateActiveList();
    }

    function updateActiveList() {
      const items = playlistUI.querySelectorAll('li');
      items.forEach((li, idx) => {
        li.classList.toggle('active', idx === currentIndex);
      });
    }

    // Autoplay next video when one ends
    video.addEventListener('ended', () => {
      if (currentIndex + 1 < playlist.length) {
        playStream(currentIndex + 1);
      }
    });
  </script>
</body>
</html>
