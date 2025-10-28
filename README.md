 
<!-- Save as converter.html and open in a modern browser -->
<!doctype html>
<html lang="hi">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>Video → MP3 Converter (Legal: user uploads file)</title>
  <style>
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto;max-width:900px;margin:3rem auto;padding:1rem}
    .box{border:1px dashed #bbb;padding:1rem;border-radius:8px;text-align:center}
    button{padding:0.6rem 1rem;margin:0.5rem}
    progress{width:100%}
  </style>
</head>
<body>
  <h1>Video → MP3 (Client-side)</h1>
  <p>Yeh tool sirf aapke *apne* uploaded videos ke liye hai. Kisi aur ke copyrighted videos upload/convert na karein bina permission.</p>

  <div class="box">
    <input id="file" type="file" accept="video/*"/><br/>
    <small>Select a video file (mp4,mov,webm...).</small>
    <div style="margin-top:1rem">
      <button id="toMp3">Convert to MP3</button>
      <button id="toMp4">Re-mux to MP4 (no re-encode)</button>
    </div>
    <div style="margin-top:1rem">
      <progress id="progress" value="0" max="100" style="display:none"></progress>
      <div id="status"></div>
    </div>
  </div>

  <script type="module">
    import {createFFmpeg, fetchFile} from "https://unpkg.com/@ffmpeg/ffmpeg@0.12.0/dist/ffmpeg.min.js";

    const ffmpeg = createFFmpeg({ log: true });
    const fileInput = document.getElementById('file');
    const toMp3Btn = document.getElementById('toMp3');
    const toMp4Btn = document.getElementById('toMp4');
    const progressEl = document.getElementById('progress');
    const status = document.getElementById('status');

    async function loadFF() {
      if (!ffmpeg.isLoaded()) {
        status.textContent = 'Loading converter (first-time, may take a few seconds)...';
        await ffmpeg.load();
        status.textContent = 'Ready.';
      }
    }

    toMp3Btn.addEventListener('click', async () => {
      if (!fileInput.files.length) return alert('Pehle file select karo.');
      await loadFF();
      const file = fileInput.files[0];
      const inName = 'input' + (file.name.match(/\.\w+$/)||['.mp4'])[0];
      const outName = 'output.mp3';
      status.textContent = 'Uploading file to virtual FS...';
      ffmpeg.FS('writeFile', inName, await fetchFile(file));
      status.textContent = 'Converting to MP3... (browser CPU use karega)';
      progressEl.style.display = 'block';
      // basic conversion: keep default audio bitrate
      await ffmpeg.run('-i', inName, '-q:a', '0', '-map', 'a', outName);
      const data = ffmpeg.FS('readFile', outName);
      const url = URL.createObjectURL(new Blob([data.buffer], { type: 'audio/mpeg' }));
      const a = document.createElement('a');
      a.href = url; a.download = file.name.replace(/\.\w+$/, '') + '.mp3';
      a.textContent = 'Download MP3';
      status.innerHTML = 'Conversion finished. ';
      status.appendChild(a);
      progressEl.style.display = 'none';
    });

    toMp4Btn.addEventListener('click', async () => {
      if (!fileInput.files.length) return alert('Pehle file select karo.');
      await loadFF();
      const file = fileInput.files[0];
      const ext = (file.name.match(/\.\w+$/)||['.mp4'])[0];
      const inName = 'input' + ext;
      const outName = 'remuxed.mp4';
      ffmpeg.FS('writeFile', inName, await fetchFile(file));
      status.textContent = 'Re-muxing to MP4 (no re-encode)...';
      // -c copy tries to avoid re-encoding if possible
      await ffmpeg.run('-i', inName, '-c', 'copy', outName);
      const data = ffmpeg.FS('readFile', outName);
      const url = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
      const a = document.createElement('a');
      a.href = url; a.download = file.name.replace(/\.\w+$/, '') + '.mp4';
      a.textContent = 'Download MP4';
      status.innerHTML = 'Ready. ';
      status.appendChild(a);
    });

  </script>

  <p style="margin-top:1rem;font-size:0.9rem;color:#555">Note: large files may be slow. For better UX, consider server-side ffmpeg or chunked processing.</p>
</body>
</html>
