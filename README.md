> 文档模板：v0.4.2

<p align="center">
  <h1 align="center"> <code>watermelondb</code> </h1>
</p>

本项目基于 [watermelondb@v0.28.1-0](https://github.com/Nozbe/WatermelonDB) 开发。

该第三方库的仓库已迁移至 Gitcode，且支持直接从 npm 下载，新的包名为：`@react-native-ohos/watermelondb`，版本所属关系如下：

| 三方库名称 | 三方库版本（npm地址） | 发布信息 | 支持RN版本 | Autolink | 编译API版本 | 社区基线版本 | 源码地址 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| @react-native-ohos/watermelondb | [~ 0.29.0](https://www.npmjs.com/package/@react-native-ohos/watermelondb) | [Gitcode Releases](https://gitcode.com/CPF-RN/rntpc_watermelondb/releases) | 0.82.* | 是 | API12+ | v0.28.1-0 | [master](https://gitcode.com/CPF-RN/rntpc_watermelondb/tree/master) |
| @react-native-ohos/watermelondb | [~ 0.28.2（开发中）](https://www.npmjs.com/package/@react-native-ohos/watermelondb) | [Gitcode Releases](https://gitcode.com/CPF-RN/rntpc_watermelondb/releases) | 0.72.* / 0.77.* | 否 | API12+ | v0.28.1-0 | [master](https://gitcode.com/CPF-RN/rntpc_watermelondb/tree/master) |

## 简介

WatermelonDB 是一个高性能的 React Native 响应式数据库框架，可支撑从数百条到数万条记录规模的应用并保持快速流畅。<br/>
它默认离线优先（Offline-first），并采用按需懒加载（Lazy loading）设计，数据只有在真正需要时才从底层 SQLite 数据库加载。<br/>
本库鸿蒙侧基于关系型数据库（RelationalStore）等系统能力实现底层 SQLite 适配，API 与使用方式跨平台保持一致。

## 下载安装

进入到工程目录并输入以下命令：

**npm**

```bash
npm install @react-native-ohos/watermelondb
```

**yarn**

```bash
yarn add @react-native-ohos/watermelondb
```

## Link

| 版本 | 是否支持autolink | RN框架版本 |
| --- | --- | --- |
| ~ 0.29.0 | 是 | 0.82.* |

使用AutoLink的工程需要根据该文档配置，Autolink框架指导文档：https://gitcode.com/CPF-RN/ohos_react_native/blob/main/docs/zh-cn/02-开发/02-开发指南/Autolinking.md

如您使用的版本支持 Autolink，并且工程已接入 Autolink，可跳过ManualLink配置。
<details>
  <summary>ManualLink: 此步骤为手动配置原生依赖项的指导</summary>

首先需要使用 DevEco Studio 打开项目里的 HarmonyOS 工程 `harmony`。

### 1. Overrides RN SDK

为了让工程依赖同一个版本的 RN SDK，需要在工程根目录的 `oh-package.json5` 添加 overrides 字段，指向工程需要使用的 RN SDK 版本。替换的版本既可以是一个具体的版本号，也可以是一个模糊版本，还可以是本地存在的 HAR 包或源码目录。

关于该字段的作用请阅读[官方说明](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#zh-cn_topic_0000001792256137_overrides)

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "~0.82.18" // ohpm 在线版本
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony.har" // 指向本地 har 包的路径
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // 指向源码路径
  }
}
```

### 2. 引入原生端代码

目前有两种方法：

- 通过 har 包引入；
- 直接链接源码。

方法一：通过 har 包引入（推荐）

> [!TIP] har 包位于三方库安装路径的 `harmony` 文件夹下。

打开 `entry/oh-package.json5`，添加以下依赖

```json
"dependencies": {
    "@react-native-ohos/watermelondb": "file:../../node_modules/@react-native-ohos/watermelondb/harmony/watermelondb.har"
  }
```

点击右上角的 `sync` 按钮

或者在命令行终端执行：

```bash
cd entry
ohpm install
```

方法二：直接链接源码

> [!TIP] 如需使用直接链接源码，请参考[直接链接源码说明](https://gitcode.com/CPF-RN/usage-docs/blob/master/zh-cn/link-source-code.md)

### 3. 配置 CMakeLists 和引入 WatermelondbPackage

打开 `entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
set(OH_MODULE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules/@rnoh/react-native-openharmony/src/main/cpp")
set(RNOH_GENERATED_DIR "${CMAKE_CURRENT_SOURCE_DIR}/generated")
set(LOG_VERBOSITY_LEVEL 1)
set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
+ set(OH_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")

set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
add_compile_definitions(WITH_HITRACE_SYSTRACE)

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: manual_package_linking_1
+ add_subdirectory("${OH_MODULES}/@react-native-ohos/watermelondb/src/main/cpp" ./watermelondb)
# RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "${RNOH_GENERATED_DIR}/*.cpp")

add_library(rnoh_app SHARED
    ${GENERATED_CPP_FILES}
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)

target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: manual_package_linking_2
+ target_link_libraries(rnoh_app PUBLIC rnoh_watermelondb)
# RNOH_END: manual_package_linking_2
```

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
#include "RNOH/PackageProvider.h"
+ #include "WatermelondbPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
+     std::make_shared<WatermelondbPackage>(ctx),
    };
}
```

### 4. 在 ArkTs 侧引入 WatermelonDBPackage

打开 `entry/src/main/ets/RNPackagesFactory.ts`，添加：

```diff
  ...
+ import { WatermelonDBPackage } from '@react-native-ohos/watermelondb';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+   new WatermelonDBPackage(ctx)
  ];
}
```
</details>

### 运行

点击右上角的 `sync` 按钮

或者在命令行终端执行：

```bash
cd entry
ohpm install
```

然后编译、运行即可。

## 约束与限制

### 兼容性

本文档内容基于以下版本验证通过：

1. RNOH: 0.82.7; SDK: HarmonyOS 6.0.2 Release SDK; IDE: DevEco Studio 6.0.2.642; ROM: 6.0.0.328;

### 权限要求

无特殊权限要求。

## 使用示例

下面的代码展示了这个库的基本使用场景：

> [!WARNING] 使用时 import 的库名不变。

**jsi: false（默认，未启用 JSI）**

```js
// jsi: false
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet, ScrollView } from 'react-native';
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from '@nozbe/watermelondb/Schema';
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schemaMigrations } from '@nozbe/watermelondb/Schema/migrations';

const userTable = tableName('users');

const userColumns = {
  name: columnName('name'),
  email: columnName('email'),
  age: columnName('age'),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: 'string' },
    { name: userColumns.email, type: 'string', isIndexed: true },
    { name: userColumns.age, type: 'number', isOptional: true },
  ],
});

const userInfoTable = tableName('user_info');
const userInfoColumns = {
  userId: columnName('user_id'),
  address: columnName('address'),
  phone: columnName('phone'),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: 'string', isIndexed: true },
    { name: userInfoColumns.address, type: 'string' },
    { name: userInfoColumns.phone, type: 'string', isOptional: true },
  ],
});

const orderTable = tableName('order');
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName('order_no'), type: 'string', isIndexed: true },
    { name: columnName('user_id'), type: 'string', isIndexed: true },
    { name: columnName('amount'), type: 'number' },
    { name: columnName('status'), type: 'string', defaultValue: 'pending' },
  ],
});

const productTable = tableName('product');
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName('product_name'), type: 'string' },
    { name: columnName('price'), type: 'number' },
    { name: columnName('stock'), type: 'number', defaultValue: 0 },
  ],
});

const tableSchemaMap = {
  tableName: userTableSchema,
  columnName: userInfoTableSchema,
  tableSchema: orderTableSchema,
  appSchema: productTableSchema,
};


const getAppDatabaseSchema = (tableSchema) => {
  return appSchema({
    version: 1,
    tables: [tableSchema],
  });
};

const validateUserColumns = () => {
  const results = [];
  userTableSchema.columnArray.forEach(column => {
    try {
      validateColumnSchema(column);
      results.push({ column: column.name, valid: true });
    } catch (error) {
      results.push({ column: column.name, valid: false, error: error.message });
    }
  });
  console.log(results);
  return results;
};

const initializeDatabase = async (tableType) => {
  const targetTableSchema = tableSchemaMap[tableType] || userTableSchema;
  const adapter = new SQLiteAdapter({
    dbName: `watermelon_${tableType}_db`,
    schema: getAppDatabaseSchema(targetTableSchema),
    jsi: false,
    migrations: schemaMigrations({ migrations: [] }),
  });

  await adapter.initializingPromise;
  return new Database({ adapter, modelClasses: [] });
};

const SchemaExample = () => {
  const [validationResult, setValidationResult] = useState(null);
  const [createResult, setCreateResult] = useState(null);
  const [database, setDatabase] = useState({});


  const handleValidateSchema = () => {
    const results = validateUserColumns();
    setValidationResult(results);
    setCreateResult(null);
  };

  const handleCreateTables = async (tableType) => {
    try {

      if (database[tableType]) {
        const tableName = tableSchemaMap[tableType]?.name || userTable;
        setCreateResult({
          success: true,
          message: `数据库(${tableType})已存在`,
          tables: [tableName],
        });
        return;
      }

      const db = await initializeDatabase(tableType);

      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `数据库表(${tableType})创建成功`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `数据库表(${tableType})创建失败`,
        error: error.message,
      });
    }
  };


  const handleClear = () => {
    setValidationResult(null);
    setCreateResult(null);
    setDatabase({});
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>WatermelonDB 表创建示例</Text>

      <View style={styles.buttons}>
        <Button
          title="使用tableName创建用户表"
          onPress={() => handleCreateTables('tableName')}
          color="#4CAF50"
        />
        <Button
          title="使用columnName创建用户表"
          onPress={() => handleCreateTables('columnName')}
          color="#4CAF50"
        />
        <Button
          title="使用tableSchema创建用户表"
          onPress={() => handleCreateTables('tableSchema')}
          color="#4CAF50"
        />
        <Button
          title="使用appSchema创建用户表"
          onPress={() => handleCreateTables('appSchema')}
          color="#4CAF50"
        />
        <Button
          title="使用validateColumnSchema验证字段定义"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="清空状态" onPress={handleClear} color="#f44336" />
      </View>


      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>字段验证结果</Text>
          {validationResult.every(r => r.valid) ? (
            <Text style={styles.success}>所有字段定义均有效</Text>
          ) : (
            <View>
              <Text style={styles.error}>发现无效字段定义</Text>
              <Text style={styles.error}>
                {' '}
                WatermelonDB 版本过旧，该版本未导出 validateColumnSchema
                函数（该函数是较新版本才对外暴露的
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                字段 {item.column}:{' '}
                {item.valid ? '有效' : `无效 (${item.error})`}
              </Text>
            </View>
          ))}
        </View>
      )}


      {createResult && (
        <View style={styles.result}>
          <Text
            style={[
              styles.resultTitle,
              { color: createResult.success ? 'green' : 'red' },
            ]}>
            {createResult.success ? '操作成功' : '操作失败'}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>已创建表:</Text>
              {createResult.tables.map((table, i) => (
                <Text key={i}>- {table}</Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};


const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  buttons: {
    flexDirection: 'row',
    gap: 10,
    marginBottom: 20,
    flexWrap: 'wrap',
  },
  result: {
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  success: {
    color: 'green',
    marginBottom: 10,
  },
  error: {
    color: 'red',
  },
});

export default SchemaExample;
```

**jsi: true（启用 JSI）**

```js
// jsi: true
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet, ScrollView } from 'react-native';
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from '@nozbe/watermelondb/Schema';
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schemaMigrations } from '@nozbe/watermelondb/Schema/migrations';


const userTable = tableName('users');


const userColumns = {
  name: columnName('name'),
  email: columnName('email'),
  age: columnName('age'),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: 'string' },
    { name: userColumns.email, type: 'string', isIndexed: true },
    { name: userColumns.age, type: 'number', isOptional: true },
  ],
});


const userInfoTable = tableName('user_info');
const userInfoColumns = {
  userId: columnName('user_id'),
  address: columnName('address'),
  phone: columnName('phone'),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: 'string', isIndexed: true },
    { name: userInfoColumns.address, type: 'string' },
    { name: userInfoColumns.phone, type: 'string', isOptional: true },
  ],
});


const orderTable = tableName('order');
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName('order_no'), type: 'string', isIndexed: true },
    { name: columnName('user_id'), type: 'string', isIndexed: true },
    { name: columnName('amount'), type: 'number' },
    { name: columnName('status'), type: 'string', defaultValue: 'pending' },
  ],
});


const productTable = tableName('product');
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName('product_name'), type: 'string' },
    { name: columnName('price'), type: 'number' },
    { name: columnName('stock'), type: 'number', defaultValue: 0 },
  ],
});


const tableSchemaMap = {
  tableName: userTableSchema,
  columnName: userInfoTableSchema,
  tableSchema: orderTableSchema,
  appSchema: productTableSchema,
};


const getAppDatabaseSchema = tableSchema => {
  return appSchema({
    version: 1,
    tables: [tableSchema],
  });
};


const validateUserColumns = () => {
  const results = [];

  userTableSchema.columnArray.forEach(column => {
    try {
      validateColumnSchema(column);
      results.push({ column: column.name, valid: true });
    } catch (error) {
      results.push({ column: column.name, valid: false, error: error.message });
    }
  });
  console.log(results);
  return results;
};


const initializeDatabase = tableType => {

  const targetTableSchema = tableSchemaMap[tableType] || userTableSchema;
  const adapter = new SQLiteAdapter({
    dbName: `watermelonT_${tableType}_db`,
    schema: getAppDatabaseSchema(targetTableSchema),
    jsi: true,
    migrations: schemaMigrations({ migrations: [] }),
  });

  adapter.initializingPromise;
  return new Database({ adapter, modelClasses: [] });
};

const SchemaExampleT = () => {
  const [validationResult, setValidationResult] = useState(null);
  const [createResult, setCreateResult] = useState(null);
  const [database, setDatabase] = useState({});


  const handleValidateSchema = () => {
    const results = validateUserColumns();
    setValidationResult(results);
    setCreateResult(null);
  };


  const handleCreateTables = tableType => {
    try {
      if (database[tableType]) {
        const tableName = tableSchemaMap[tableType]?.name || userTable;
        setCreateResult({
          success: true,
          message: `数据库(${tableType})已存在`,
          tables: [tableName],
        });
        return;
      }

      const db = initializeDatabase(tableType);
      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `数据库表(${tableType})创建成功`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `数据库表(${tableType})创建失败`,
        error: error.message,
      });
    }
  };

  const handleClear = () => {
    setValidationResult(null);
    setCreateResult(null);
    setDatabase({});
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>WatermelonDB 表创建示例</Text>

      <View style={styles.buttons}>
        <Button
          title="使用tableName创建用户表"
          onPress={() => handleCreateTables('tableName')}
          color="#4CAF50"
        />
        <Button
          title="使用columnName创建用户表"
          onPress={() => handleCreateTables('columnName')}
          color="#4CAF50"
        />
        <Button
          title="使用tableSchema创建用户表"
          onPress={() => handleCreateTables('tableSchema')}
          color="#4CAF50"
        />
        <Button
          title="使用appSchema创建用户表"
          onPress={() => handleCreateTables('appSchema')}
          color="#4CAF50"
        />
        <Button
          title="使用validateColumnSchema验证字段定义"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="清空状态" onPress={handleClear} color="#f44336" />
      </View>

      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>字段验证结果</Text>
          {validationResult.every(r => r.valid) ? (
            <Text style={styles.success}>所有字段定义均有效</Text>
          ) : (
            <View>
              <Text style={styles.error}>发现无效字段定义</Text>
              <Text style={styles.error}>
                {' '}
                WatermelonDB 版本过旧，该版本未导出 validateColumnSchema
                函数（该函数是较新版本才对外暴露的
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                字段 {item.column}:{' '}
                {item.valid ? '有效' : `无效 (${item.error})`}
              </Text>
            </View>
          ))}
        </View>
      )}

      {createResult && (
        <View style={styles.result}>
          <Text
            style={[
              styles.resultTitle,
              { color: createResult.success ? 'green' : 'red' },
            ]}>
            {createResult.success ? '操作成功' : '操作失败'}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>已创建表:</Text>
              {createResult.tables.map((table, i) => (
                <Text key={i}>- {table}</Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  buttons: {
    flexDirection: 'row',
    gap: 10,
    marginBottom: 20,
    flexWrap: 'wrap',
  },
  result: {
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  success: {
    color: 'green',
    marginBottom: 10,
  },
  error: {
    color: 'red',
  },
});

export default SchemaExampleT;
```

## 使用说明

**Babel 配置**

本库使用了装饰器等语法特性，需要在工程的 babel.config.js 中启用对应插件：

```js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    ['@babel/plugin-proposal-decorators', { legacy: true }], ['@babel/plugin-proposal-class-properties', { loose: true }],
  ],
};
```

安装 Babel 相关依赖：

```bash

npm install @babel/plugin-proposal-decorators --save-dev

npm install @babel/plugin-transform-class-properties --save-dev
```

## 接口说明

> [!TIP] "Platform"列表示该属性在原三方库上支持的平台。

> [!TIP] "OpenHarmony Support"列为 yes 表示 OpenHarmony平台支持 该属性；no 则表示不支持；partially 表示部分支持。使用方法跨平台一致，效果对标 iOS 或 Android 的效果。

**Collection 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| db | 获取该集合所属的数据库实例引用 | function | no | Android、iOS | Yes |
| find | 根据指定条件查询集合中匹配的记录 | function | yes | Android、iOS | Yes |
| findAndObserve | 查询记录并建立响应式观察以监听数据变更 | function | yes | Android、iOS | Yes |
| create | 在集合中创建并保存新纪录 | function | yes | Android、iOS | Yes |
| prepareCreate | 预创建记录对象但不立即持久化到数据库 | function | yes | Android、iOS | Yes |
| prepareCreateFromDirtyRaw | 从原始未经验证的数据预创建记录对象 | function | yes | Android、iOS | Yes |
| disposableFromDirtyRaw | 基于原始脏数据创建一次性使用的临时记录 | function | yes | Android、iOS | Yes |
| table | 获取该集合对应的数据库表名称 | function | no | Android、iOS | Yes |
| schema | 获取该集合对应的数据表结构定义 | function | no | Android、iOS | Yes |
| experimentalSubscribe | 订阅集合级别的数据变更通知 | function | yes | Android、iOS | Yes |

**Database 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| get&lt;T extends Model&gt; | 根据模型类型和 ID 从数据库获取特定记录 | function | yes | Android、iOS | Yes |
| localStorage | 获取数据库的本地存储适配器用于键值对数据 | function | yes | Android、iOS | Yes |
| batch | 创建批量操作以高效执行多个数据库写入 | function | yes | Android、iOS | Yes |
| write&lt;T&gt; | 在写入事务中执行数据库修改操作 | function | yes | Android、iOS | Yes |
| read&lt;T&gt; | 在只读事务中执行数据库查询操作 | function | yes | Android、iOS | Yes |
| action&lt;T&gt; | 执行需要读写权限的数据库动作 | function | yes | Android、iOS | Yes |
| withChangesForTables | 创建监听指定表数据变更的观察者 | function | yes | Android、iOS | Yes |
| experimentalSubscribe | 数据库注册表变更监听变化 | function | yes | Android、iOS | Yes |
| unsafeResetDatabase | 完全重置数据库并清除所有数据 | function | no | Android、iOS | Yes |
| _ensureInWriter | 确保当前在写入线程中执行操作 | function | yes | Android、iOS | Yes |
| _fatalError | 处理数据库致命错误并终止连接 | function | yes | Android、iOS | Yes |

**Relation 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| get id | 获取关联目标模型的唯一标识符 | function | no | Android、iOS | Yes |
| set id | 设置关联目标模型的标识符值 | function | yes | Android、iOS | Yes |
| fetch | 异步获取关联记录的完整数据承诺 | function | no | Android、iOS | Yes |
| then&lt;U&gt; | 支持 Promise 链式调用，处理异步获取的关联数据 | function | yes | Android、iOS | Yes |
| set | 建立与另一个模型实例的关联关系 | function | yes | Android、iOS | Yes |
| observe | 监听关联模型数据的实时变化 | function | no | Android、iOS | Yes |

**Model 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| associations | 定义模型与其他数据表之间的关联关系映射 | function | no | Android、iOS | Yes |
| prepareMarkAsDeleted | 标记为删除状态 | function | no | Android、iOS | Yes |
| _getChanges | 获取模型变更观察对象 | function | no | Android、iOS | Yes |
| get id | 获取实例的唯一标识符 | function | no | Android、iOS | Yes |
| get syncStatus | 获取数据同步状态 | function | no | Android、iOS | Yes |
| update | 更新模型并立即提交 | function | no | Android、iOS | Yes |
| prepareUpdate | 准备更新 | function | no | Android、iOS | Yes |
| prepareDestroyPermanently | 准备永久销毁但不执行 | function | no | Android、iOS | Yes |
| markAsDeleted | 删除模型并提交 | function | no | Android、iOS | Yes |
| destroyPermanently | 永久删除模型记录 | function | no | Android、iOS | Yes |
| experimentalMarkAsDeleted | 软删除功能 | function | no | Android、iOS | Yes |
| experimentalDestroyPermanently | 永久删除功能 | function | no | Android、iOS | Yes |
| observe | 监听模型数据变化 | function | no | Android、iOS | Yes |
| collection | 获取所属数据集合 | function | no | Android、iOS | Yes |
| get collections | 获取所有集合映射 | function | no | Android、iOS | Yes |
| get database | 获取数据库实例 | function | no | Android、iOS | Yes |
| get db | 获取数据库实例 | function | no | Android、iOS | Yes |
| get asModel | 返回自身实例 | function | no | Android、iOS | Yes |
| callWriter&lt;T&gt; | 在写入线程执行操作 | function | yes | Android、iOS | Yes |
| callReader&lt;T&gt; | 在读取线程执行查询 | function | yes | Android、iOS | Yes |
| subAction&lt;T&gt; | 在事务中执行子操作 | function | yes | Android、iOS | Yes |
| get table | 获取对应数据表名 | function | no | Android、iOS | Yes |
| _prepareCreateFromDirtyRaw | 从脏数据创建模型 | function | yes | Android、iOS | Yes |
| _disposableFromDirtyRaw | 创建一次性模型 | function | yes | Android、iOS | Yes |
| experimentalSubscribe | 实验性订阅模型变更 | function | yes | Android、iOS | Yes |
| _notifyChanged | 内部通知模型变更 | function | no | Android、iOS | Yes |
| _notifyDestroyed | 内部通知模型销毁 | function | no | Android、iOS | Yes |
| _getRaw | 内部获取原始字段值 | function | yes | Android、iOS | Yes |
| _setRaw | 内部设置字段值并标记变更 | function | yes | Android、iOS | Yes |
| _dangerouslySetRawWithoutMarkingColumnChange | 内部不安全设置字段值 | function | yes | Android、iOS | Yes |
| __ensureCanSetRaw | 内部确保可设置原始值 | function | no | Android、iOS | Yes |
| __ensureNotDisposable | 内部确保非一次性实例 | function | yes | Android、iOS | Yes |

**Query 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| pipe&lt;T&gt; | 对查询结果进行链式转换操作 | function | no | Android、iOS | Yes |
| fetch | 立即执行查询获取结果 | function | no | Android、iOS | Yes |
| then&lt;U&gt; | 支持 Promise 链式调用查询结果 | function | no | Android、iOS | Yes |
| observe | 监听查询结果的实时变化 | function | no | Android、iOS | Yes |
| observeWithColumns | 监听特定字段变化的查询结果 | function | yes | Android、iOS | Yes |
| fetchCount | 获取查询结果的数量统计 | function | yes | Android、iOS | Yes |
| get count | 获取查询结果总数 | function | no | Android、iOS | Yes |
| observeCount | 监听查询结果数量的变化 | function | no | Android、iOS | Yes |
| fetchIds | 获取查询结果的 ID 列表 | function | no | Android、iOS | Yes |
| unsafeFetchRaw | 不安全地获取原始查询数据 | function | no | Android、iOS | Yes |
| experimentalSubscribe | 实验性查询订阅功能 | function | yes | Android、iOS | Yes |
| experimentalSubscribeWithColumns | 实验性字段变化订阅 | function | yes | Android、iOS | Yes |
| experimentalSubscribeToCount | 实验性数量变化订阅 | function | yes | Android、iOS | Yes |
| markAllAsDeleted | 标记所有查询结果为软删除 | function | no | Android、iOS | Yes |
| destroyAllPermanently | 永久删除所有查询结果 | function | no | Android、iOS | Yes |
| get modelClass | 获取查询对应的模型类 | function | no | Android、iOS | Yes |
| get table | 获取查询的主表名称 | function | no | Android、iOS | Yes |
| get secondaryTables | 获取查询涉及的关联表 | function | no | Android、iOS | Yes |
| get allTables | 获取查询所有相关表 | function | no | Android、iOS | Yes |
| get associations | 获取查询的关联关系 | function | no | Android、iOS | Yes |
| serialize | 序列化查询为可传输格式 | function | no | Android、iOS | Yes |

**Schema 类**

| 名称 | 描述 | 类型 | 必填 | 平台 | OpenHarmony平台支持 |
| --- | --- | --- | --- | --- | --- |
| tableName&lt;T extends Model&gt; | 定义数据库表的名称 | function | yes | Android、iOS | Yes |
| columnName | 定义数据库字段的名称 | function | yes | Android、iOS | Yes |
| appSchema | 定义整个应用数据库的结构模式 | function | yes | Android、iOS | Yes |
| validateColumnSchema | 验证字段模式定义的合法性 | function | yes | Android、iOS | Yes |
| tableSchema | 定义单个数据表的结构和字段配置 | function | yes | Android、iOS | Yes |

## 遗留问题

无

## 其他

- get id中get为JavaScript基础语法，获取关联目标模型的唯一标识符，通过 .id 形式调用
- set id中set为JavaScript基础语法，设置关联目标模型的标识符值，通过 .id = 'xxx' 形式调用

## 目录结构

````
/WatermelonDB  # 项目根目录
├── src                           # RN 代码（JS 源码）
│   ├── Collection                # 集合（Collection）实现
│   ├── Database                  # 数据库实现
│   ├── DatabaseProvider          # 数据库 Provider 组件
│   ├── Model                     # 模型（Model）实现
│   ├── Query                     # 查询（Query）实现
│   ├── QueryDescription          # 查询条件描述
│   ├── RawRecord                 # 原始记录
│   ├── Relation                  # 关联（Relation）实现
│   ├── Schema                    # 数据库模式定义（appSchema/tableSchema 等）
│   ├── adapters                  # 数据库适配器（sqlite、lokijs 等）
│   ├── decorators                # 装饰器（column、children、action 等）
│   ├── hooks                     # React Hooks
│   ├── observation               # 响应式观察实现
│   ├── react                     # React 组件封装（DatabaseProvider、withDatabase 等）
│   ├── sync                      # 数据同步实现
│   ├── utils                     # 工具函数
│   └── index.js                  # 入口文件
├── native                        # 原生适配代码
│   ├── android                   # Android 原生实现
│   ├── android-jsi               # Android JSI 实现
│   ├── ios                       # iOS 原生实现
│   ├── shared                    # 跨平台共享 C++ 实现
│   └── windows                   # Windows 原生实现
├── examples                      # 示例工程
├── doc                           # 旧版本归档文档
├── docs-website                  # 文档站点（Docusaurus）
├── assets                        # 静态资源
├── scripts                       # 构建与测试脚本
├── flow-typed                    # Flow 类型定义
├── package.json                  # 包配置文件
├── LICENSE                       # 开源协议文件
├── README.md                     # 中文安装使用方法
└── README_en.md                  # 英文安装使用方法
````

## 贡献代码

使用过程中发现任何问题都可以提交 [Issue](https://gitcode.com/CPF-RN/rntpc_watermelondb/issues)，当然，也非常欢迎提交 [PR](https://gitcode.com/CPF-RN/rntpc_watermelondb/pulls) 。

## 开源协议

本项目基于 [The MIT License (MIT)](https://github.com/Nozbe/WatermelonDB/blob/master/LICENSE) ，请自由地享受和参与开源。
