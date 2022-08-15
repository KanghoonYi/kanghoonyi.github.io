---
title: Lambda Application optimization on Monorepo
author:
name: KanghoonYi(pour)
date: 2022-08-14 20:25:00 +0900
categories: [Typescript, Javascript]
tags: [Typescript, Mono-repo, npm, MSA]
pin: false
---

### Monorepo(Monolithic repository)란?
하나의 repository(git repository)에서 여러개의 project를 관리하는, repository를 사용하는 전략중에 하나입니다.  
MSA(MicroService Architecture)에서 각 도메인 단위로 서비스를 분리하면서, 이 코드들을 관리하기위한 전략으로서 많이 쓰입니다.

### Bundling이란?
Application을 구동하는데 필요한 코드들만 모아, packaging하는것을 말합니다.  
code를 작성하면서, 실제로 사용할 코드와 그렇지 않은코드를 분리하여 관리하는것은 어렵습니다. 때문에, Code 작성시점이 아닌, 별도의 번들링과정을 통해, 작동에 필요한 코드들만 filtering하여 서비스합니다.


### Monorepo와 Bundling이 무슨 상관인데?
Monorepo를 운영하다보면, 각 service에서 공통으로 사용되는 코드들이 생깁니다(ex: DB커넥션 생성코드). 때문에, 이런 코드들을 관리하기 위한 공통 모듈공간을 만들게 되는데,
이때 공통모듈(이하 common모듈)을 참조하는 방식을 Bundling과정에서 최적화(Optimization)할 수 있습니다.  

### 이게 왜 필요한데?
lambda를 이용하여 서비스를 운영한다면, lambda의 cold start 문제에 부딪히게 됩니다. 이때 app initialization시간을 최소화하기 위해 이런 최적화가 필요합니다.
lambda에 initialize 시간에 영향을 미치는 요소는 다음과 같습니다.([Lambda performance optimization](https://aws.amazon.com/ko/blogs/compute/operating-lambda-performance-optimization-part-2/))
```text
The size of the function package, in terms of imported libraries and dependencies, and Lambda layers.
The amount of code and initialization work.
The performance of libraries and other services in setting up connections and other resources.
```

HTTP 서버 운영을 `lambda + API Gateway`를 통해 한다면, lambda는 각 end-point별로 생성될 것입니다. 이때 각 lambda 구동에 필요한 코드들만 bundling하여 성능최적화를 달성합니다.

### 공통(Common)모듈 운영하기
1. common모듈을 운영하는 가장 쉬운 방법은, 각 서비스에서 외부모듈로서 참조하는 방식입니다. 예를들어, `npm`을 사용하는 경우, local path로서 `npm install`을 사용하는 것입니다.  
   이 방법은 운영할때, 외부 모듈로서 다루면 되기 때문에 적용하기 쉽지만, '외부 모듈'이기 때문에, bundling시에 optimization 대상이 되지 않는 단점이 있습니다.
```text
$ npm install ../common
```

2. code관리는 별도의 path를 분리해서 관리하지만, 참조할때는 각 서비스 내부에서 생성한 파일처럼 사용합니다.
이는 Code를 관리하는 측면과, 사용하는 측면을 분리하여 구성하는 방법입니다.  
공통된 code를 한곳에서 관리하기 위한 `관리`의 목적과, 코드를 참조할때는 service수준의 code로서 취급하여 optimization 대상이 되도록 하는 `최적화`를 각각 별도로 달성하는 방법입니다.  


이 포스트에서는 2번 방법을 다룹니다.

### 본격적으로 최적화(Optimization) 하기
#### typescript의 path alias
typescript는 특정 path를 alias로 지정할 수 있습니다.([tsconfig paths](https://www.typescriptlang.org/tsconfig#paths))
예를 들어, 다음과 같이 tsconfig를 설정하여, path alias를 생성할 수 있습니다
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@common": "../common"
    }
  }
}
```

#### webpack bundling
common 모듈을 webpack에서 번들링 및 최적화할 수 있도록 설정합니다.  
nodejs를 사용하는 경우, 외부 모듈을 따로 번들링하지 않습니다. 이는, `webpack-node-externals`이라는 모듈을 이용해 세팅합니다.  
그러나, common모듈은 typescript의 path alias 기능을 사용한 것이기 때문에, 별도의 bundling과정을 거치도록, `webpack-node-externals`설정에서 allowlist에 포함시켜 줍니다.
이렇게 설정하면, `@common/~`경로로 import한 모든 구문이 bundling대상이 되며, 구동에 필요한 모든 모듈을 가져와 packaging합니다.
```typescript
// webpack.config.ts
import nodeExternals from 'webpack-node-externals';
import TsconfigPathsPlugin from 'tsconfig-paths-webpack-plugin';

export default {
	...,
  module: {
    rules: [
      {
        test: /\.(tsx?)$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              transpileOnly: true,
              experimentalWatchApi: true,
            },
          },
        ],
        exclude: [
          path.resolve(contextDir, 'node_modules'),
          path.resolve(contextDir, '.serverless'),
          path.resolve(contextDir, '.webpack'),
          path.resolve(contextDir, 'coverage'),
        ],
      },
    ],
  },
  resolve: {
    extensions: ['.js', '.ts', '.mjs', '.json'],
    symlinks: false,
    cacheWithContext: false,
    plugins: [
      new TsconfigPathsPlugin({
        configFile: './tsconfig.json',
        extensions: ['.ts'],
      }),
    ],
    enforceExtension: false,
  },
  optimization: {
    concatenateModules: false,
    minimize: true,
    minimizer: [new TerserPlugin()],
  },
  externalsPresets: {
    node: true,
  },
  externals: [
    nodeExternals({
      modulesDir: nodeModulePath,
      allowlist: [/^@common/, /^lodash/],
    }),
  ],
  ...,
}
```

### 결과
이제 다음과 같이 import한 내용들은 모두 bundling및 최적화 과정을 거칩니다.
```typescript
import { test, addFunction } from '@common/util'; 
```

이를 통해, lambda에 올릴 package를 구동에 필요한것만 갖추어 용량을 낮출 수 있었으며, 개발과정에서 따로 신경쓰지 않아도, 관련없는 initialize과정을 자동으로 filtering할 수 있게 되었습니다.


### Reference
- [Monorepo Wiki](https://en.wikipedia.org/wiki/Monorepo)  
- [tsconfig paths](https://www.typescriptlang.org/tsconfig#paths)  
- [Lambda performance optimization](https://aws.amazon.com/ko/blogs/compute/operating-lambda-performance-optimization-part-2/)
