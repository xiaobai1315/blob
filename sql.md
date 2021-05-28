[toc]

### CONCAT 函数

concat函数将多个字段拼接成一个字段

```sql
select CONCAT('姓名：', name) as '新名字' from student s  // 姓名：111
select CONCAT(name, '-', age) FROM student s   // 小王-10
```

