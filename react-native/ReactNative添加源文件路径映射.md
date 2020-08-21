## 在React Native中增加路径映射功能

在开发项目过程中, 经常遇到文件路径变更的问题, 导致移动一个文件的位置之后, 很多源文件都需要去修改import路径, 非常麻烦. 于是找到了[这篇文章](https://www.tslang.cn/docs/handbook/module-resolution.html) , 可以简化使用相对路径import文件的问题.

* 编辑tsconfig.json中的`paths`模块, 增加新的模块别名:

  ```json
{

  //.......

  "compilerOptions": {
   "baseUrl": "./", /* Base directory to resolve non-absolute module names. */
    "paths": {
      "apis/*": ["src/apis/*"],
      "assets/*": ["src/assets/*"],
     }, /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
  },

  // ......
  }
  ```



* 编辑babel.config.js:

首先安装模块: `yarn add -D babel-plugin-module-resolver`

  ```json
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
  plugins: [
    [
     'module-resolver',
     {
	      root: ['./src'],
        extensions: ['.tsx', '.ts'],
        alias: {
         apis: './src/apis/',
         assets: './src/assets/',
        },
     },
    ],
  ],
};
  ```



*  编辑metro.config.js, 添加 `resetCache: true`

```json
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
       experimentalImportSupport: false,
        inlineRequires: false,
        resetCache: true,
     },
    }),
  },
};
```



* 使用映射之后的路径import文件, 无需再关注它们的相对路径了

```js
import {APIS} from 'apis/APIS'
import {Assets} from 'assets/Assets'

// .....
```

