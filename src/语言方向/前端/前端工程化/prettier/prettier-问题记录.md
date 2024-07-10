[toc]

---

# SyntaxError: File appears to be binary. (1:1)
- 异常信息：在执行  `npx prettier --writer .` 时，出现如下异常信息

  ```shell
  components/index.ts: SyntaxError: File appears to be binary. (1:1)
  [error] > 1 | ��
  ```

- 解决办法：排查发现是字符编码问题，将从UTF-16LF 改为 UTF-8即可

