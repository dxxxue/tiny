[TOC]
## F&Q
### 1. mysql插入数据字符集报错
- 错误码 : panic error 1366 Incorrect string value
- 原因: mysql有4层，服务器，数据库，表，列。服务器的utf8只能保证数据库使用utf8,而表和列的编码可能不同，
- 操作: alter table <some_table> convert to character set utf8 collate utf8_unicode_ci;
 