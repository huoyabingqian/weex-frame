# weex-frame

> 基于weex集成的android、ios、h5框架~

# 工作原理

先简单熟悉一下weex的工作原理，这里引用一下weex官网上的一直图片，[详细信息见官网](https://weex.apache.org/cn/guide/intro/how-it-works.html)

![Weex工作原理](http://upload-images.jianshu.io/upload_images/2843033-a11114f55ceb7478.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

# 开发环境搭建

## weex 开发环境搭建

[关于weex开发环境搭建问题见官方文档](https://weex.apache.org/cn/guide/set-up-env.html)

## android 、iOS 开发环境

[关于native开发环境搭建问题见官方文档](https://weex.apache.org/cn/guide/integrate-to-your-app.html)

# 框架说明

## 基于vue2.0搭建

像前面说的那样weex和vue一直在努力的进行生态互通，而且weex实现web标准化是早晚的问题，所以也建议开发者不要在用.we做后缀来开发了

## 多页模式（抛弃vue-router）

单页形态对于原生可能体验不够好，目前在 native App 里单页模式不太合适

## 集成三端（android、ios、h5平台）

关于android、ios、h5平台的集成与打包问题，在项目中都以解决~

##  集成eslint代码检查

代码检查是必要的操作，为了能够拥有vue开发的体验，将eslint集成进来~

**注：**

由于weexpack暂不支持vue问题，打包相关后续会集成进来~

# 框架介绍

## package.json依赖

```
"dependencies": {
    "vue": "^2.1.8",
    "vue-router": "^2.1.1",
    "vuex": "^2.1.1",
    "vuex-router-sync": "^4.0.1",
    "weex-vue-render": "^0.1.4"
  },
  "devDependencies": {
    "babel-core": "^6.20.0",
    "babel-eslint": "^7.1.1",
    "babel-loader": "^6.2.9",
    "babel-preset-es2015": "^6.18.0",
    "css-loader": "^0.26.1",
    "eslint": "^3.15.0",
    "eslint-config-standard": "^6.2.1",
    "eslint-loader": "^1.6.1",
    "eslint-plugin-html": "^2.0.1",
    "eslint-plugin-promise": "^3.4.2",
    "eslint-plugin-standard": "^2.0.1",
    "postcss-cssnext": "^2.9.0",
    "serve": "^1.4.0",
    "vue-loader": "^10.0.2",
    "vue-template-compiler": "^2.1.8",
    "webpack": "^1.14.0",
    "weex-devtool": "^0.2.64",
    "weex-loader": "^0.4.1",
    "weex-vue-loader": "^0.2.5"
  }
```

## 打包配置

1、 遍历.vue文件后缀，生成相应的entry.js文件

```
function getEntryFileContent (entryPath, vueFilePath) {
  const relativePath = path.relative(path.join(entryPath, '../'), vueFilePath);
  return 'var App = require(\'' + relativePath + '\')\n'
    + 'App.el = \'#root\'\n'
    + 'new Vue(App)\n'
}

function walk (dir) {
  dir = dir || '.'
  let directory = path.join(__dirname, './src', dir)
  let entryDirectory = path.join(__dirname, './src/entry');
  fs.readdirSync(directory)
    .forEach(file => {
      let fullpath = path.join(directory, file)
      let stat = fs.statSync(fullpath)
      let extname = path.extname(fullpath)
      if (stat.isFile() && extname === '.vue') {
        let entryFile = path.join(entryDirectory, dir, path.basename(file, extname) + '.js')
        fs.outputFileSync(entryFile, getEntryFileContent(entryFile, fullpath))
        let name = path.join(dir, path.basename(file, extname))
        entry[name] = entryFile + '?entry=true'
      } else if (stat.isDirectory()) {
        let subdir = path.join(dir, file)
        walk(subdir)
      }
    })
}

walk()
```

2、通过weex-loader打包生成native jsbundle
3、 通过weex-vue-loader打包生成web jsbundle

```
function getBaseConfig () {
  return {
    entry: entry,
    output: {
      path: 'dist'
    },
    resolve: {
      extensions: ['', '.js', '.vue'],
      fallback: [path.join(__dirname, './node_modules')],
      alias: {
        'assets': path.resolve(__dirname, './src/assets/'),
        'components': path.resolve(__dirname, './src/components/'),
        'constants': path.resolve(__dirname, './src/constants/'),
        'api': path.resolve(__dirname, './src/api/'),
        'router': path.resolve(__dirname, './src/router/'),
        'store': path.resolve(__dirname, './src/store/'),
        'views': path.resolve(__dirname, './src/views/'),
        'config': path.resolve(__dirname, './config'),
        'utils': path.resolve(__dirname, './src/utils/')
      }
    },
    module: {
      preLoaders: [
        {
          test: /\.vue$/,
          loader: 'eslint',
          exclude: /node_modules/
        },
        {
          test: /\.js$/,
          loader: 'eslint',
          exclude: /node_modules/
        }
      ],
      loaders: [
        {
          test: /\.js$/,
          loader: 'babel',
          exclude: /node_modules/
        }, {
          test: /\.vue(\?[^?]+)?$/,
          loaders: []
        }
      ]
    },
    vue: {
      postcss: [cssnext({
        features: {
          autoprefixer: false
        }
      })]
    },
    plugins: [bannerPlugin]
  }
}

const webConfig = getBaseConfig()
webConfig.output.filename = 'web/[name].js'
webConfig.module.loaders[1].loaders.push('vue')

const weexConfig = getBaseConfig()
weexConfig.output.filename = 'weex/[name].js'
weexConfig.module.loaders[1].loaders.push('weex')
```

# 项目结构

```
weex-frame
├── android (android项目)
│       
├── ios （ios项目代码）
│
├── src （weex模块）
│      ├── api (api模块)
│      ├── components（组件模块） 
│      ├── constants（常量配置）   
│      ├── utils （工具模块）   
│      └── views（视图模块）  
│
└── dist （build输出模块）
       ├── weex (native使用jsbundle)
       └── web（web使用jsbundle） 
```

# npm 指令

1. npm run build
2. npm run dev
3. nom run serve

# 项目启动

1. git clone git@github.com:osmartian/weex-frame.git
2. cd weex-frame
3. npm install
4. 执行 ./start

## android 启动

1. 打开andorid studio
2. File -> New -> Import Project -> weex-frame/android -> 启动

## iOS 启动

1. cd ios
2. pod install (未安装pod，请先安装)
3. open WeexFrame.xcworkspace

## h5 启动方式

 打开 [http://localhost:12580/weex.html](http://localhost:12580/weex.html)

# 项目示例

## h5 端示例

![h5我的页面](http://upload-images.jianshu.io/upload_images/2843033-2c404d16e05b8f0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

![h5发起页面](http://upload-images.jianshu.io/upload_images/2843033-8eda0114ba0ca246.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

## android 端示例

![android首页](http://upload-images.jianshu.io/upload_images/2843033-26182ae64ca5171a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

![android我的页面](http://upload-images.jianshu.io/upload_images/2843033-d1c5d7de21ce940e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

![android发起页面](http://upload-images.jianshu.io/upload_images/2843033-d92f0ba3f5af4372.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

## iOS 端示例

![iOS首页](http://upload-images.jianshu.io/upload_images/2843033-120c80bc608d0471.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

![iOS我的页面](http://upload-images.jianshu.io/upload_images/2843033-95742f3b0dc964fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)

![iOS发起页面](http://upload-images.jianshu.io/upload_images/2843033-55bbaf814b08c429.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/320)
