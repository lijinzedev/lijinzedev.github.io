---
title: Electron静默打印
top: false
cover: false
toc: true
mathjax: true
categories:
  - 前端
  - electron
tags:
  - 前端
  - electron
date: 2021-11-12 10:16:30
password:
summary:
---

# 前言

> 本文为在学习electron静默打印后的笔记，非原创。原创连接https://www.cnblogs.com/coderDemo/p/14450853.htm



# 一、Electron 静默打印

[源码](https://gitee.com/boolean-true/study/tree/master/electron/print)

# 打印HTML格式

### main进程中

```javascript
const path = require('path')
const {BrowserWindow, app, ipcMain} = require('electron')

const isPrdEnv = process.env.NODE_ENV === 'production'
const staticPath = isPrdEnv ? './static' : '../../../../static' // 根据当前代码的js相对static文件夹路径

let printWindow = null
const url = `file://${path.resolve(__dirname, `${staticPath}/print.html`)}`

app.whenReady().then(() => {
    printWindow = new BrowserWindow({
        show: false,
        webPreferences: {
            nodeIntegration: true,
            contextIsolation: false
        }
    })
    printWindow.loadURL(url)
})

/**
 * 静默打印html
 * @Param content Html字符串
 * @Param deviceName 打印机名称
 * @return promise
 * */
const htmlToPrint = (content, deviceName, margin) => {
    return new Promise((resolve, reject) => {
        if (!printWindow) return reject('请等待控件加载完成后重试')
        const htmlPrintingListener = () => {
            printWindow.webContents.print({
                silent: true,
                printBackground: false,
                deviceName
            }, (success, failureReason) => {
                ipcMain.removeListener('htmlPrinting', htmlPrintingListener)
                if (success) resolve(true)
                else reject('打印失败')
            })
        }

        printWindow.webContents.send('htmlPrint', { content, margin, deviceName })
        ipcMain.on('htmlPrinting', htmlPrintingListener)
    })
}


export default htmlToPrint
```

### static文件夹下新建print.html键入以下代码

```html
复制代码123456789101112131415161718
HTML<!DOCTYPE html>
<html lang="en">
<head>
    <title>Document</title>
</head>

<body>
<div id='container'></div>
</body>
<script>
    //引入ipcRenderer对象
    const { ipcRenderer } = require('electron')
    ipcRenderer.on('htmlPrint', (e, content) => { //接收响应
        document.querySelector('#container').innerHTML = content
        ipcRenderer.send('htmlPrinting') //向webview所在页面的进程传达消息
    })
</script>
</html>
```

------

# 打印网络PDF

**打印本地PDF同理更简单, 需要借助一个第三方软件SumatraPDF 去官网下压缩包解压后放到static文件夹下就行**

### main进程中

```javascript
const path = require('path')
const https = require('https')
const fs = require('fs')
const cp = require('child_process')

const isPrdEnv = process.env.NODE_ENV === 'production'
const staticPath = isPrdEnv ? './static' : '../../../../static' // 根据当前代码的js相对static文件夹路径

/*
* 处理await 的异常捕获
* */
const awaitWrapper = promise => promise.then(result => [null, result]).catch(error => [error, null])

/*
* 生成随机字符串
* */
const randomString = () => Math.random().toString(36).slice(-6)

/*
* 获取网络pdf的buffer
* */
const getFileBuffer = url => {
    return new Promise(((resolve, reject) => {
        https.get(url, response => {
            const chunks = []
            let size = 0
            response.on('data', chunk => {
                chunks.push(chunk)
                size += chunk.length
            })
            response.on('end', () => {
                const buffer = Buffer.concat(chunks, size)
                resolve(buffer)
            })
        })
    }))
}

/*
* 将buffer保存为本地临时文件
* */
const savePdf = buffer => {
    return new Promise((resolve, reject) => {
        const pdfUrl = path.resolve(__dirname, `${staticPath}/${randomString()}.pdf`)
        fs.writeFile(pdfUrl, buffer, {encoding: 'utf8'}, err => {
            if (err) {
                reject('缓存pdf打印文件失败')
            } else {
                resolve(pdfUrl)
            }
        })
    })
}

/*
* 调用SumatraPDF 执行pdf打印
* */
const executePrint = (pdfPath, deviceName) => {
    return new Promise((resolve, reject) => {
        cp.exec(`SumatraPDF.exe -print-to "${deviceName}"  "${pdfPath}"`,
            {
                windowsHide: true,
                cwd: path.resolve(__dirname, staticPath)
            },
            e => {
                if (e) {
                    reject(`${url}在${deviceName}上打印失败`)
                } else {
                    resolve(true)
                }
                /* 打印完成后删除创建的临时文件 */
                fs.unlink(pdfPath, Function.prototype)
            })
    })
}

/*
* 静默打印pdf
* */
const pdfToPrint = (url, deviceName) => {
    return new Promise(async (resolve, reject) => {
        /* 根据url获取buffer并返回，如果获取失败就直接reject */
        const [bufferError, buffer] = await awaitWrapper(getFileBuffer(url))
        if (bufferError) return reject('获取网络pdf文件信息失败')
        /* 根据buffer将文件缓存到本地并返回临时pdf文件路径，如果存储失败就直接reject */
        const [pdfPathError, pdfPath] = await awaitWrapper(savePdf(buffer))
        if (pdfPathError) return reject(pdfPathError)
        /* 根据临时pdf文件路径 和打印机名称来执行打印*/
        const [execPrintError, printResult] = await awaitWrapper(executePrint(pdfPath, deviceName))
        if (execPrintError) {
            reject(execPrintError)
        } else {
            resolve(printResult)
        }
    })
}
```

------

# 关键事项

### package.json中

1.打包static目录的文件没有打包进去，需要在package.json 里面添加extraResources 额外资源
2.打包后远程下载pdf无法放入/static下，原因是electron-vue 默认是用asar打包，而asar只能读取不能写入，所以需要远程打印pdf就不能打包成asar

```json
"win": {
         "icon": "dist/electron/static/icon2.ico",
         "extraResources": [
            "./static/*.html",
            "./static/*.txt",
            "./static/*.exe",
            "./static/*.pdf"
         ],
         "asar": false
    }
```

# 多平台打印兼容

> 多平台打印为使用linux命令进行打印，做好环境判断

```javascript
/*
* 调用SumatraPDF 执行pdf打印
* */
const executePrint = (pdfPath, deviceName) => {
  debugger
  return new Promise((resolve, reject) => {
    console.log(path.resolve(staticPath))
    debugger
    switch (process.platform) {
      case 'darwin':
      case 'linux':
        cp.exec(
          'lp ' + pdfPath, (e) => {
            if (e) {
              throw e;
            }
            /* 打印完成后删除创建的临时文件 */
            fs.unlink(pdfPath, Function.prototype)
          });
        break;
      case 'win32':
            // 调用默认打印机打印
        cp.exec(`SumatraPDF.exe -print-to-default  "${pdfPath}"`,
          {
            windowsHide: true,
            cwd: exePath
          },
          e => {
            if (e) {
              debugger
              reject(`${pdfPath}在${deviceName}上打印失败`)
            } else {
              debugger
              resolve(true)
            }
            /* 打印完成后删除创建的临时文件 */
            fs.unlink(pdfPath, Function.prototype)
          })
        break;
      default:
        throw new Error(
          'Platform not supported.'
        );
    }
  })
}
```

# 参阅

[主要参阅](https://www.cnblogs.com/coderDemo/p/14450853.html#static%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%8B%E6%96%B0%E5%BB%BAprinthtml%E9%94%AE%E5%85%A5%E4%BB%A5%E4%B8%8B%E4%BB%A3%E7%A0%81)

[文档]: https://www.sumatrapdfreader.org/docs/Command-line-argumentsv

