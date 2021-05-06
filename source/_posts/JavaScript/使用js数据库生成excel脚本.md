---
title: 使用JS将MySQL数据库表信息导出Excel
typora-copy-images-to: ./使用js数据库生成excel脚本
typora-root-url: ./使用js数据库生成excel脚本
updated:
tags:
  - JavaScript
categories:
  - 小东西
---

# 使用JS将MySQL数据库表信息导出Excel

## 完成效果：

![效果图](/image-20210412114322955.png)

<!--more-->

## 0、依赖：

```json
"dependencies": {
    "knex": "^0.21.5",
    "mysql": "^2.18.1",
    "xlsx": "^0.16.9",
    "xlsx-style": "^0.8.13"
  }
```

这里使用knex处理数据库，直接用mysql包也可以

xlsx就是大名鼎鼎的[sheet.js](https://github.com/SheetJS/sheetjs)，可以非常好的处理excel的文本内容，但是对于样式的修改需要pro版本

[xlsx-style](https://github.com/protobi/js-xlsx)是根据xlsx开发的处理xlsx样式的开源工具

使用npm工具并将三个依赖导入，就不过多赘述了

## 1、使用第三方工具

```javascript
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host: '127.0.0.1',
    user: 'root',
    password: '123456',
    database: 'information_schema'
  }
})

// 将两个进行区分，处理时用xlsx提供的函数，导出时用xlsx-style提供的函数
const XLSX = require('xlsx')
const XLSX_STYLE = require('xlsx-style')
```

## 2、获取数据库中字段信息

数据库中相关信息 都在`information_schema`表中，写一个简单的sql获取就可以了。

```javascript
const asyncGetData = async () => {
  return await knex('COLUMNS')
    .where({
      // 需要导出的数据库名
      TABLE_SCHEMA: 'oa'
    })
    .select('TABLE_NAME as 表名',
      'COLUMN_NAME as 字段名',
      'DATA_TYPE as 数据类型',
      'CHARACTER_MAXIMUM_LENGTH as 字符长度',
      'NUMERIC_PRECISION as 数字长度',
      'NUMERIC_SCALE as 小数位数',
      'IS_NULLABLE as 允许空',
      'EXTRA as 自增策略',
      'COLUMN_DEFAULT as 默认值',
      'COLUMN_COMMENT as 备注')
}
```

## 3、创建workBook与workSheet

workBook对象是xlsx用来操作的基本对象，workSheet对应xlsx文件中的每个表；

```javascript
const workBook = XLSX.utils.book_new()
const workSheet = { '!merges': [], '!cols': [{ wch: 18 }, { wch: 18 }, {}, {}, {}, {}, {}, { wch: 14 }, {}, { wch: 50 }] }
```

workSheet中

- !merges 数组中每个对象表示合并单元格的情况，管理如下对象

  ```javascript
  {
  	s: { r: startRow, c: 0 },
  	e: { r: endRow, c: 0 }
  }
  ```

  s: start 开始的坐标

  e: end 结束的坐标

- !cols表示每列属性，这里对一些列设置了宽度

## 4、 对获取到的数据进行处理

a.  将数组按照**表名**分组，存储到data对象中

b.  遍历data对象，使用**sheet_add_json()**添加到sheet末尾

```javascript
asyncGetData().then(
  res => {
    const data = {}
    for (let i = 0; i < res.length; i++) {
      const element = res[i]
      if (element['允许空'] === 'YES') element['允许空'] = ''
      if (element['字符长度'] === null) element['字符长度'] = ''
      if (element['数字长度'] === null) element['数字长度'] = ''
      if (element['小数位数'] === null) element['小数位数'] = ''
      if (element['默认值'] === null) element['默认值'] = ''

      if (!data[element.表名]) {
        data[element.表名] = [element]
      } else {
        data[element.表名].push(element)
      }
    }

    // 设置每次合并的开始和结束行数
    let startRow = 1
    let endRow = -1

    for (const key in data) {
      if (Object.hasOwnProperty.call(data, key)) {
        const table = data[key]
        startRow = endRow + 2
        endRow = startRow + table.length - 1

        XLSX.utils.sheet_add_json(workSheet, table, { origin: -1 })

        workSheet['!merges'].push({
          s: { r: startRow, c: 0 },
          e: { r: endRow, c: 0 }
        })
        workSheet['A' + (startRow + 1)].s = {
          alignment: { vertical: 'center' },
          font: { bold: true, color: { rgb: '00C0504D' } }
        }
        for (let i = 0; i < Object.keys(table[0]).length; i++) {
          workSheet[`${String.fromCharCode(65 + i)}${startRow}`].s = {
            font: { bold: true, name: '仿宋', color: { rgb: '004F96D3' } },
            fill: { bgColor: { indexed: 64 }, fgColor: { rgb: "F2F2F2" } }
          }
        }
      }
    }

    XLSX.utils.book_append_sheet(workBook, workSheet, "Entity Definition");

    XLSX_STYLE.writeFile(workBook, 'out.xlsx');

    console.log("----------输出成功！---------");
  }
)
```

