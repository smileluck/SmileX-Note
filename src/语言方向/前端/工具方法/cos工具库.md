[toc]

---

```javascript


import COS from 'cos-js-sdk-v5'; // 通过 npm 安装的 SDK
import { fetchCommonSts } from "@/api/tap/common"
import { nextTick, ref } from 'vue';
import {ElMessage} from 'element-plus'

const region = 'ap-guangzhou'

// 2024年7月29日，缓存身份验证
// 缓存身份验证
let cachedAuth = null;
let authPromise = null;

async function getCachedAuthorization() {
  // console.log(cachedAuth)
  if (cachedAuth && cachedAuth.expiredTime > Math.floor(Date.now() / 1000)) {
    // console.log("有缓存", cachedAuth);
    return Promise.resolve(cachedAuth);
  }

  if (authPromise) {
    // console.log("有任务执行", cachedAuth);
    return authPromise;
  }

  // console.log("没有缓存和任务", cachedAuth);

  authPromise = fetchCommonSts().then(res => {
    cachedAuth = {
      tmpSecretId: res.tmpSecretId,
      tmpSecretKey: res.tmpSecretKey,
      sessionToken: res.sessionToken,
      startTime: res.startTime,
      expiredTime: res.expiredTime,
    };
    authPromise = null;  // 重置 authPromise 以便下次请求
    return Promise.resolve(cachedAuth);;
  }).catch(err => {
    authPromise = null;
    throw err;
  });

  return authPromise;
}

export function useHasLogin() {

  const _cos = new CosMethod()


  return {
    putObject: _cos.putObject,
    uploadFile: _cos.uploadFile,
    uploadMultiFile: _cos.uploadMultiFile,
    getUrl: _cos.getUrl,
    getMultiUrl: _cos.getMultiUrl,
    getBucket: _cos.getBucket,
    doesObjectExist: _cos.doesObjectExist,
    updateCos: _cos.updateCos,
    region
  }
}

class CosMethod {
  #_cos;
  _showProgress = ref(false);
  _progressPercent = ref(0);
  _progressData = ref();

  constructor() {
    this.#_cos = new COS({
      // getAuthorization 必选参数
      getAuthorization: function (options, callback) {
        // 初始化时不会调用，只有调用 cos 方法（例如 cos.putObject）时才会进入
        // 异步获取临时密钥
        // 服务端 JS 和 PHP 例子：https://github.com/tencentyun/cos-js-sdk-v5/blob/master/server/
        // 服务端其他语言参考 COS STS SDK ：https://github.com/tencentyun/qcloud-cos-sts-sdk
        // STS 详细文档指引看：https://cloud.tencent.com/document/product/436/14048

        getCachedAuthorization().then((res) => {
          // console.log(2222222222, res)
          // fetchCommonSts().then((res) => {
          callback({
            TmpSecretId: res.tmpSecretId,
            TmpSecretKey: res.tmpSecretKey,
            SecurityToken: res.sessionToken,
            // 建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
            StartTime: res.startTime, // 时间戳，单位秒，如：1580000000
            ExpiredTime: res.expiredTime, // 时间戳，单位秒，如：1580000000
          });
        })
      }
    });

  }


  /**
   * 单文件简单上传
   * @param taskId
   */
  putObject = (uploadData) => {
    this.#_cos.putObject({
      Bucket: uploadData.bucket, /* 填入您自己的存储桶，必须字段 */
      Region: region,  /* 存储桶所在地域，例如ap-beijing，必须字段 */
      Key: uploadData.Key,  /* 存储在桶里的对象键（例如1.jpg，a/b/test.txt），必须字段 */
      Body: uploadData.Body, /* 必须，上传文件对象，可以是input[type="file"]标签选择本地文件后得到的file对象 */
      SliceSize: 1024 * 1024 * 20,     /* 触发分块上传的阈值，超过5MB使用分块上传，非必须 */
      onTaskReady: function (taskId) {                   /* 非必须 */
        // // console.log(taskID);
      },
      onProgress: (_progressData) => {           /* 非必须 */
        this._progressData.value = _progressData
        this._showProgress.value = true;
        this._progressPercent.value = _progressData.percent
      }
    }, (err, data) => {
      if (err) {
       ElMessage.error("上传失败")
        if (uploadData?.fail && typeof uploadData?.fail === 'function') {
          uploadData?.fail();
        }
        return;
      }
      this._showProgress.value = false;
      nextTick(() => {
        this._progressPercent.value = 0
      })
      if (uploadData?.success && typeof uploadData.success === 'function') {
        uploadData?.success(data);
      }
    })
  }



  updateCos = (cosParams) => {
    this.#_cos = new COS({
      // getAuthorization 必选参数
      getAuthorization: function (options, callback) {
        // 初始化时不会调用，只有调用 cos 方法（例如 cos.putObject）时才会进入
        // 异步获取临时密钥
        // 服务端 JS 和 PHP 例子：https://github.com/tencentyun/cos-js-sdk-v5/blob/master/server/
        // 服务端其他语言参考 COS STS SDK ：https://github.com/tencentyun/qcloud-cos-sts-sdk
        // STS 详细文档指引看：https://cloud.tencent.com/document/product/436/14048

        // console.log(2222222222, res)
        // fetchCommonSts().then((res) => {
        callback({
          TmpSecretId: cosParams.tmpSecretId,
          TmpSecretKey: cosParams.tmpSecretKey,
          SecurityToken: cosParams.sessionToken,
          // 建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
          StartTime: cosParams.startTime, // 时间戳，单位秒，如：1580000000
          ExpiredTime: cosParams.expiredTime, // 时间戳，单位秒，如：1580000000
        });
      }
    });
  }


  /**
   * 任务上传文件
   * @param taskId
   */
  uploadFile = (uploadData) => {
    this.#_cos.uploadFile({
      Bucket: uploadData.bucket, /* 填入您自己的存储桶，必须字段 */
      Region: region,  /* 存储桶所在地域，例如ap-beijing，必须字段 */
      Key: uploadData.Key,  /* 存储在桶里的对象键（例如1.jpg，a/b/test.txt），必须字段 */
      Body: uploadData.Body, /* 必须，上传文件对象，可以是input[type="file"]标签选择本地文件后得到的file对象 */
      // SliceSize: 1024 * 1024 * 5,     /* 触发分块上传的阈值，超过5MB使用分块上传，非必须 */
      onTaskReady: function (taskId) {                   /* 非必须 */
        // // console.log(taskID);
      },
      onProgress: (_progressData) => {           /* 非必须 */
        this._progressData.value = _progressData
        this._showProgress.value = true;
        this._progressPercent.value = _progressData.percent
      },
      onFileFinish: (err, data, options) => {   /* 非必须 */
        if (err) {
         ElMessage.error("上传失败")
          if (uploadData?.fail && typeof uploadData?.fail === 'function') {
            uploadData?.fail();
          }
          return;
        }
        this._showProgress.value = false;
        nextTick(() => {
          this._progressPercent.value = 0
        })
        if (uploadData?.success && typeof uploadData.success === 'function') {
          uploadData?.success(data, options);
        }
      }
    }, function (err, data) {
      console.log(err || data);
    })
  }


  /**
   * 任务批量上传文件
   * @param taskId
   */
  uploadMultiFile = (uploadData) => {
    const { files, keys } = uploadData
    if (files.length == 0) {
     ElMessage.error("请选择上传文件");
      return;
    }
    if (files.length != keys.length) {
     ElMessage.error("上传异常");
      return;
    }

    const taskIds = [];  // 存储任务ID
    const arr = []

    for (let i = 0; i < files.length; i++) {
      arr.push({
        Bucket: uploadData.bucket, /* 填入您自己的存储桶，必须字段 */
        Region: region,  /* 存储桶所在地域，例如ap-beijing，必须字段 */
        Key: keys[i],  /* 存储在桶里的对象键（例如1.jpg，a/b/test.txt），必须字段 */
        Body: files[i], /* 必须，上传文件对象，可以是input[type="file"]标签选择本地文件后得到的file对象 */
        onTaskReady: function (taskId) {
          // 获取taskId存储
          taskIds.push(taskId);
        }
      })
    }

    this.#_cos.uploadFiles({
      files: arr,
      SliceSize: 1024 * 1024 * 20,     /* 触发分块上传的阈值，超过5MB使用分块上传，非必须 */
      onProgress: (_progressData) => {           /* 非必须 */
        this._progressData.value = _progressData
        this._showProgress.value = true;
        this._progressPercent.value = _progressData.percent
      },
      onFileFinish: (err, data, options) => {   /* 非必须 */
        // console.log(options.Key + '上传' + (err ? '失败' : '完成'));
        if (err) {
          //ElMessage.error("上传失败")
          // 取消该任务
          // const index = keys.indexOf(options.Key);
          // const list = cos.getTaskList();
          // if (index !== -1 && taskIds[index]) {
          //     cos.cancelTask(taskIds[index]);
          //     console.log([...keys], taskIds, options.Key, index)
          // }

          // const list = cos.getTaskList();
          // for (let j = 0; j < list.length; j++) {
          //     const item = list[j];
          //     console.log(item.id, item.Key, item.state)
          //     if ((item.state === "waiting" || item.state === 'uploading') && taskIds.indexOf(item.id) > 0) {
          //         cos.cancelTask(item.id);
          //     }
          // }

        } else {
          console.log(`${options.Key} 上传成功`);
        }

        this._showProgress.value = false;
        nextTick(() => {
          this._progressPercent.value = 0
        })
        // console.log(1111);
        // console.log(err, 'err');
        // console.log(data, 'data');
      }
    }, function (err, data) {
      // console.log('err', err);
      // console.log(err, data);

      // if (err) {
      //     console.log('批次上传失败:', err);
      //     return;
      // }
      // 检查是否所有文件都上传成功
      const failedList = data.files.filter(file => file.error);
      if (failedList.length > 0) {
        console.log(`部分文件上传失败: ${failedList}`);
       ElMessage.error('上传失败');

        if (uploadData?.fail && typeof uploadData.fail === 'function') {
          uploadData?.fail();
        }
        return;
      }

      if (uploadData?.success && typeof uploadData.success === 'function') {
        uploadData?.success(data);
      }
    })
  }
  /**
   * 遍历列表
   */
  getBucket = (params) => {
    return new Promise((resolve, reject) => {
      this.#_cos.getBucket({
        Bucket: params.Bucket, // 填入您自己的存储桶，必须字段
        Region: params.Region || region,  // 存储桶所在地域，例如ap-beijing，必须字段
        Prefix: params.Prefix,              // Prefix表示列出的object的key以prefix开始，非必须
        Delimiter: params.Delimiter,            // Deliter表示分隔符, 设置为/表示列出当前目录下的object, 设置为空表示列出所有的object，非必须
      }, function (err, data) {
        console.log(err || data.CommonPrefixes);
        if (err) {
          reject(err)
        } else {
          resolve(data.Contents)
        }
      });
    });
  }

  // 使用 getUrl 方法获取对象的 URL
  getUrl = (params) => {
    const { Bucket, Region, Key, Sign } = params;
    return new Promise((resolve, reject) => {
      this.#_cos.getObjectUrl({
        Bucket: Bucket,
        Region: Region || region,
        Key: Key,
        Sign: Sign && true,
      },
        function (err, data) {
          if (err) {
            reject(err); // 如果有错误，拒绝 Promise
          } else {
            resolve(data.Url); // 如果成功，解析 Promise 并返回 URL
          }
        });
    });
  }

  // 批量获取 URL 的函数
  getMultiUrl = (params) => {
    // const limit = 10;
    // const { Bucket, Region, Key } = params;
    // const results= []; // 存储最终结果的数组
    // let index = 0; // 当前处理的索引
    // let runningCount = 0; // 当前正在运行的请求数

    // return new Promise((resolve, reject) => {
    //     function run() {
    //         if (index >= Key.length) {
    //             if (runningCount === 0) {
    //                 resolve(results);
    //             }
    //             return;
    //         }

    //         runningCount++;
    //         const keyIdx = index++;
    //         const key = Key[keyIdx];
    //         console.log(index)
    //         getUrl({ Bucket, Region, Key: key })
    //             .then(url => {
    //                 results[keyIdx] = url;
    //             })
    //             .catch(err => {
    //                 reject(err);
    //             })
    //             .finally(() => {
    //                 runningCount--;
    //                 run();
    //             });
    //     }

    //     for (let i = 0; i < limit && i < Key.length; i++) {
    //         run();
    //     }

    // });
    const { Bucket, Region, Key, Sign } = params;


    return limitQueueFn(Key, 10, (key) => { return this.getUrl({ Bucket, Region: Region || region, Key: key, Sign: Sign && true }); });
  }

  doesObjectExist = (params) => {
    const { Bucket, Region, Key } = params;
    return new Promise((resolve, reject) => {
      this.#_cos.headObject({
        Bucket: Bucket,
        Region: Region || region,
        Key: Key,
      }, function (err, data) {
        if (data) {
          console.log('对象存在');
          resolve(true);  // 对象存在
        } else if (err && err.statusCode === 404) {
          console.log('对象不存在');
          resolve(false);  // 对象不存在
        } else if (err && err.statusCode === 403) {
          console.log('没有该对象读权限');
          reject(new Error('没有该对象读权限'));
        } else {
          reject(new Error('未知错误'));
        }
      });
    });
  }
}



// 2024年7月29日
// 并发控制函数
export function limitQueueFn(arr, limit, handleFn) {
  const results = []; // 存储最终结果的数组
  let index = 0; // 当前处理的索引
  let runningCount = 0; // 当前正在运行的请求数

  return new Promise((resolve, reject) => {
    function run() {
      if (index >= arr.length) {
        if (runningCount === 0) {
          resolve(results);
        }
        return;
      }

      runningCount++;
      const itemIdx = index++;
      const item = arr[itemIdx];

      handleFn(item)
        .then(result => {
          results[itemIdx] = result;
        })
        .catch(err => {
          reject(err);
        })
        .finally(() => {
          runningCount--;
          run();
        });
    }

    for (let i = 0; i < limit && i < arr.length; i++) {
      run();
    }
  });
}

```