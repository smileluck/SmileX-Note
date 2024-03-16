[toc]

---

# java调用ffmpeg指令

## Runtime.getRuntime()

```java
Process process = Runtime.getRuntime().exec("ffprobe -version");
int exitCode = process.waitFor();
if (exitCode == 0) {
    logger.info("获取pprobe版本成功！");
} else {
    logger.error("获取pprobe版本失败！");
}
```

## ProcessBuilder

```java
ProcessBuilder processBuilder = new ProcessBuilder().directory(new File(ffmpegPath));

// 这里是个问题点，稍后会说
processBuilder.command("ffprobe", "-version");
// 错误写法
// processBuilder.command("ffprobe -version");

Process process = processBuilder.start();
int exitCode = process.waitFor();
if (exitCode == 0) {
    logger.info("获取pprobe版本成功！");
} else {
    logger.error("获取pprobe版本失败！");
}
```





# 错误记录

## java.io.IOException: error=2, No such file or directory

- 问题描述：在linux系统下，`ProcessBuilder` 执行 `ffprobe -version` 时，会抛出异常提示 `java.io.IOException: error=2, No such file or directory` 。但是实际在系统中执行时没有问题的，但是 Runtime.getRuntime().exec执行时又不会出现问题。

  ```java
  // 此时会抛出异常，
  ProcessBuilder processBuilder = new ProcessBuilder(); processBuilder.command("ffprobe -version");
  Process process = processBuilder.start();
  
  // 此时正常执行
  Runtime.getRuntime().exec("ffprobe -version");
  ```

- 详情分析：让我们分别看一下源码部分差异在哪里

  - ProcessBuilder::command方法。可以看出他是把这个字符串看作一个元素放到 List 里面。

    ```java
    // 相当于：["ffprobe -version"]
    public ProcessBuilder command(String... command) {
        this.command = new ArrayList<>(command.length);
        for (String arg : command)
            this.command.add(arg);
        return this;
    }
    ```

  - Runtime.getRuntime().exec方法。可以看出他是做了切割，然后放到一个String数组的。它的底层也是调用了ProcessBuilder去执行命令的。

    ```java
    // 相当于: ["ffprobe","-version"]
    public Process exec(String command, String[] envp, File dir)
        throws IOException {
        if (command.length() == 0)
            throw new IllegalArgumentException("Empty command");
    
        StringTokenizer st = new StringTokenizer(command);
        String[] cmdarray = new String[st.countTokens()];
        for (int i = 0; st.hasMoreTokens(); i++)
            cmdarray[i] = st.nextToken();
        return exec(cmdarray, envp, dir);
    }
    
     public Process exec(String[] cmdarray, String[] envp, File dir)
         throws IOException {
         return new ProcessBuilder(cmdarray)
             .environment(envp)
             .directory(dir)
             .start();
     }
    
    ```

  - 知道问题出现在关于格式这个问题，不妨大胆推断一下，是不是前者将 `ffprobe -version` 视为一个命令，而不是调用 ffprobe 下的 -version选项。

- 解决办法：将`processBuilder.command("ffprobe -version");`分开 `processBuilder.command("ffprobe","-version");`

  ```java
  processBuilder.command("ffprobe","-version");
  ```

  另外说一句，在window下不两种方式都可以。但有些时候也会出问题，就很奇怪。

## 关于waitFor卡住的问题

- 问题描述：在执行视频转码的过程中，就是如下这条指令，执行到process.waitFor时会卡住，但在cmd执行时很快完成，那么问题点出来哪里呢？

  ```shell
  ffmpeg -y -i D:\ruoyi\uploadPath\2024\03\520373139859968000.mp4 -c:v libx264 -crf 23 -preset slow D:/ruoyi/uploadPath/2024/03/520373139859968000_new.mp4
  ```

- 详情分析：经过尝试，发现该指令会输出大量的日志。WaitFor等待进程完成时， 可能会遇到日志堵塞的问题。这是因为进程的输出缓冲区已满，导致进程无法继续执行。为了避免这个问题，您可以在等待进程完成之前，使用单独的线程读取进程的输出。 

- 解决办法：

  1. 增加了 `-loglevel error` 选项。 使得日志输出很少，输出区不会堆满。

  ```shell
  ffmpeg -loglevel error -y -i D:\ruoyi\uploadPath\2024\03\520373139859968000.mp4 -c:v libx264 -crf 23 -preset slow D:/ruoyi/uploadPath/2024/03/520373139859968000_new.mp4
  ```

  2. 通过创建线程输出。

     ```java
     Process process = new ProcessBuilder(command).start();
     
     // 创建一个线程来读取进程的输出
     Thread outputThread = new Thread(() -> {
         try (BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()))) {
             String line;
             while ((line = reader.readLine()) != null) {
                 System.out.println(line);
             }
         } catch (IOException e) {
             e.printStackTrace();
         }
     });
     
     // 启动线程
     outputThread.start();
     
     // 等待进程完成
     int exitCode = process.waitFor();
     System.out.println("Process exited with code: " + exitCode);
     
     // 等待输出线程完成
     outputThread.join();
     ```

     