[toc]

---

# 计算坐标

```javascript
function cropImage(imagedata: ImageData) {
    let left = -1, top = -1, right = -1, bottom = -1;
    const idata = imagedata.data
    const count = idata.length;
    const countWidth = imagedata.width * 4;
    console.log(count)
    for (let i = 0, y = 0; i < count; i = i + 4) {
        if (i != 0 && i % countWidth == 0) {
            y++;
        }
        // console.log([idata[i], idata[i + 1], idata[i + 2], idata[i + 3]])
        if (
            idata[i] !== 255
            || idata[i + 1] !== 255
            || idata[i + 2] !== 255
            || idata[i + 3] !== 255
        ) {
            const x = (i - countWidth * y) / 4;
            if (left === -1) {
                left = x
            } else {
                if (left > x) {
                    left = x;
                }
                if (right === -1) {
                    right = x
                } else if (right < x) {
                    right = x
                }
            }
            if (top === -1) {
                top = y;
            } else {
                if (bottom === -1) {
                    bottom = y
                } else if (bottom < y) {
                    bottom = y
                }
            }
        }
    }
    return [left, top, right - left, bottom - top]
}
```

