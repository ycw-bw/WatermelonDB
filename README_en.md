> Document Template: v0.4.2

<p align="center">
  <h1 align="center"> <code>watermelondb</code> </h1>
</p>

This project is based on [watermelondb@v0.28.1-0](https://github.com/Nozbe/WatermelonDB).

This third-party library has been migrated to Gitcode and is now available for direct download from npm, the new package name is: `@react-native-ohos/watermelondb`, the version correspondence details are as follows:

| Name | Version(Npm Address) | Release Information | Supported RN Version | Supported Autolink | Compile API Version | Community Baseline Version | Source code address |
| --- | --- | --- | --- | --- | --- | --- | --- |
| @react-native-ohos/watermelondb | [~ 0.29.0](https://www.npmjs.com/package/@react-native-ohos/watermelondb) | [Gitcode Releases](https://gitcode.com/CPF-RN/rntpc_watermelondb/releases) | 0.82.* | Yes | API12+ | v0.28.1-0 | [master](https://gitcode.com/CPF-RN/rntpc_watermelondb/tree/master) |
| @react-native-ohos/watermelondb | [~ 0.28.2 (In development)](https://www.npmjs.com/package/@react-native-ohos/watermelondb) | [Gitcode Releases](https://gitcode.com/CPF-RN/rntpc_watermelondb/releases) | 0.72.* / 0.77.* | No | API12+ | v0.28.1-0 | [master](https://gitcode.com/CPF-RN/rntpc_watermelondb/tree/master) |

## Introduction

WatermelonDB is a high-performance reactive database framework for React Native, designed to keep apps fast while scaling from hundreds to tens of thousands of records.<br/>
It is offline-first by default and uses lazy loading on demand - data is loaded from the underlying SQLite database only when it is actually needed.<br/>
On the HarmonyOS side, this library implements the underlying SQLite adaptation based on system capabilities such as the relational database (RelationalStore), keeping the API and usage consistent across platforms.

## Installation

Go to the project directory and execute the following instruction:

**npm**

```bash
npm install @react-native-ohos/watermelondb
```

**yarn**

```bash
yarn add @react-native-ohos/watermelondb
```

## Link

| Version | Supported Autolink | Supported RN Version |
| --- | --- | --- |
| ~ 0.29.0 | Yes | 0.82.* |

Projects using AutoLink need to be configured according to this document, AutoLink framework guide: https://gitcode.com/CPF-RN/ohos_react_native/blob/main/docs/en/02-development/02-development-guide/autolinking.md

If the version you are using supports Autolink and the project has integrated Autolink, you can skip the ManualLink configuration.
<details>
  <summary>ManualLink: This step provides guidance for manually configuring native dependencies.</summary>

Open the `harmony` directory of the OpenHarmony project in DevEco Studio.

### 1. Overrides RN SDK

To ensure the project relies on the same version of the RN SDK, you need to add an `overrides` field in the project's root `oh-package.json5` file, specifying the RN SDK version to be used. The replacement version can be a specific version number, a semver range, or a locally available HAR package or source directory.

For more information about the purpose of this field, please refer to the [official documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#en-us_topic_0000001792256137_overrides).

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony": "~0.82.18" // ohpm version
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony.har" // a locally available HAR package
    // "@rnoh/react-native-openharmony" : "./react_native_openharmony" // source code directory
  }
}
```

### 2. Introducing Native Code

Currently, two methods are available:

- Use the HAR file.
- Directly link to the source code.

Method 1 (recommended): Use the HAR file.

> [!TIP] The HAR file is stored in the `harmony` directory in the installation path of the third-party library.

Open `entry/oh-package.json5` file and add the following dependencies:

```json
"dependencies": {
    "@react-native-ohos/watermelondb": "file:../../node_modules/@react-native-ohos/watermelondb/harmony/watermelondb.har"
  }
```

Click the `sync` button in the upper right corner.

Alternatively, run the following instruction on the terminal:

```bash
cd entry
ohpm install
```

Method 2: Directly link to the source code.

> [!TIP] For details, see [Directly Linking Source Code](https://gitcode.com/CPF-RN/usage-docs/blob/master/en/link-source-code.md).

### 3. Configuring CMakeLists and Introducing WatermelondbPackage

Open `entry/src/main/cpp/CMakeLists.txt` and add the following code:

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

Open `entry/src/main/cpp/PackageProvider.cpp` and add the following code:

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

### 4. Introducing WatermelonDBPackage to ArkTS

Open `entry/src/main/ets/RNPackagesFactory.ts` and add the following code:

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

### Running

Click the `sync` button in the upper right corner.

Alternatively, run the following instruction on the terminal:

```bash
cd entry
ohpm install
```

Then build and run the code.

## Constraints

### Compatibility

This document is verified based on the following versions:

1. RNOH: 0.82.7; SDK: HarmonyOS 6.0.2 Release SDK; IDE: DevEco Studio 6.0.2.642; ROM: 6.0.0.328;

### Permission Requirements

No special permission is required.

## Example

The following code shows the basic use scenario of the repository:

> [!WARNING] The name of the imported repository remains unchanged.

**jsi: false (default, JSI not enabled)**

```js
// jsi: false
import React, { useState } from "react";
import { View, Text, Button, StyleSheet, ScrollView } from "react-native";
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from "@nozbe/watermelondb/Schema";
import { Database } from "@nozbe/watermelondb";
import SQLiteAdapter from "@nozbe/watermelondb/adapters/sqlite";
import { schemaMigrations } from "@nozbe/watermelondb/Schema/migrations";

const userTable = tableName("users");

const userColumns = {
  name: columnName("name"),
  email: columnName("email"),
  age: columnName("age"),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: "string" },
    { name: userColumns.email, type: "string", isIndexed: true },
    { name: userColumns.age, type: "number", isOptional: true },
  ],
});

