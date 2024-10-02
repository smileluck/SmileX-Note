[TOC]

---

# 加载mov问题

1. `mov` 文件加载，如果转换成 `base64`，将无法加载；

2. 同时 `loadeddata` 和 `loadedmetadata` 无法触发；

3. 但是使用 `getObjectURL` 加载后，可以正常触发函数

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <video width="400" autoplay controls>
        <source src="IMG_7666.MOV" type="video/mp4">>

    </video>


    <video id="test" width="400" src="IMG_7666.MOV" autoplay controls>


    </video>

    <input type="file" onchange="handleFile(this)" />
</body>

</html>
<script>
    window.onload = () => {


        const video = document.getElementById("test");

        video.addEventListener('loadeddata', async () => {
            console.log(333331222222323)
            const duration = video.duration; //obtain video length
            const parentWidth = myCanvas.value.clientWidth;
            const parentHeight = myCanvas.value.clientHeight;
            // console.log(parentWidth + " " + parentHeight);
            //reset the innerHTML property
            myCanvas.value.innerHTML = ''; //innerHTML is a HTML property NOT attribute (property can be changed at run time, attributes cannot)

        });
    }
    handleFile = (file) => {
        console.log(file)
        document.getElementById("test").src = getObjectURL(file.files[0])
    }

    // 创建一个可存取该file的url的函数
    function getObjectURL(file) {
        console.log(file)
        var url = null;
        // 下面函数执行的效果是一样的，只是需要针对不同的浏览器执行不同的 js 函数而已
        if (window.createObjectURL != undefined) { // basic
            url = window.createObjectURL(file);
        } else if (window.URL != undefined) { // mozilla(firefox)
            url = window.URL.createObjectURL(file);
        } else if (window.webkitURL != undefined) { // webkit or chrome
            url = window.webkitURL.createObjectURL(file);
        }
        return url;
    }
</script>
```


