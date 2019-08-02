---
title: create-react-app 怎么这么 Diao!
date: 2019-08-02 12:00:00
tags: React 前端工程 create-react-app
categories: React
thumbnail: /img/Facebook.png
---

## 前言

前段时间自己做了一个前端脚手架工具 Zeus，通过 cli 命令去搭建前端框架，目前支持了 web, node 工具类，chrome 扩展插件等模板。
其实原理都是一样的，根据不同的配置去 down 不同的 template，以及执行相应的 script。在做的过程中发现模板的搭建还真不是一个容易的活，踩过了各种坑，但同时对 webpack, rollup 这些工具也有了深入的了解。所以就在想 [create-react-app](https://facebook.github.io/create-react-app/) 是如何实现类似功能的。

这个系列打算从 create-react-app 着手去分析它的源码。希望自己能够坚持下去~

## 目录结构

#### 一级目录结构

<p class="center">![一级目录结构](tree_root.1.png)</p>

create-react-app 使用 [Lerna](https://lerna.js.org/) 管理前端 packages

值得关注的代码在 packages 这个目录下面

我们来看看 packages 里面都有什么

#### packages 目录结构

<p class="center">![packages 目录结构](tree_root_packages.1.png)</p>

这里有我们熟悉的 create-react-app 目录，其余的包也挺重要，比如 react-scripts 但是我们先找到入口再来看其他文件夹的作用吧。

我们要找的入口就在 create-react-app 里面

#### create-react-app 目录结构

<p class="center">![create-react-app 目录结构](tree_root_packages_create_react_app.1.png)</p>

## create-react-app 源码解读

### create-react-app 用法

在看源码之前，我们要先知道如何使用 creat-react-app 这样能够帮助我们快速理解各部分代码的作用。

<p class="center">![how to use](how-to-use.png)</p>

可以看到用法就是 `create-react-app <project-directory> [options]`

非常的简单易用，而且各参数的作用简单易懂，一看就知道怎么使用，命名非常的好。

### create-rect-app/index.js

这是入口文件非常简明，就是判断当前 node 版本，如果主版本低于 8 就输出一段错误提示信息，并终止进程。并返回 1 作为退出码。*(Linux 系统中只有 0 是成功码，其余的退出码都表示进程没有按预期执行成功)*
如果主版本适配，那么就执行 creatReactApp 里的代码

```javascript
var currentNodeVersion = process.versions.node;
var semver = currentNodeVersion.split('.');
var major = semver[0];

if (major < 8) {
  console.error(
    'You are running Node ' +
      currentNodeVersion +
      '.\n' +
      'Create React App requires Node 8 or higher. \n' +
      'Please update your version of Node.'
  );
  process.exit(1);
}

require('./createReactApp');
```

### create-rect-app/createReactApp.js

这里的代码就有点多了，不能把全部都给粘上来。我们一步步分析。

```javascript
const chalk = require('chalk');
const commander = require('commander');
const dns = require('dns');
const envinfo = require('envinfo');
const execSync = require('child_process').execSync;
const fs = require('fs-extra');
const hyperquest = require('hyperquest');
const inquirer = require('inquirer');
const os = require('os');
const path = require('path');
const semver = require('semver');
const spawn = require('cross-spawn');
const tmp = require('tmp');
const unpack = require('tar-pack').unpack;
const url = require('url');
const validateProjectName = require('validate-npm-package-name');

const packageJson = require('./package.json');
```

这里我们忽略头上的包的引用，直接看代码

使用 [commander](https://github.com/tj/commander.js) 做命令行工具

```javascript
// 这段代码只是对该工具可以用哪些方法做一个声明和初始化

let projectName;

const program = new commander.Command(packageJson.name)
  // 版本号
  .version(packageJson.version)
  .arguments('<project-directory>')
  // 申明使用方法
  .usage(`${chalk.green('<project-directory>')} [options]`)
  .action(name => {
    // 这里的 name 就是使用 create-react-app 传入的 <project-directory>
    projectName = name;
  })
  // 申明 option 信息
  .option('--verbose', 'print additional logs')
  .option('--info', 'print environment debug info')
  .option(
    '--scripts-version <alternative-package>',
    'use a non-standard version of react-scripts'
  )
  .option('--use-npm')
  .option('--use-pnp')
  .option('--typescript')
  .allowUnknownOption()
  // 帮助信息
  .on('--help', () => {/* 省略...  */})
  .parse(process.argv);

// 如果使用了 --info 参数就输出系统信息且不创建项目并返回
if (program.info) {
  console.log(chalk.bold('\nEnvironment Info:'));
  return envinfo
    .run(
      {
        System: ['OS', 'CPU'],
        Binaries: ['Node', 'npm', 'Yarn'],
        Browsers: ['Chrome', 'Edge', 'Internet Explorer', 'Firefox', 'Safari'],
        npmPackages: ['react', 'react-dom', 'react-scripts'],
        npmGlobalPackages: ['create-react-app'],
      },
      {
        duplicates: true,
        showNotFound: true,
      }
    )
    .then(console.log);
}

// 如果没哟输入项目名称就提示如何使用并退出(退出码为 1)
if (typeof projectName === 'undefined') {
  /* 省略... */
  process.exit(1);
}

// 这申明了一个内部程序，当 --scripts-version 选项使用时才出发相应的操作
// 可以下载自定义的模板文件
const hiddenProgram = new commander.Command()
  .option(
    '--internal-testing-template <path-to-template>',
    '(internal usage only, DO NOT RELY ON THIS) ' +
      'use a non-standard application template'
  )
  .parse(process.argv);


// 这里才是 creat-react-app 处理完输入参数后执行的函数，可以看到它吧相关参数信息都传到这个函数里的
createApp(
  projectName,
  program.verbose,
  program.scriptsVersion,
  program.useNpm,
  program.usePnp,
  program.typescript,
  hiddenProgram.internalTestingTemplate
);
```

那么下面我们看看 creatApp 这个函数里面做了什么吧


```javascript
function createApp(
  // 项目名称
  name,
  // 是否打印附加 log 信息
  verbose,
  // 版本
  version,
  // 是否使用 npm
  useNpm,
  // 是否使用 Yarn Plug'n'Play
  usePnp,
  // 是否使用 ts
  useTypescript,
  // 模板
  template
) {
  const root = path.resolve(name);
  const appName = path.basename(root);
  // 检测名称是否合法
  checkAppName(appName);
  // 具体实现 -> node-fs-extra/lib/mkdirs/mkdirs-sync.js
  // 总之就是如果该目录不存在就创建一个
  fs.ensureDirSync(name);
  // 判断该目录下是否已有一些冲突文件，如果有就认为覆盖现有文件有风险，就退出
  if (!isSafeToCreateProjectIn(root, name)) {
    process.exit(1);
  }

  console.log(`Creating a new React app in ${chalk.green(root)}.`);
  console.log();

  const packageJson = {
    name: appName,
    version: '0.1.0',
    private: true,
  };
  // 写入 package.json
  fs.writeFileSync(
    path.join(root, 'package.json'),
    JSON.stringify(packageJson, null, 2) + os.EOL
  );
  // useYarn 和 useNpm 不传的话
  const useYarn = useNpm ? false : shouldUseYarn();
  const originalDirectory = process.cwd();
  process.chdir(root);
  if (!useYarn && !checkThatNpmCanReadCwd()) {
    process.exit(1);
  }

  if (!semver.satisfies(process.version, '>=8.10.0')) {
    console.log(
      chalk.yellow(
        `You are using Node ${
          process.version
        } so the project will be bootstrapped with an old unsupported version of tools.\n\n` +
          `Please update to Node 8.10 or higher for a better, fully supported experience.\n`
      )
    );
    // Fall back to latest supported react-scripts on Node 4
    version = 'react-scripts@0.9.x';
  }

  if (!useYarn) {
    const npmInfo = checkNpmVersion();
    if (!npmInfo.hasMinNpm) {
      if (npmInfo.npmVersion) {
        console.log(
          chalk.yellow(
            `You are using npm ${
              npmInfo.npmVersion
            } so the project will be bootstrapped with an old unsupported version of tools.\n\n` +
              `Please update to npm 5 or higher for a better, fully supported experience.\n`
          )
        );
      }
      // Fall back to latest supported react-scripts for npm 3
      version = 'react-scripts@0.9.x';
    }
  } else if (usePnp) {
    const yarnInfo = checkYarnVersion();
    if (!yarnInfo.hasMinYarnPnp) {
      if (yarnInfo.yarnVersion) {
        console.log(
          chalk.yellow(
            `You are using Yarn ${
              yarnInfo.yarnVersion
            } together with the --use-pnp flag, but Plug'n'Play is only supported starting from the 1.12 release.\n\n` +
              `Please update to Yarn 1.12 or higher for a better, fully supported experience.\n`
          )
        );
      }
      // 1.11 had an issue with webpack-dev-middleware, so better not use PnP with it (never reached stable, but still)
      usePnp = false;
    }
  }

  if (useYarn) {
    let yarnUsesDefaultRegistry = true;
    try {
      yarnUsesDefaultRegistry =
        execSync('yarnpkg config get registry')
          .toString()
          .trim() === 'https://registry.yarnpkg.com';
    } catch (e) {
      // ignore
    }
    if (yarnUsesDefaultRegistry) {
      fs.copySync(
        require.resolve('./yarn.lock.cached'),
        path.join(root, 'yarn.lock')
      );
    }
  }

  run(
    root,
    appName,
    version,
    verbose,
    originalDirectory,
    template,
    useYarn,
    usePnp,
    useTypescript
  );
}
```







这里我不知道的知识
**拓展 [Yarn Plug'n'Play](https://yarn.bootcss.com/docs/pnp/)**