const userInfoTable = tableName("user_info");
const userInfoColumns = {
  userId: columnName("user_id"),
  address: columnName("address"),
  phone: columnName("phone"),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: "string", isIndexed: true },
    { name: userInfoColumns.address, type: "string" },
    { name: userInfoColumns.phone, type: "string", isOptional: true },
  ],
});

const orderTable = tableName("order");
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName("order_no"), type: "string", isIndexed: true },
    { name: columnName("user_id"), type: "string", isIndexed: true },
    { name: columnName("amount"), type: "number" },
    { name: columnName("status"), type: "string", defaultValue: "pending" },
  ],
});

const productTable = tableName("product");
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName("product_name"), type: "string" },
    { name: columnName("price"), type: "number" },
    { name: columnName("stock"), type: "number", defaultValue: 0 },
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
  userTableSchema.columnArray.forEach((column) => {
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
          message: `The database (${tableType}) already exists`,
          tables: [tableName],
        });
        return;
      }

      const db = await initializeDatabase(tableType);
      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `The database table (${tableType}) has been created successfully`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `The database table (${tableType}) failed to be created`,
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
      <Text style={styles.title}>WatermelonDB Table Creation Example</Text>

      <View style={styles.buttons}>
        <Button
          title="Create the user table using tableName"
          onPress={() => handleCreateTables("tableName")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using columnName"
          onPress={() => handleCreateTables("columnName")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using tableSchema"
          onPress={() => handleCreateTables("tableSchema")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using appSchema"
          onPress={() => handleCreateTables("appSchema")}
          color="#4CAF50"
        />
        <Button
          title="Validate field definitions using validateColumnSchema"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="Clear state" onPress={handleClear} color="#f44336" />
      </View>

      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>Field Validation Result</Text>
          {validationResult.every((r) => r.valid) ? (
            <Text style={styles.success}>All field definitions are valid</Text>
          ) : (
            <View>
              <Text style={styles.error}>Invalid field definitions found</Text>
              <Text style={styles.error}>
                WatermelonDB version is too old, the validateColumnSchema
                function is not exported in this version (this function is only
                exposed in newer versions)
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                Field {item.column}:{" "}
                {item.valid ? "Valid" : `Invalid (${item.error})`}
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
              { color: createResult.success ? "green" : "red" },
            ]}
          >
            {createResult.success ? "Operation Successful" : "Operation Failed"}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>Created tables:</Text>
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
    backgroundColor: "#f5f5f5",
  },
  title: {
    fontSize: 18,
    fontWeight: "bold",
    marginBottom: 20,
    textAlign: "center",
  },
  buttons: {
    flexDirection: "row",
    gap: 10,
    marginBottom: 20,
    flexWrap: "wrap",
  },
  result: {
    padding: 15,
    backgroundColor: "white",
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: "bold",
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: "#eee",
  },
  success: {
    color: "green",
    marginBottom: 10,
  },
  error: {
    color: "red",
  },
});

