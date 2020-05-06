[TOC]
## 深入理解protobuf

### 优点
1. 语言无关，平台无关
2. 高效
3. 扩展性，兼容性好

### 基本语法规则
#### message相关
```proto
message xxx {
    //字段规则：required -> 字段只能且必须出现1次 （proto3已舍弃）
    //字段规则：optional -> 字段可出现0次或1次 (proto3为默认，不用显示填入)
    //字段规则：repeated -> 字段可以出现任意多次(包括0次)
    //类型：int32, int64, sint32, sint64, string, 32-bit...
    //字段编号 0 ~ 536870911
    //字段规则 类型 名称 = 字段编号
}
```
#### import导入
```proto
import "old.proto"  //引用只有message可以复用，但是枚举在proto2不能用在proto3中
```

#### 服务service
```proto
service FooService {
    rpc GetSomething(FooRequest) returns(FooResponse);
}

```

### 生成语法
```shell
protoc --proto_path=IMPORT_PATH --go_out=DST_DIR path/to/file.proto
# IMPORT_PATH 指定在解析导入指令时查找 .proto 文件的目录。 如果省略，则使用当前目录
# --go_out指生成对应的程序，DST_DIR目前文件的目录
# 你必须提供一个或多个 .proto 文件作为输入。可以一次指定多个 .proto 文件。虽然文件是相对于当前目录命名的，但每个文件必须驻留在其中一个 IMPORT_PATH 中，以便编译器可以确定其规范名称。

```
