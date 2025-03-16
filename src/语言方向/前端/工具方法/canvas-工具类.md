# 工具方法

```javascript

import { unref, ref } from "vue";

export function useCanvasTools() {
    let lastX = 0;
    let lastY = 0;
    const _isDraw = ref(false)
    let currentCanvas = null;

    function startDrawing(x, y, canvas) {
        currentCanvas = canvas
        _isDraw.value = true
        lastX = x;
        lastY = y
        if (currentCanvas != null) {
            const ctx = currentCanvas.getContext('2d');
            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
        }
    }

    function isDrawing() {
        return unref(_isDraw)
    }

    function stopDrawing() {
        _isDraw.value = false
        if (currentCanvas != null) {
            console.log(11)
            const ctx = currentCanvas.getContext('2d');
            ctx.closePath();
            // ctx.globalAlpha = 1;
            // ctx.globalCompositeOperation = 'source-over';
            currentCanvas = null
        }
    }

    /**
     * 绘画功能
     * @param {*} canvas 
     * @param {*} x 
     * @param {*} y 
     */
    function paint(canvas, x, y, size = 10) {
        const _canvas = canvas || currentCanvas
        const ctx = _canvas.getContext('2d');

        ctx.globalCompositeOperation = 'source-over';
        ctx.strokeStyle = 'rgba(0, 0, 255,0.7)';
        ctx.globalAlpha = 0.75;
        ctx.lineWidth = size;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        // 线性插值平滑线条
        const steps = 5;
        for (let i = 1; i <= steps; i++) {
            const t = i / steps;
            const nx = lastX + (x - lastX) * t;
            const ny = lastY + (y - lastY) * t;
            ctx.lineTo(nx, ny);
        }
        ctx.stroke();
        [lastX, lastY] = [x, y];
    }


    /**
     * 擦除功能
     * @param {*} canvas 
     * @param {*} x 
     * @param {*} y 
     */
    function erase(canvas, x, y, size = 10) {
        const _canvas = canvas || currentCanvas
        const ctx = _canvas.getContext('2d');

        ctx.globalCompositeOperation = 'destination-out';
        ctx.globalAlpha = 0.5;
        ctx.lineWidth = size;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        ctx.lineTo(x, y);
        ctx.stroke();

        [lastX, lastY] = [x, y];
    }

    function clear(canvas) {
        const ctx = canvas.getContext('2d');
        ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    /**
     * 同步尺寸
     * @param {*} canvas 
     * @param {*} img 
     */
    function sameSize(canvas, img) {
        canvas.width = img.width;
        canvas.height = img.height;
    }

    function drawB64(canvas, base64Image) {
        const ctx = canvas.getContext('2d');
        // 创建一个 img 元素
        const img = new Image();
        // 设置 img 元素的 src 属性为 Base64 编码的图像
        img.src = base64Image;

        if (img.complete) {
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        } else {

            // 当图像加载完成后，将其绘制到 canvas 上
            img.onload = function () {
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            };
        }
    }
    return {
        startDrawing,
        stopDrawing,
        isDrawing,
        paint,
        erase,
        sameSize,
        drawB64,
        clear
    }
}
```