export default SchemaExample;
```

**jsi: true (with JSI enabled)**

```js
// jsi: true
import React, { useState } from "react";
import { View, Text, Button, StyleSheet, ScrollView } from "react-native";
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from "@nozbe/watermelondb/Schema";
import { Database } from "@nozbe/watermelondb";
import SQLiteAdapter from "@nozbe/watermelondb/adapters/sqlite";
import { schemaMigrations } from "@nozbe/watermelondb/Schema/migrations";

const userTable = tableName("users");

const userColumns = {
  name: columnName("name"),
  email: columnName("email"),
  age: columnName("age"),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: "string" },
    { name: userColumns.email, type: "string", isIndexed: true },
    { name: userColumns.age, type: "number", isOptional: true },
  ],
});

const userInfoTable = tableName("user_info");
const userInfoColumns = {
  userId: columnName("user_id"),
  address: columnName("address"),
  phone: columnName("phone"),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: "string", isIndexed: true },
    { name: userInfoColumns.address, type: "string" },
    { name: userInfoColumns.phone, type: "string", isOptional: true },
  ],
});

const orderTable = tableName("order");
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName("order_no"), type: "string", isIndexed: true },
    { name: columnName("user_id"), type: "string", isIndexed: true },
    { name: columnName("amount"), type: "number" },
    { name: columnName("status"), type: "string", defaultValue: "pending" },
  ],
});

