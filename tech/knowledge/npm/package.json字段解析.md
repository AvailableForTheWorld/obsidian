1. 必要的核心元数据
	1. name 项目名
	2. version 版本号 

2. entry points 入口点
	这是文件中最重要的部分，它规定了引入npm包的时候加载哪一个文件。关键字段如下：
	1. main
		老版本的node.js中作文用户引入包的入口
		```
		{
			"main": "./lib/index.js"
		}
		```
		但是它只能指向单一CommonJS入口文件，而且无法兼容EsModule/条件导出等现代模式
	2. exports
		现代导出的首选方式，用于替代main
		```json
		{
		  "name": "my-awesome-package",
		  "exports": {
		    ".": {
		      "node": {
		        "require": "./lib/index.cjs",
		        "import": "./lib/index.mjs"  
		      },
		      "types": "./lib/index.d.ts",
		      "default": "./lib/index.esm.js"
		    },
		
		    "./utils": {
		      "node": "./utils/node-specific.cjs",
		      "default": "./utils/utils.esm.js"
		    },
		    "./package.json": "./package.json"
		  }
		
		}
		```
		其中有三种模式 1. 主入口.导出 2. 子路径导出 3. 子路径单一文件导出
		**重要规则**：**如果定义了 `"exports"`，则 `"main"`、`"module"` 等字段会被现代工具忽略。**
	3. module
		约定俗成的字段，非node.js官方标准，是打包工具提供es module的入口点，不用esports时，会与main字段配合使用（main 指向commonJS模块，module指向ES module），以便打包工具执行tree-shaking
		```json
		{
		  "main": "./lib/index.cjs",
		  "module": "./lib/index.esm.js"  
		}
		```
	4. browser
		在**浏览器环境**下的入口点，替换node.js中的main/exports
		```json
		{
		  "main": "./index.node.js",
		  "browser": "./index.browser.js"
		}
		```

3. scripts 脚本命令
	用于npm 执行脚本自动化的命令集，一般包括 dev/build/test/lint等命令

4. type 项目类型 
	1. module： es module模式
	2. commonjs： commonJs模式

5. types 导出ts文件出口
	老版本node.js打包出口，现在一般不用，在现代打包工具一般打包出口从exports中的types中取，如下：
	```json
	"exports": {
	    ".": {
	      "import": "./dist/index.js",
	      "require": "./dist/index.cjs",
	      "types": "./dist/index.d.ts"
	    },
	    "./styles.css": "./dist/styles.css"
	  },
	```

6. files 文件名
	为数组，指定哪写文件会放置在打包好的依赖中
	```json
	{
	  "files": [
	    "dist",
	    "lib",
	    "src",
	    "README.md"
	  ]
	}
	```



