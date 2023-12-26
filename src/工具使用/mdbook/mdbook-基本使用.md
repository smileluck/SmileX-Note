[toc]

---

# 部署github page action

关于mdbook.yml文件记录

```yml
# Sample workflow for building and deploying a mdBook site to GitHub Pages
#
# To get started with mdBook see: https://rust-lang.github.io/mdBook/index.html
#
name: Deploy mdBook site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      MDBOOK_VERSION: 0.4.21
    steps:
      - uses: actions/checkout@v3
      - name: Install mdBook
        run: |
          curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf -y | sh
          rustup update
          cargo install --version ${MDBOOK_VERSION} mdbook
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Build with mdBook
        run: mdbook build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./book

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

# 生成目录
## java
> https://github.com/smileluck/utils-project.git

```java

import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.concurrent.atomic.AtomicInteger;

public class MdBookApplication {

    private final static String path = "D:\project\B.Smile\SmileX-Note\src";

   
    public static void main(String[] args) throws IOException {
        AtomicInteger dirCount = new AtomicInteger();
        AtomicInteger fileCount = new AtomicInteger();
        StringBuilder sb = new StringBuilder();
        Files.walkFileTree(Paths.get(path), new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println("文件夹：" + dir);
                String fileName = dir.getFileName().toString();
                if (!fileName.endsWith("assets")) {
                    String absolutePath = dir.toAbsolutePath().toString();

                    String replace = absolutePath.replace(path, "");
                    String[] split = replace.split("\\\\");

                    int len = split.length - 1;
                    for (int i = 0; i < len; i++) {
                        sb.append("\t");
                    }
                    sb.append("- [" + fileName + "]()\n");


                    dirCount.incrementAndGet();
                }
                return super.preVisitDirectory(dir, attrs);
            }

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println("文件" + file);
                String fileName = file.getFileName().toString();
                String absolutePath = file.toAbsolutePath().toString();
                if (fileName.equalsIgnoreCase("README.md") || fileName.equalsIgnoreCase("SUMMARY.md")) {

                } else if (fileName.endsWith(".md") || fileName.endsWith(".md")) {
                    String replace = absolutePath.replace(path + "\\", "");
                    absolutePath.replace("\\\\", "/");
                    String[] split = replace.split("\\\\");

                    int len = split.length - 1;
                    for (int i = 0; i <= len; i++) {
                        sb.append("\t");
                    }
                    replace = replace.replace("\\", "/");
                    replace = replace.replace(" ", "%20");
                    sb.append("- [" + fileName.substring(0, fileName.length() - 3) + "](" + replace + ")\n");

                    fileCount.incrementAndGet();
                }
                return super.visitFile(file, attrs);
            }
        });
//        System.out.println("dir count:" + dirCount);
//        System.out.println("file count:" + fileCount);
        System.out.println(sb.toString());
    }
}

```


# 踩坑注意

## 子目录访问404问题

在 `SUMMARY.MD` 的路径中，不能使用 `\`，必须得使用 "/"，否则部署的时候子目录访问会404



```markdown
- [糯米类]() 
  -  [糯米糍](foods/nuomici.md) 
```

## 关于[toc]

1. 更换 `action` 的 `mdbook.md文件`。增加 `cargo install mdbook-toc`

   ```yaml
   # Sample workflow for building and deploying a mdBook site to GitHub Pages
   #
   # To get started with mdBook see: https://rust-lang.github.io/mdBook/index.html
   #
   name: Deploy mdBook site to Pages
   
   on:
     # Runs on pushes targeting the default branch
     push:
       branches: ["main"]
   
     # Allows you to run this workflow manually from the Actions tab
     workflow_dispatch:
   
   # Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
   permissions:
     contents: read
     pages: write
     id-token: write
   
   # Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
   # However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
   concurrency:
     group: "pages"
     cancel-in-progress: false
   
   jobs:
     # Build job
     build:
       runs-on: ubuntu-latest
       env:
         MDBOOK_VERSION: 0.4.36
       steps:
         - uses: actions/checkout@v4
         - name: Install mdBook
           run: |
             curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf -y | sh
             rustup update
             cargo install --version ${MDBOOK_VERSION} mdbook
             cargo install mdbook-toc
         - name: Setup Pages
           id: pages
           uses: actions/configure-pages@v4
         - name: Build with mdBook
           run: mdbook build
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
           with:
             path: ./book
   
     # Deployment job
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       needs: build
       steps:
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

2. `book.toml` 增加 自定义指令

   ```toml
   [preprocessor.toc]
   command = "mdbook-toc" # 自定义指令
   marker = "[toc]" # 识别标签
   renderer = ["html"] # 渲染
   ```

   