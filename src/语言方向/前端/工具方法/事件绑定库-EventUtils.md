[toc]

---

# 库代码

```typescript
// 添加事件
export function addEventListener(element: any, type: string, handler: EventListener) {
    if (element.addEventListener) {
        element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
        element.attachEvent("on" + type, handler);
    } else {
        element["on" + type] = handler;
    }
}

//删除句柄---匿名函数不能被移除
export function removeEventListener(element: any, type: string, handler: EventListener) {
    if (element.removeEventListener) {
        element.removeEventListener(type, handler, false);
    } else if (element.detachEvent) {
        element.detachEvent("on" + type, handler);
    } else {
        element["on" + type] = null;
    }
}
```



# 关于input file 取消上传文件

```javascript

const videoUploadLoading = ref(true)

async function cancelMediaFile() {
    setTimeout(() => {
        videoUploadLoading.value = false;
        removeEventListener(window, "focus", cancelMediaFile)
    }, 200)
}

const inputFile = document.getElementById('inputFile');
inputFile.onclick=function(){
	// 核心部分
	addEventListener(window, "focus", cancelMediaFile)
}
```

