[toc]

---

# 索引失效

1. 范围查询的范围过大
2. 联合索引时违背最左匹配原则
3. 各种语法待试验....面经里写的那些感觉有的不太对，如：
   1. 有or必全有索引;
   2. 复合索引未用左列字段;
   3. like以%开头;
   4. 需要类型转换;
   5. where中索引列有运算;
   6. where中索引列使用了函数;
   7. 如果mysql觉得全表扫描更快时（数据少）;