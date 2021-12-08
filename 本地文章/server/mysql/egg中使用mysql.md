# egg.js中使用egg-mysql操作mysql

### 1、安装egg-mysql

```js
npm i egg-mysql --save
```

### 2、在 {app_root}/config/plugin.js 中启用 egg-mysql 插件:

```js
'use strict';
 
/** @type Egg.EggPlugin */
module.exports = {
    //mysql
    mysql:{
        enable:true,
        package:'egg-mysql',
    }
};
```

### 3、配置 mysql 数据库连接地址 {app_root}/config/config.default.js

```js
config.mysql={
  client:{
    host:'192.168.2.135',
    port:'3306',
    user:'root',
    password:'123',
    database:'pc-comp'
  },
};
```



### 4、创建表

```js
await this.app.mysql.query(`
  CREATE TABLE IF NOT EXISTS test_pc(
    id INTEGER PRIMARY KEY AUTO_INCREMENT, 
    name VARCHAR(255),
    deptId INT(11),
    salary FLOAT
  )
`);
```



### 5、删除表

```js
await this.app.mysql.query(`
  DROP TABLE test_pc
`);
```



### 6、插入数据

```js
await this.app.mysql.query(`
  INSERT INTO test_pc (
    name,
    deptId,
    salary
  ) VALUES (
    'dylanLv',
    1,
    9999.00
  )
`);
```



### 7、删除数据

```js
await this.app.mysql.query(`
  DELETE FROM test_pc
    WHERE name = 'dylanLv'
`);
```



### 8、更新数据

```js
await this.app.mysql.query(`
  UPDATE test_pc
  SET
    salary = 10000
  WHERE name = 'dylanLv'
`);
```



### 9、新增字段

```js
await this.app.mysql.query(`
  ALTER TABLE test_pc
  ADD target FLOAT NULL
`);
```



### 10、删除字段

```js
await this.app.mysql.query(`
  ALTER TABLE test_pc
  DROP COLUMN target
`);
```



### 11、重命名表

```js
await this.app.mysql.query(`
  RENAME TABLE test_pc TO test_pc_1
`);
```







