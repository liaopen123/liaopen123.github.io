# flutter DB sqflite

## 使用步骤：

1. 添加依赖:

   ```yaml
     sqflite: ^1.1.7+3
   ```

   

2. 编码:

```dart
import 'package:path/path.dart';  //join()导包
import 'package:sqflite/sqflite.dart';
  static const int _VERSION=1;//数据库版本号
  static const String DB_NAME = "todo.db";//数据库名称
  static Database _database;

	///2.先获取数据库路径,然后与自己定义的数据库名称拼接，形成新的数据库path(String):
    var databasesPath = await getDatabasesPath();
    String path = join(databasesPath, 'demo.db');

	///3.通过现有path 打开数据库
_database =  await openDatabase(path,version: _VERSION,onCreate:(Database db,int version) async{
    //回调方法会返回 数据库对象db和版本号，可以进行初始化控制，也可以进行创建表格控制，个人觉得再此处创建数据库表格不妥当，因为每次开启数据库的时候，都会重复创建数据库，这样会报错(table xxx already exists)。
} );

//剩下的就可以执行对应的sql语句了。
///创建表格:
///'CREATE TABLE TableName (id INTEGER PRIMARY KEY, name TEXT not null, value INTEGER, num REAL)'
///插入数据：
///'INSERT INTO Test233(name, value, num) VALUES("some name2333", 1234, 456.789)'
///查询数据:
///'SELECT * FROM TableName'
///
///

```