const productTable = tableName("product");
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName("product_name"), type: "string" },
    { name: columnName("price"), type: "number" },
    { name: columnName("stock"), type: "number", defaultValue: 0 },
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
  userTableSchema.columnArray.forEach((column) => {
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

const initializeDatabase = (tableType) => {
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

  const handleCreateTables = (tableType) => {
    try {
      if (database[tableType]) {
        const tableName = tableSchemaMap[tableType]?.name || userTable;
        setCreateResult({
          success: true,
          message: `The database (${tableType}) already exists`,
          tables: [tableName],
        });
        return;
      }

      const db = initializeDatabase(tableType);
      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `The database table (${tableType}) has been created successfully`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `The database table (${tableType}) failed to be created`,
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
      <Text style={styles.title}>WatermelonDB Table Creation Example</Text>

      <View style={styles.buttons}>
        <Button
          title="Create the user table using tableName"
          onPress={() => handleCreateTables("tableName")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using columnName"
          onPress={() => handleCreateTables("columnName")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using tableSchema"
          onPress={() => handleCreateTables("tableSchema")}
          color="#4CAF50"
        />
        <Button
          title="Create the user table using appSchema"
          onPress={() => handleCreateTables("appSchema")}
          color="#4CAF50"
        />
        <Button
          title="Validate field definitions using validateColumnSchema"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="Clear state" onPress={handleClear} color="#f44336" />
      </View>

      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>Field Validation Result</Text>
          {validationResult.every((r) => r.valid) ? (
            <Text style={styles.success}>All field definitions are valid</Text>
          ) : (
            <View>
              <Text style={styles.error}>Invalid field definitions found</Text>
              <Text style={styles.error}>
                WatermelonDB version is too old, the validateColumnSchema
                function is not exported in this version (this function is only
                exposed in newer versions)
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                Field {item.column}:{" "}
                {item.valid ? "Valid" : `Invalid (${item.error})`}
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
              { color: createResult.success ? "green" : "red" },
            ]}
          >
            {createResult.success ? "Operation Successful" : "Operation Failed"}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>Created tables:</Text>
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
    backgroundColor: "#f5f5f5",
  },
  title: {
    fontSize: 18,
    fontWeight: "bold",
    marginBottom: 20,
    textAlign: "center",
  },
  buttons: {
    flexDirection: "row",
    gap: 10,
    marginBottom: 20,
    flexWrap: "wrap",
  },
  result: {
    padding: 15,
    backgroundColor: "white",
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: "bold",
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: "#eee",
  },
  success: {
    color: "green",
    marginBottom: 10,
  },
  error: {
    color: "red",
  },
});

export default SchemaExampleT;
```

## How to Use

**Babel Configuration**

This library uses syntax features such as decorators. You need to enable the corresponding plugins in the babel.config.js file of the project:

```js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    ['@babel/plugin-proposal-decorators', { legacy: true }], ['@babel/plugin-proposal-class-properties', { loose: true }],
  ],
};
```

Install dependencies related to Babel:

```bash

npm install @babel/plugin-proposal-decorators --save-dev

npm install @babel/plugin-transform-class-properties --save-dev
```

## Available APIs

> [!TIP] The **Platform** column indicates the platform where the properties are supported in the original third-party library.

> [!TIP] If the value of **HarmonyOS Support** is **yes**, it means that the OpenHarmony platform supports this property; **no** means the opposite; **partially** means some capabilities of this property are supported. The usage method is the same on different platforms and the effect is the same as that of iOS or Android.

**Collection class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| db | Get the reference to the database instance that this collection belongs to | function | no | Android, iOS | Yes |
| find | Query for matching records in the collection based on specified conditions | function | yes | Android, iOS | Yes |
| findAndObserve | Query records and establish responsive observation to monitor data changes | function | yes | Android, iOS | Yes |
| create | Create and save new records in the collection | function | yes | Android, iOS | Yes |
| prepareCreate | Pre-create a record object without immediately persisting it to the database | function | yes | Android, iOS | Yes |
| prepareCreateFromDirtyRaw | Pre-create a record object from raw unvalidated data | function | yes | Android, iOS | Yes |
| disposableFromDirtyRaw | Create a one-time temporary record based on raw dirty data | function | yes | Android, iOS | Yes |
| table | Get the name of the database table corresponding to this collection | function | no | Android, iOS | Yes |
| schema | Get the data table structure definition corresponding to this collection | function | no | Android, iOS | Yes |
| experimentalSubscribe | Subscribe to collection-level data change notifications | function | yes | Android, iOS | Yes |

**Database class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| get&lt;T extends Model&gt; | Retrieve a specific record from the database according to the model class and ID | function | yes | Android, iOS | Yes |
| localStorage | Get the local storage adapter of the database for key-value pair data | function | yes | Android, iOS | Yes |
| batch | Create batch operations to efficiently perform multiple database writes | function | yes | Android, iOS | Yes |
| write&lt;T&gt; | Perform database modification operations in a write transaction | function | yes | Android, iOS | Yes |
| read&lt;T&gt; | Perform database query operations in a read-only transaction | function | yes | Android, iOS | Yes |
| action&lt;T&gt; | Perform database actions that require read and write permissions | function | yes | Android, iOS | Yes |
| withChangesForTables | Create an observer that monitors data changes in specified tables | function | yes | Android, iOS | Yes |
| experimentalSubscribe | Subscribe to data change notifications at the database level | function | yes | Android, iOS | Yes |
| unsafeResetDatabase | Completely reset the database and clear all data | function | no | Android, iOS | Yes |
| _ensureInWriter | Ensure that the current operation is executed in the write thread | function | yes | Android, iOS | Yes |
| _fatalError | Handle database fatal errors and terminate the connection | function | yes | Android, iOS | Yes |

**Relation class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| get id | Get the unique identifier of the associated target model | function | no | Android, iOS | Yes |
| set id | Set the identifier value of the associated target model | function | yes | Android, iOS | Yes |
| fetch | Asynchronously obtain the promise of complete data of the associated record | function | no | Android, iOS | Yes |
| then&lt;U&gt; | Support Promise chain calls to process asynchronously obtained associated data | function | yes | Android, iOS | Yes |
| set | Establish an association relationship with another model instance | function | yes | Android, iOS | Yes |
| observe | Monitor real-time changes in associated model data | function | no | Android, iOS | Yes |

**Model class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| associations | Define the association relationship mapping between the model and other data tables | function | no | Android, iOS | Yes |
| prepareMarkAsDeleted | Mark as deleted status | function | no | Android, iOS | Yes |
| _getChanges | Get the model change observation object | function | no | Android, iOS | Yes |
| get id | Get the unique identifier of the instance | function | no | Android, iOS | Yes |
| get syncStatus | Get the data synchronization status | function | no | Android, iOS | Yes |
| update | Update the model and submit immediately | function | no | Android, iOS | Yes |
| prepareUpdate | Prepare for update | function | no | Android, iOS | Yes |
| prepareDestroyPermanently | Prepare for permanent destruction without execution | function | no | Android, iOS | Yes |
| markAsDeleted | Delete the model and submit | function | no | Android, iOS | Yes |
| destroyPermanently | Permanently delete model records | function | no | Android, iOS | Yes |
| experimentalMarkAsDeleted | Soft delete function | function | no | Android, iOS | Yes |
| experimentalDestroyPermanently | Permanent delete function | function | no | Android, iOS | Yes |
| observe | Monitor model data changes | function | no | Android, iOS | Yes |
| collection | Get the associated data set | function | no | Android, iOS | Yes |
| get collections | Get all collection mappings | function | no | Android, iOS | Yes |
| get database | Get the database instance | function | no | Android, iOS | Yes |
| get db | Get the database instance | function | no | Android, iOS | Yes |
| get asModel | Return its own instance | function | no | Android, iOS | Yes |
| callWriter&lt;T&gt; | Perform operations in the write thread | function | yes | Android, iOS | Yes |
| callReader&lt;T&gt; | Perform queries in the read thread | function | yes | Android, iOS | Yes |
| subAction&lt;T&gt; | Perform sub-operations in a transaction | function | yes | Android, iOS | Yes |
| get table | Get the corresponding data table name | function | no | Android, iOS | Yes |
| _prepareCreateFromDirtyRaw | Create a model from dirty data | function | yes | Android, iOS | Yes |
| _disposableFromDirtyRaw | Create a one-time model | function | yes | Android, iOS | Yes |
| experimentalSubscribe | Experimental subscription to model changes | function | yes | Android, iOS | Yes |
| _notifyChanged | Internally notify model changes | function | no | Android, iOS | Yes |
| _notifyDestroyed | Internally notify model destruction | function | no | Android, iOS | Yes |
| _getRaw | Internally get the original field value | function | yes | Android, iOS | Yes |
| _setRaw | Internally set field values and mark changes | function | yes | Android, iOS | Yes |
| _dangerouslySetRawWithoutMarkingColumnChange | Internally unsafely set field values | function | yes | Android, iOS | Yes |
| __ensureCanSetRaw | Internally ensure that raw values can be set | function | no | Android, iOS | Yes |
| __ensureNotDisposable | Internally ensure non-disposable instances | function | yes | Android, iOS | Yes |

**Query class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| pipe&lt;T&gt; | Perform chain conversion operations on query results | function | no | Android, iOS | Yes |
| fetch | Execute the query immediately to get results | function | no | Android, iOS | Yes |
| then&lt;U&gt; | Support Promise chain calls for query results | function | no | Android, iOS | Yes |
| observe | Monitor real-time changes in query results | function | no | Android, iOS | Yes |
| observeWithColumns | Monitor query results of specific field changes | function | yes | Android, iOS | Yes |
| fetchCount | Get the quantity statistics of query results | function | yes | Android, iOS | Yes |
| get count | Get the total number of query results | function | no | Android, iOS | Yes |
| observeCount | Monitor changes in the number of query results | function | no | Android, iOS | Yes |
| fetchIds | Get the ID list of query results | function | no | Android, iOS | Yes |
| unsafeFetchRaw | Unsafely get raw query data | function | no | Android, iOS | Yes |
| experimentalSubscribe | Experimental query subscription function | function | yes | Android, iOS | Yes |
| experimentalSubscribeWithColumns | Experimental field change subscription | function | yes | Android, iOS | Yes |
| experimentalSubscribeToCount | Experimental quantity change subscription | function | yes | Android, iOS | Yes |
| markAllAsDeleted | Mark all query results as soft deleted | function | no | Android, iOS | Yes |
| destroyAllPermanently | Permanently delete all query results | function | no | Android, iOS | Yes |
| get modelClass | Get the model class corresponding to the query | function | no | Android, iOS | Yes |
| get table | Get the main table name of the query | function | no | Android, iOS | Yes |
| get secondaryTables | Get the associated tables involved in the query | function | no | Android, iOS | Yes |
| get allTables | Get all tables related to the query | function | no | Android, iOS | Yes |
| get associations | Get the association relationships of the query | function | no | Android, iOS | Yes |
| serialize | Serialize the query into a transferable format | function | no | Android, iOS | Yes |

**Schema class**

| Name | Description | Type | Required | Platform | OpenHarmony Platform Support |
| --- | --- | --- | --- | --- | --- |
| tableName&lt;T extends Model&gt; | Define the name of the database table | function | yes | Android, iOS | Yes |
| columnName | Define the name of the database field | function | yes | Android, iOS | Yes |
| appSchema | Define the structural schema of the entire application database | function | yes | Android, iOS | Yes |
| validateColumnSchema | Verify the validity of the field schema definition | function | yes | Android, iOS | Yes |
| tableSchema | Define the structure and field configuration of a single data table | function | yes | Android, iOS | Yes |

## Known Issues

None

## Other

- get id: get is a basic JavaScript syntax used to retrieve the unique identifier of the associated target model, which is invoked in the form of .id.
- set id: set is a basic JavaScript syntax used to set the value of the identifier for the associated target model, which is invoked in the form of .id = 'xxx'.

## Directory Structure

````
/WatermelonDB  # Project root directory
├── src                           # React Native code (JS source)
│   ├── Collection                # Collection implementation
│   ├── Database                  # Database implementation
│   ├── DatabaseProvider          # Database Provider component
│   ├── Model                     # Model implementation
│   ├── Query                     # Query implementation
│   ├── QueryDescription          # Query condition description
│   ├── RawRecord                 # Raw record
│   ├── Relation                  # Relation implementation
│   ├── Schema                    # Database schema definitions (appSchema/tableSchema, etc.)
│   ├── adapters                  # Database adapters (sqlite, lokijs, etc.)
│   ├── decorators                # Decorators (column, children, action, etc.)
│   ├── hooks                     # React Hooks
│   ├── observation               # Reactive observation implementation
│   ├── react                     # React wrappers (DatabaseProvider, withDatabase, etc.)
│   ├── sync                      # Data synchronization implementation
│   ├── utils                     # Utility functions
│   └── index.js                  # Entry file
├── native                        # Native adaptation code
│   ├── android                   # Android native implementation
│   ├── android-jsi               # Android JSI implementation
│   ├── ios                       # iOS native implementation
│   ├── shared                    # Cross-platform shared C++ implementation
│   └── windows                   # Windows native implementation
├── examples                      # Example projects
├── doc                           # Archived documentation of old versions
├── docs-website                  # Documentation website (Docusaurus)
├── assets                        # Static resources
├── scripts                       # Build and test scripts
├── flow-typed                    # Flow type definitions
├── package.json                  # Package configuration file
├── LICENSE                       # License file
├── README.md                     # Chinese installation and usage instructions
└── README_en.md                  # English installation and usage instructions
````

## How to Contribute

If you find any problem during the use, you can submit an [Issue](https://gitcode.com/CPF-RN/rntpc_watermelondb/issues). Of course, we also welcome you to submit a [PR](https://gitcode.com/CPF-RN/rntpc_watermelondb/pulls).

## License

This project is based on [The MIT License (MIT)](https://github.com/Nozbe/WatermelonDB/blob/master/LICENSE). Feel free to enjoy and participate in open source.
