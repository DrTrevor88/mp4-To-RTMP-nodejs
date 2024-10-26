# Giữ chất lượng video gốc => Không tốn CPU server => Tăng băng thông mạng thiết bị client xem video

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

# Giảm bitrate => tốn CPU => giảm băng thông client

```
const ffmpeg = require('fluent-ffmpeg');

// Đường dẫn tới tệp MP4 
const inputFile = '/www/wwwroot/pushLiveSixSix/input.mp4';

// URL RTMP đầy đủ bao gồm stream key
const rtmpUrl = 'rtmp://abc.com/live/c4203bb3?u=userId&t=B42XXXXXXXXD6FBXXXXXXXXXXXXX557CCCCC&s=733333092Xxxxxxxx902xxxE';

// Hàm phát luồng
function startStreaming() {
    const ffmpegPath = '/usr/bin/ffmpeg'; // Thay thế bằng đường dẫn thực tế tới ffmpeg
    ffmpeg.setFfmpegPath(ffmpegPath);

    ffmpeg()
        .input(inputFile)
        .inputOptions(['-re', '-stream_loop', '-1'])
        // Thiết lập codec và bitrate cho video và âm thanh
        .videoCodec('libx264')
        .audioCodec('aac')
        .videoBitrate('3000k') // Điều chỉnh bitrate phù hợp
        .audioBitrate('128k')
        // Không sử dụng .size() để giữ nguyên độ phân giải gốc
        .outputOptions([
            '-preset veryfast',
            '-tune zerolatency',
            '-profile:v baseline',
            '-level 3.1',
            '-g 60',
            '-r 30',
        ])
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

# Config systemd
Tạo file `/etc/systemd/system/pushlive66.service`

```
[Unit]
Description=Node.js Streaming Application
After=network.target

[Service]
ExecStart=/usr/bin/node /home/user/myapp/index.js
Restart=always
RestartSec=5
User=user
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

Để app tự động bật khi reboot lại server, chạy thêm lệnh 

```
sudo systemctl enable myapp.service
```

Chạy lệnh này khi có thay đổi liên quan systemd

```
sudo systemctl daemon-reload
```


# Thao tác thực hiện sau khi edit code file index.js
```
sudo systemctl restart pushlive66.service
```

```
sudo systemctl status pushlive66.service
```
