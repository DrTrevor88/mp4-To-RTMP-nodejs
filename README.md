```
const ffmpeg = require('fluent-ffmpeg');

// Đường dẫn tới tệp MP4 
const inputFile = 'input.mp4';

// URL RTMP đầy đủ bao gồm stream key
const rtmpUrl = 'rtmp://abc.com/live/c4203bb3?u=userId&t=B42XXXXXXXXD6FBXXXXXXXXXXXXX557CCCCC&s=733333092Xxxxxxxx902xxxE';

// Hàm phát luồng
function startStreaming() {
  const command = ffmpeg()
    .input(inputFile)
    .inputOptions(['-re', '-stream_loop', '-1'])
    .outputOptions('-c copy')
    .format('flv')
    .output(rtmpUrl)
    .on('start', function(commandLine) {
      console.log('Đã bắt đầu FFmpeg với lệnh: ' + commandLine);
    })
    .on('progress', function(progress) {
      console.log('Đang phát: ' + progress.timemark);
    })
    .on('error', function(err, stdout, stderr) {
      console.error('Có lỗi xảy ra: ' + err.message);
      console.error('stdout: ' + stdout);
      console.error('stderr: ' + stderr);
      console.log('Đang thử phát lại sau 5 giây...');
      // Đợi 5 giây trước khi thử lại
      setTimeout(() => {
        startStreaming();
      }, 5000);
    })
    .on('end', function() {
      console.log('Quá trình phát luồng kết thúc!');
      console.log('Đang thử phát lại sau 5 giây...');
      // Đợi 5 giây trước khi thử lại
      setTimeout(() => {
        startStreaming();
      }, 5000);
    })
    .run();
}

// Bắt đầu phát luồng
startStreaming();

```
