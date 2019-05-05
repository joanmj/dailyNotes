Chrome Extensions

- 1. 包含以下部分:

  2. background scripts
  3. content scripts,
  4. an options page
  5. UI elements
  6. and various logic files
- 1. Added as an extension in developer mode

  2. navigating to `chrome://extensions`.
  3. Enable Developer Mode by clicking the toggle switch next to **Developer mode**.
  4. Click the **LOAD UNPACKED** button and select the extension directory.

# 提示关闭

提示“请停用以开发者模式运行的扩展程序”的问题, 可以通过打包导出报警提示的chrome插件重新安装的方法解决;

如果以上的方法还不行的话，那么就尝试找出是哪个插件导致提示“请停用以开发者模式运行的扩展程序”，然后将其打包导出扩展程序，方法参照：[怎么从Chrome浏览器中导出扩展程序为crx文件？](http://www.cnplugins.com/tool/export-cnplugins-crx.html)“打包扩展程序”这样在相应目录生成了 .crx 文件，然后直接拖到 chrome://extensions/ chrome扩展程序页面进行安装就行！如果不知道怎么安装CRX文件，请参照：[怎么在谷歌浏览器中安装.crx扩展名的离线chrome插件](http://www.cnplugins.com/tool/outline-install-crx-file.html)

