# WPS Office for Linux 二次开发演示

这篇文章介绍如何用 biForm 进行 WPS Office 的二次开发。

## 背景说明

- 开发用到了 [pywpsrpc](https://github.com/timxx/pywpsrpc) 这个项目提供的WPS Python二次开发接口
- 文章中的例子都是在 deepin V20（AMD64）中进行的测试，使用UOS的用户需要进入开发者模式才能使用，否则不能安装pywpsrpc和往biForm安装目录下复制文件
- 测试时用deepin V20 应用商店安装的WPS（版本：11.1.0.9505 - Release 正式版）有问题，直接从官网下载安装WPS for Linux （版本：11.1.0.9615 - Release 正式版）是没问题的
- 目前pywpsrpc只能在**Linux**上用

## 开发前的准备
- 安装 WPS （官网下载）
- 运行 WPS，接受协议之类的操作都完成。确定一下“设置中心”-“切换窗口管理模式”中显示的是“多组件模式”，如果不是就切换到“多组件模式”并重启 WPS。
- 通过应用商店安装 biForm 
- 打开终端，输入以下命令安装 pywpsrpc
` pip3 install pywpsrpc`
- 将 /home/deepin/.local/lib/python3.7/site-packages 下的 pywpsrpc 目录复制到 biform 相关目录下
`cp -rp /home/deepin/.local/lib/python3.7/site-packages/pywpsrpc /opt/apps/com.bilive.biform/files/bin/lib/python3.6/site-packages/`
其中deepin改成你自己的用户名，如果不清楚pywpsrpc装到哪里去了，可以在终端中运行 python3，用以下命令查看
```
import sys
sys.path
```

## biForm 调用演示

- 启动 biForm

- 在空白表单上添加一个按钮

![form](wps/4.png)

- 在按钮的“点击时”脚本中写入：
`		testWps()`

注意testWps()前有一个**tab**符，因为这段脚本是在函数button_clicked()内部的语句，开发者写入的语句都是从第二级开始的，必须要整体缩进一级。

?> 在 biForm 中统一用**tab**缩进，不能用空格。

![button_click](3.png)

- 在脚本编辑器中选择“表单-公共模块”，写入以下Python脚本
```
from pywpsrpc.rpcwpsapi import (createWpsRpcInstance, wpsapi)
from pywpsrpc import RpcIter
def testWps():
	hr, rpc = createWpsRpcInstance()
	hr, app = rpc.getWpsApplication()
	if app is None:
		this.form.showSplashMsg('WPS初始化失败')
	else:
		app.setVisible(False)
		hr, doc = app.Documents.Add()
		selection = app.Selection
		selection.InsertAfter('Hello,world!\n你好，世界！')
		v=doc.SaveAs2("/home/icevi/dev/test/hello1.docx")
		if v!=0:
			this.form.showSplashMsg('保存失败',True)
		else:
			this.form.showSplashMsg('新文档保存在 /home/icevi/dev/test/hello.docx')
		app.Quit(wpsapi.wdDoNotSaveChanges)
```
这段脚本因为是处于公共模块，所有语句都是从第一级开始的，所以不需要整体缩进。

![testWps](wps/2.png)

- 运行
完成以上几步，程序实际上就可以运行了。
点击 biForm 主窗口中的“运行”按钮，或按F5试运行。

![试运行](wps/1.png)

在试运行过程中，可以通过命令交互的方式，输入Python语句进行调试。
![运行调试](wps/5.png)

?> 在 biForm 中的运行只是用于开发时调试，如果程序要提供给最终用户，需要将程序打包成PFF文件，并且最终用户处按前面章节所述配置好环境后才可使用。

- 最后看看生成的文档
用WPS打开生成的 hello.docx ：
![结果](wps/6.png)

## 下载示例
- [本示例所用BIF文件](wps/test_wps.BIF)  

## 更多资料
- [pywpsrpc](https://github.com/timxx/pywpsrpc)  
- WPS官方的开发网站[https://open.wps.cn/docs/office](https://open.wps.cn/docs/office)
- VBA官方文档[https://docs.microsoft.com/en-us/office/vba/api/overview/](https://docs.microsoft.com/en-us/office/vba/api/overview/)
