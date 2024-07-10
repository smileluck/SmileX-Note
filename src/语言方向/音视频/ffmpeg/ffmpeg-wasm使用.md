[toc]

---

# 框架信息

- 官网：https://ffmpegwasm.netlify.app/

- 仓库地址：https://github.com/ffmpegwasm/ffmpeg.wasm





# 异常记录

##  vite/deps/worker.js 

- 异常信息：error http://localhost:5173/@fs/C:/Users/Username/Desktop/react-vite-app/node_modules/.vite/deps/worker.js?type=module&worker_file 

- 解决办法：vite.config.ts文件添加如下配置。 [issue](https://github.com/ffmpegwasm/ffmpeg.wasm/issues/532)

  ```shell
  
  export default defineConfig(configEnv => {
  	return{
          optimizeDeps: {
            exclude: ["@ffmpeg/ffmpeg", "@ffmpeg/util"],
          },
      }
    };
  });
  
  ```

  

