# 入口

​	**入口起点** 指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。webpack 会找出有哪些模块和库是入口起点（直接和间接）的依赖

```javascript
module.exports = {
    //...
    entry: {
        home: './home.js',
        shared: ['react', 'react-dom', 'redux', 'react-redux'],
        catalog: {
            import './catalog.js',	// 启动时需要加载的模块
            filename: 'pages/catalog.js',	// 指定输出的文件名称
            dependOn: 'shared'	// 默认情况下，每个 chunk 保存了全部弃用的模块，使 dependOn 选项你可以与另一个入口 chunk 共享模块，catalog 这个模块就不会包含 shared 拥有的模块了
        }
    }
}
```

## 分离  app（应用程序）和  vendor（第三方库）入口

`webpack.config.js`

```
module.exports = {
	entry: {
		main: './src/app.js',
		vendor: './src/vendor.js'
	}
}
```

​	这样可以在 `vendor.js` 中存入未做修改的必要 library 或文件（例如Bootstrap, JQuery, 图片等），然后将它们打包在一起成为独立的 chunk。内容哈希保持不变，这使游览器可以独立地缓存它们，从而减少加载时间。

> 在 `webpack4` 中，使用 `optimization.splitChunks`选项，将 vendor 和 app（应用程序）模块分开，并为其创建一个单独的文件。

# 输出

​	**output** 属性告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件。

> 即使可以存在多个 entry 起点，但只能指定一个 output 配置

```javascript
module.exports = {
	//...
    output: {
        /** 
        所有输出文件的目标路径
        必须是绝对路径（使用 Node.js 的 path 模块）	
        */
        path: path.resolve(__dirname, 'dist'),	
        /**
        决定每个输出 bundle 的名称。这些 bundle 将写入到 output.path 选项指定目录下
        当创建多个 bundle 时，应当使用替换方式 [name].[contenthash].bundle.js
        */
        filename: '[name].js',
        publicPath: '/assets/',	// 指定在游览器中所引用的[此输出目录对应的公开URL]，相对于 HTML 页面
        library: {	// 打包 javascript 用于其他人引入使用
            type: 'umd',
            name: 'MyLibrary'
        } 
    }
}
```

`output.publicPath`

> 引用资源时的一个基础路径，类比 `http` 中的 `baseUrl` 
>
> 静态资源最终访问路径 = `output.publicPath` + 资源 `loader`或插件等配置路径
>
> 一般情况下 `publicPath`以 '\' 结尾，而其他`loader`或插件的配置不要以 '/' 开头 

```javascript
output.publicPath = '/dist/'

// image
options: {
    name: 'img/[name].[ext]?[hash]'
}

// 最终图片的访问路径为
output.publicPath + 'img/[name].[ext]?[hash]' = '/dist/img/[name].[ext]?[hash]'
```

# loader

​	webpack 只能理解`javaScript`和`JSON`文件，`loader`让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块以供应用程序使用，以及被添加到依赖图中

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                // 包含
                include: [
                    path.resolve(__dirname, 'app')
                ],
                // 排除
                exclude: [
                    path.resolve(__dirname, 'app/demo-files')
                ]
                use: [
                    { loader: 'style-loader' },
                    {
                        loader: 'css-loader',
                        options: {
                            modules: true
                        }
                    },
                    { loader: 'sass-loader' }
                ],
            }
        ]
    }
}
```

# 插件（plugin）

​	插件用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量

```javascript
const HtmlWebpackPlugin = reqiure('html-webpack-plugin');	// 通过 npm 安装
const webpack = reqiure('webpack');
const path = require('path');

module.exports = {
    entry: './path/to/my/entry/file.js',
    output: {
        filename: 'my-first-webpack.bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                use: 'babel-loader'
            },
        ],
    },
    plugins: [
        new webpack.ProgressPlugin();
        new HtmlWebpackPlugin({ template: './src/index.html' }),
    ]
}
```

# 模块解析

```javascript
module.exports = {
	//...
    resolve: {
         modules: ['node_modules', path.resolve(__dirname, 'app')]	// 告诉 webpack 解析模块时应该搜索的目录，自己添加的目录优先于 node_modules/ 搜索
    	extensions: ['.js', '.json', '.jsx', 'css'],	// 尝试按顺序解析这些后缀名
		alias: {
        	"module": "new-module",
	   		// 别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"
      		"only-module$": "new-module",
      		// 别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"
      		"module": path.resolve(__dirname, "app/third/module.js"),
      		// alias "module" -> "./app/third/module.js" and "module/file" results in error
      		"module": path.resolve(__dirname, "app/third"),
      		// alias "module" -> "./app/third" and "module/file" -> "./app/third/file"
    	}
    }
}
```

>require(...) 默认引用 `package.json` 中的 main
>import .... 默认引用 `package.json` 中的 module

# Module Federation

​	

# manifest

​	runtime 和 manifest 是 webpack 用于管理所有模块交互的代码。理解这个过程对使用浏览器缓存改善项目性能极为重要。

# 模块热替换

​	模块热替换功能会在应用程序运行过程中，替换、添加或删除模块，而无需重新加载整个页面。主要通过以下几种方式来加快开发速度

	- 保留在完全重新加载页面期间丢失的应用程序状态
	- 值只更新变更内容，以节省宝贵的开发时间
	- 在源代码中`CSS/JS`产生修改时，会立即在浏览器中进行更新，这几乎相当于在浏览器`devtools`直接更改样式











