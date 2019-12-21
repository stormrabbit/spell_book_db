# 说明

## 安装 & 运行 mongodb

1. 升级 `brew`

```
brew update
```

2. 安装 `mongodb`

```
brew install mongodb
```

3. 设定 db 位置

```
mkdir -p /Users/xxx/github/mongodb
```

4. 启动数据库
```
mongod --dbpath /Users/xxx/github/mongodb
```

> 如此启动的 mongodb 并不安全（没设定密码用户权限 and so on），仅做本机调试和练习使用。

## 使用

### 方法一

1. 安装 vscode 插件 `Azure Cosmos DB`。
2. 选择 `"Attach Database Account"` 并输入默认地址 `mongodb://127.0.0.1:27017`。
3. 新建 `.mongodb` 文件，执行命令行操作。
4. [常用命令](https://www.jianshu.com/p/0a52c672ae78)。


### 方法二

1. [Robo 3T](https://robomongo.org/download) 可视化操作。

## 初始化数据

```
node index.js
```

## 建表并导入数据

```
mongoimport  --jsonArray  --host=127.0.0.1  --db dnd_spells --collection spells --file ./mock/spells.json

mongoimport  --jsonArray  --host=127.0.0.1  --db dnd_spells --collection classes --file ./mock/classes.json

mongoimport  --jsonArray  --host=127.0.0.1  --db dnd_spells --collection spells_classes --file ./mock/spells_classes.json

```

## 查询语句

- 查询塑能系法术

```
db.getCollection('spells').find({school: '塑能'})
```

- 统计法师所拥有法术的数量

```
db.getCollection('spells_classes').find({class_name: 'wizard'}).count()
```

- 将 spells_classes 中的 spell_name 替换为 spell_id

```
db.getCollection('spells_classes').update({}, {$rename:{"spell_name":"spell_id"}}, false, true)

// 参数提示：
// 第一个false表示：可选，这个参数的意思是，如果不存在update的记录，true为插入新的记录，默认是false，不插入。
// 第二个true表示：可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。


```

- 使用 classes 表中的 id 代替 spells_classes 表中 classes_id 的值（原 name）

```
db.getCollection('classes').find({}).forEach(
   function(item){                
       db.getCollection('spells_classes').update({"class_id":item.name},{$set:{"class_id":item._id}}, {multi:true})
   }
)

// update 第一个 {} ：条件
// 第二个 {} ：列与值
// 第三个 {} ：update 参数，multi:true 是批量更新
```

- 查询法师拥有的法术 (`$lookup`，3.2 之后版本支持)

```
db.getCollection('spells_classes').aggregate([{
  $lookup: { // 左连接
    from: "spells", // 关联到 spells 表
    localField: "spell_id", // spells_classes 表关联的字段
    foreignField: "_id", // spells 表关联的字段
    as: "spell"
  }
}, {
  $lookup: { // 左连接
    from: "classes", // 关联到 classes 表
    localField: "class_id", // spells_classes 表关联的字段
    foreignField: "_id", // class 表关联的字段
    as: "class"
  }
},{
  $unwind: { // 拆分子数组
    path: "$spell",
    preserveNullAndEmptyArrays: true // 空的数组也拆分
  }
},{
  $unwind: { // 拆分子数组
    path: "$class",
    preserveNullAndEmptyArrays: true // 空的数组也拆分
  }
}
])
```

