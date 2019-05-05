# 安装

bash on windows 上安装版本10的node:

```bash
wget -qO- https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```

每次安装后之前安装的gitbook会丢失, 需要重新安装一次:

```bash
npm install gitbook-cli -g
```

## 开发环境

Atom

1. 安装扩展:

   1. Script: 运行js

      |      | Mac    | Windows      |
      | ---- | ------ | ------------ |
      | 运行 | cmd+i  | ctrl+shift+b |
      | 结束 | ctrl+c | ctrl+q       |

   2. atom-ternjs: 补全js

      1. Package > Atom Ternjs > Configure project配置;

      2. 配置常用字段:

         | 字段名      | 含义                                              |
         | ----------- | ------------------------------------------------- |
         | ecmaVersion | 选择 ECMAScript 版本                              |
         | libs        | browser 表示原生 js 补全，jquery 代表 jQuery 补全 |
         | loadEagerly | 指定加载解析的 js 文件                            |
         | dontLoad    | 排除加载的文件                                    |
         | plugins     | ternjs 使用的插件，配置的扩展补全的库等           |

      3. Save & Restart server 之后会在项目根目录生成 .tern-project 文件,此时输入已经有了自动补全效果了;

   3. atom-beautify:自动格式化美化代码; 快捷键Ctrl+Alt+B

   4. 

