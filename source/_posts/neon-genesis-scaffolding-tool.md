---
title: 新世纪福音脚手架
subtitle: she said, "it's futuristic."
date: 2017-05-21 20:20:34
tags:
---
好的作品不在于其形式，重要的是内容，我的很多音乐作品都是没有名字的，然而没有名字的东西无法商业流通，所以你才会看到我那些没有特殊意义、单纯靠滚键盘得到的名字。 —— [泽野弘之](http://music.163.com/#/artist?id=15290)曾经这样说过(其实并没有)。

这篇文章也是这个道理，想介绍一下我在半年前开始做的脚手架工具。给文章起标题也是一件很困难的事，最开始想到的是「史上最优雅的脚手架工具」、「来自未来的脚手架」，不过这些未免显得太标题党了，华而不实我自己都很尴尬。最近我刚好做了个[新世纪福音战士标题卡生成器](https://github.com/egoist/evangelion-card)，于是就随机将几个关键词组合在一起 —— **新世纪福音脚手架**，意外地显得还不错。

![sao](https://ooo.0o0.ooo/2017/05/21/5921898822cae.png)

**为了不让你对标题不明所以，简单地说明了它的来历，以下才是正文。**

## 脚手架

经常造轮子就会发现脚手架的重要性，这也是为什么 Yeoman 的发明者之一是 [@sindresorhus](https://github.com/sindresorhus) 的原因。后者已经在 npm 上发布了[超过 1000 个模块](https://www.npmjs.com/~sindresorhus)，很难后有来者了。

Yeoman 十分健壮，生态繁荣，然而要写[一个 generator 的复杂度](https://github.com/sindresorhus/generator-nm/blob/master/app/index.js)和写普通的代码几乎是差不多的，而我在能尽可能减少思考的时候就想减少思考，vue-cli 的思路很好地解决了我想减少思考的诉求，一个 generator 中间生成文件的过程有很多步骤是可以自动解决的。

vue-cli 虽然名字里有 vue 属性，但是作为任意类型项目的脚手架工具都是可以的，尽管运行 `vue init react` 这样的命令会显得有些奇怪。这也是为什么我做了 [SAO](https://github.com/egoist/sao) 的原因，一个类似 vue-cli 的脚手架工具。在拥有 vue-cli 的功能的同时，它也能像 Yeoman 一样用 npm package 作为模板并支持测试。

举个例子，在运行 `sao vue` 的时候，如果 template-vue 这个 npm 模块没有全局安装，它会提示你安装，之后再使用模板根目录里的配置文件 `sao.js` 将同目录里的 `template/` 中的文件生成到 `process.cwd()` 即当前目录。如果不存在配置文件，那么只会当成一个普通的目录，简单地复制粘贴到当前目录。

📄 **template-vue/sao.js:**

```js
module.exports = {
  // 从用户获取一些信息
  prompts: {
    pwa: {
      type: 'confirm',
      message: 'Add Progressive Web App support',
      default: true
    }
  },
  // 如果要发布到 npm
  // .gitignore 会自动被 npm 更名为 .npmignore
  // 为了避免这种情况需要起个另外的名字
  // 然后在生成的时候改名为 .gitignore
  move: {
    gitignore: '.gitignore'
  },
  // 只在用户确认了 pwa 选项的时候生成 pwa.js
  filters: {
    'pwa.js': 'pwa'
  }
}
```

上面的这个配置文件满足了大部分脚手架的需求，即从用户获取信息 --> 根据此信息生成需要的文件。而且几乎与代码无关，这个配置文件完全是由**数据**组成的，只不过刚好是以 JS 对象的格式。

![preview](https://ooo.0o0.ooo/2017/05/21/59218de93485b.png)

SAO 接收的第一个参数可以是:

- 本地模板路径，比如 `./my-template` `/path/to/my-template`。
- GitHub 项目缩略名，比如 `egoist/template-vue`。
- npm 模块名(自动加上 `template-` 前缀)，比如 `vue` 将会使用 npm 上的 `template-vue` 这个包。

而第二个参数是可选的，不存在时将会生成文件到工作区目录（当前目录），否则将会生成到指定的文件夹中。

## 测试脚手架

当脚手架变得复杂，你需要系统地测试以便让其在各种情况下都能生成正确的文件。对于一个脚手架，能从用户影响到它的变量只有 `prompts` 这个参数，也就是从用户获取的信息。而 SAO 的测试也主要是围绕这个来的，你可以模拟用户输入来检测生成结果。

📄 **template-vue/test.js:**

```js
import test from 'ava'
import sao from 'sao'

test('generate pwa entry', async t => {
  const template = process.cwd() // 模板根目录
  const res = await sao.mockPrompt(template, {
    // 模拟的 prompts 数据
    // 默认使用 `prompts` 中的默认值
    // 在上面的 `sao.js` 中 `pwa` 默认为 `true`
  })
  t.true(res.fileList.includes('pwa.js'))
})

test('ignore pwa entry', async t => {
  const template = process.cwd() // 模板根目录
  const res = await sao.mockPrompt(template, {
    pwa: false
  })
  t.false(res.fileList.includes('pwa.js'))
})
```

这里的 `res.fileList` 是生成的文件列表，形如:

```js
[
  '.gitignore',
  'pwa.js',
  'src/index.js'
]
```

以及 `res.files`，包含了生成文件的信息:

```js
{
  '.gitignore': {
    contents: Buffer,
    stats: {}, // fs.Stats,
    path: '/absolute/path/to/this/file'
  },
  // ...
}
```

## 最后

分享几个我自己经常使用的模板:

- [template-nm](https://github.com/egoist/template-nm): 生成一个 npm 模块，我的所有模块都是用这个生成的。
- [template-vue](https://github.com/egoist/template-vue): 生成一个*几乎*无需配置的 Vue 项目，基于 [Poi](https://poi.js.org)。
- [awesome-sao](https://github.com/egoist/awesome-sao): 相关 SAO 资源。

关于更多 SAO 的使用方法和配置文件参数，可以访问 https://sao.js.org :P 虽然本文标题是来源于新世纪福音战士，但 SAO 显然是来源于 [Sword Art Online](https://zh.moegirl.org/zh-hans/%E5%88%80%E5%89%91%E7%A5%9E%E5%9F%9F) 的。