# 好玩的响指粒子化效果

在复联里看到经典的响指消散效果，觉得很有意思。很多前端大佬争相实现，其中方案有很多，但都用到了html2canvas,比如将图片的每个像素值随机分配给其他canvas并该点变为纯白即多canvas分解方案，或者是直接生成很多canvas每个canvas随机获得主图的某个像素点，然后多个canvas随机旋转开散，整个过程中生成一个新div覆盖住原有的div，最后将原有div的可见性隐藏掉。这里用了后者的方案。

> npm install -g http-server

> httpserver ./

本地启动之后打开页面即可

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>响指</title>
    <style>
        .role {
            height: 100vh;
            display: flex;
            justify-content: center;
            align-content: center;
        }

        img {
            width: 400px;
            height: 400px;
            border-radius: 50%;
        }

        .dust-container {
            position: absolute;
        }

        .dust-container canvas {
            position: absolute;
            left: 0;
            top: 0;
            transition: transform 1s ease-out, opacity 1s ease-out;
            opacity: 1;
            transform: rotate(0deg) translate(0px, 0px) rotate(0deg);
        }
    </style>
</head>

<body>
    <div class="role">
        <img src="./mie.JPG">
    </div>
    <script src="./public/js/html2canvas.min.js"></script>
    <script>
        let REPEAT = 2;
        let SPARES = 32;
        dom_img = document.querySelector('img');

        function replaceElementVisually($old, $new) {
            const $parent = $old.offsetParent;
            $new.style.top = `${$old.offsetTop}px`;
            $new.style.left = `${$old.offsetLeft}px`;
            $new.style.width = `${$old.offsetWidth}px`;
            $new.style.height = `${$old.offsetHeight}px`;
            $parent.appendChild($new);
            $old.style.visibility = "hidden";
        }

        function generateFrames($canvas, count = 32) {
            const {
                width,
                height
            } = $canvas;
            const ctx = $canvas.getContext("2d");
            const originalData = ctx.getImageData(0, 0, width, height);
            const imageDatas = [...Array(count)].map(
                (_, i) => ctx.createImageData(width, height)
            );
            for (let x = 0; x < width; ++x) {
                for (let y = 0; y < height; ++y) {
                    for (let i = 0; i < REPEAT; ++i) {
                        const
                            dataIndex = Math.floor(count * (Math.random() + 2 * x / width) / 3);
                        const pixelIndex = (y * width + x) * 4;
                        for (let offset = 0; offset < 4; ++offset) {
                            imageDatas[dataIndex].data[pixelIndex +
                                offset] = originalData.data[pixelIndex + offset];
                        }
                    }
                }
            }
            return imageDatas.map(data => {
                const $c = $canvas.cloneNode(true);
                $c.getContext("2d").putImageData(data, 0, 0);
                return $c;
            });
        }

        function snapToDust($elm, callback) {
            html2canvas($elm, {
                allowTaint: true,
            }).then($canvas => {

                // create a container
                const $container = document.createElement("div");
                $container.classList.add("dust-container");

                // frames for animation
                const $frames = generateFrames($canvas, SPARES);
                $frames.forEach(($frame, i) => {
                    $frame.style.transitionDelay = `${1.35 * i / $frames.length}s`;
                    $container.appendChild($frame);
                });

                // insert canvas into DOM over the element
                replaceElementVisually($elm, $container);

                // animate them
                $container.offsetHeight;
                $frames.forEach($frame => {
                    const randomRadian = 2 * Math.PI * (Math.random() - 0.5);
                    $frame.style.transform =
                        `rotate(${15 * (Math.random() - 0.5)}deg) translate(${60 * Math.cos(randomRadian)}px, ${30 * Math.sin(randomRadian)}px)
                         rotate(${15 * (Math.random() - 0.5)}deg)`;
                    $frame.style.opacity = 0;
                });

                if (callback && typeof callback == 'function') {
                    callback()
                }
            });
        }
        snapToDust(dom_img);
        // snapToDust(dom_img);
        if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
            // 获取用户的 media 信息
            navigator.mediaDevices
                .getUserMedia({
                    audio: true
                })
                .then(stream => {
                    // 当输入音量超过此值时，表示检测大音量输入(响指声)
                    const TRIGGER_VALUE = 0.9;
                    const audio_context = new AudioContext()
                    // 将麦克风的声音输入这个对象
                    mediaStreamSource = audio_context.createMediaStreamSource(
                        stream
                    );
                    // 创建一个音频分析对象，采样的缓冲区大小为4096，输入和输出都是单声道
                    scriptProcessor = audio_context.createScriptProcessor(
                        4096,
                        1,
                        1
                    );
                    // 将该分析对象与麦克风音频进行连接
                    mediaStreamSource.connect(scriptProcessor);
                    // 此举无甚效果，仅仅是因为解决 Chrome 自身的 bug
                    scriptProcessor.connect(audio_context.destination);
                    // 开始处理音频
                    scriptProcessor.onaudioprocess = function (e) {
                        // 获得缓冲区的输入音频，转换为包含了PCM通道数据的32位浮点数组
                        let buffer = e.inputBuffer.getChannelData(0);
                        // 获取缓冲区中最大的音量值
                        let maxVal = Math.max.apply(Math, buffer);
                        // 显示音量值
                        if (maxVal > TRIGGER_VALUE) {
                            //灰飞烟灭动画
                            console.log('brelly!');
                            snapToDust(dom_img);

                        }
                    };

                })
                .catch(error => {
                    console.log('获取音频时出现问题', error);
                });
        } else {
            console.log('不支持获得媒体接口', error);
        }
    </script>
</body>

</html>
```

中间有关像素点的计算，还有canvas的一些方法getImageData等看文档就行了，注意图像数据会被转化成一个大长整型数组，其中每4位代表一个像素点(rgba),至于随机化算法就仁者见仁了～