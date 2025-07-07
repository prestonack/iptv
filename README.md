<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Online M3U8 Video Player</title>
  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
</head>
<body>
  <h1>Online M3U8 Video Player</h1>
  <input type="text" id="streamUrl" placeholder="Enter .m3u8 stream URL" size="60" />
  <button onclick="playStream()">Play</button>
  <br><br>
  <video id="video" controls width="640" height="360"></video>

  <script>
    function playStream() {
      const url = document.getElementById('streamUrl').value.trim();
      const video = document.getElementById('video');

      if (!url.endsWith('.m3u8')) {
        alert('Please enter a valid .m3u8 URL.');
        return;
      }

      if (Hls.isSupported()) {
        const hls = new Hls();
        hls.loadSource(url);
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED, () => {
          video.play();
        });
        hls.on(Hls.Events.ERROR, function(event, data) {
          console.error("HLS.js error:", data);
          alert("Failed to play the stream. Check the console for details.");
        });
      } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
        // Safari & iOS support HLS natively
        video.src = url;
        video.addEventListener('loadedmetadata', () => {
          video.play();
        });
      } else {
        alert('HLS not supported in this browser.');
      }
    }
  </script>
</body>
</html>
