---
title: js 导出、导入excel
date: 2023-08-14 18:02:06
tags:
---

大坑，来自朋友的一个需求，花费了我大几个小时研究，还没有得出什么结论，但感觉这个也是成长过程中不可或缺的一步呀
![iShot_2023-08-14_10.44.56.gif](https://hutianhua.com:6395/images/2023/08/14/iShot_2023-08-14_10.44.56.gif)

1.目的，从一个汇总表，拆分成N张记录的表
2.解决读取本地文件不成功的问题（后来使用上传文件的插件，获取文件流

```javascript
    async handleFile(info) {
        console.log('info:', info)
        // this.file2Xce(e)
        const {fileList} = info
        const resultInfo = await  this.file2Xce(fileList[0].originFileObj)
        console.log("resultInfo:",resultInfo)
    
        const temObject = resultInfo[0].data[3]
        console.log("===", temObject)
        this.writeLedgerAsExcel(temObject)
    }
    // 上传文件
    file2Xce(file) {
        return new Promise(function(resolve) {
            const reader = new FileReader()
            reader.onload = function(e) {
                const data = e.target.result
                const workbook = XLSX.read(data, {
                    type: 'binary'
                })
                let result = []
                workbook.SheetNames.forEach(item => {
                    result.push({
                        sheetName: item,
                        data: XLSX.utils.sheet_to_json(workbook.Sheets[item])
                    })
                })
                resolve(result)
            }
            reader.readAsBinaryString(file)
        })
    }
    writeLedgerAsExcel(temObject) {
        const {__EMPTY ,__EMPTY_1,__EMPTY_3,__EMPTY_4,__EMPTY_5,} = temObject
        const jsonData = [
            {姓名:__EMPTY,性别:__EMPTY_1,出生日期:__EMPTY_5,身份证号码:__EMPTY_3,联系号码:__EMPTY_4}
        ]
        let jsonWorkSheet = XLSX.utils.json_to_sheet(jsonData)
        let workBook = {
            SheetNames: ['jsonWorkSheet'],
            Sheets: {
                jsonWorkSheet: jsonWorkSheet
            }
        }
    
        XLSX.writeFile(workBook, './bb.xlsx')
    }
    readExcel() {
      let jsonWorkSheet = XLSX.utils.json_to_sheet(this.jsonData)
      let workBook = {
        SheetNames: ['jsonWorkSheet'],
        Sheets: {
          jsonWorkSheet: jsonWorkSheet
        }
      }

      XLSX.writeFile(workBook, './bb.xlsx')
    }
```