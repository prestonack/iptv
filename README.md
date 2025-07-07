<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>HLS Playlist Player</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
</head>
<body>
  <h1>HLS Playlist Player</h1>
  <input type="text" id="streamUrl" placeholder="Enter .m3u8 URL" />
  <button onclick="playStream()">Play</button>
  <video id="video" controls width="640" height="360"></video>

  <script>
    function playStream() {
      const url = document.getElementById('streamUrl').value;
      const video = document.getElementById('video');

      if (Hls.isSupported()) {
        const hls = new Hls();
        hls.loadSource(url);
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED, () => {
          video.play();
        });
      } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
        video.src = url;
        video.addEventListener('loadedmetadata', () => {
          video.play();
        });
      } else {
        alert('HLS not supported in this browser');
      }
    }
  </script>
</body>
</html>

  
