## 性能优化在工作中的实践

### 场景一-滚动卡顿

#### 背景

在某模块的列表页，存在**开始事件截图**和**结束事件截图**的选项，点击图片会将该图片进行放大，当前页面的pageSize为10，产品反馈该页面加载以及滑动卡顿严重。

#### 分析

首先，利用浏览器中的performance进行性能查看，可以观察到以下结论

* DOMContentLoaded：2.23s
* finish：23.7s；

然后，利用fps工具查看当前页面的fps（command+shift+p，搜索frame即可），刷新页面可以看到：

* fps：21

#### 结论

当前问题主要归结于2点：

* finish时间过长；
* 页面fps过低

所以，需要做的是优化图片大小、更换加载策略；

#### 方案

前端侧采用了以下方案：

* 图片压缩：列表页展示缩略图；
* 懒加载：懒加载该缩略图；
* 预加载：点击‘缩略图’以及点击‘详情’，都需要加载原图，所以提前加载；
* 并发加载：预加载图片，采用并发加载；

#### 成效

| 指标        | 优化前 | 目标    | 优化后  | 备注 |
| ----------- | ------ | ------- | ------- | ---- |
| Finish Time | 20+    | 10-     | 8 - 9   |      |
| FPS         | 20-25  | 40 - 60 | 45 - 55 |      |

#### 最后

总体上，达成了优化目标，最影响体验的卡顿问题得以解决。



### 场景二-流水线

#### 背景

在某天上线需求时，**流水线**崩了，也就是一直发布失败，进而联系运维以及其他人员，在凌辰4点左右解决流水线问题后，终于上线成功。通过观察流水线发布记录，可以发现在当日上线之前的**一周**以前，流水线构建的失败率就开始提升，在我上线需求时，失败率飚高达到80%+。

#### 分析

通过日志分析、运维侧人员以及谷歌搜索的查找，可以发现是由于内存问题所引起的崩溃问题。

![node_memory](D:\blog\chapter\性能优化\node_memory.png)

进而，通过利用mac的活动监视器来查看build时的CPU状况，可以看到：

* 内存使用情况在1.83G左右；

查看流水线设置，发现当前采用的node版本为10，所以推测是**node版本引起的内存溢出问题**。

更进一步，通过流水线发布记录，发现之前上线的版本中增加了新依赖：vue-code-diff，猜测是因为该依赖问题，所以利用webpack-bundle-analyzer分析打包体积，可以发现：

* vue-code-diff：压缩后的大小为1.33M

可以看出，确实该依赖体积比较大。

为了验证该猜测，拉取上线该需求之前的代码进行build打包，通过观察内存大小，发现内存大小在1.4G左右。

**需要注意的是**，node版本存在内存限制问题（上表的值存在波动范围，并不是严格阈值）。

#### 结论

问题可以归结为：

* 在上线vue-code-diff之后，整个项目的内存就即将达到node 10内存限制的阈值；
* 在上线我需求时，内存溢出了，导致整个一系列问题；

#### 方案

主要有以下几种解决方式：

* 修改node 版本：node 版本改为了12；
* 增加node内存：“build” ： “node --man_old_space_size = 1024 * 2 node_modules/.bin/vue-cli-service build”；
* 代码或依赖优化：按需加载（lodash、echart、MTD、element ui）、DLL（axios、vue）；
* webpack插件：写了一个注释文件大小的插件，可以清晰地看见每个文件的大小；

```
class FileSizePlugin {
  apply(compiler) {
    compiler.hooks.emit.tapAsync('FileSizePlugin', (compilation, callback) => {
      Object.keys(compilation.assets).forEach((filename) => {
        const source = compilation.assets[filename].source();
        const size = compilation.assets[filename].size() / 1024.0;
        const comment = `/* ${filename} file is ${size.toFixed(2)} Kb */\n\n`;
        compilation.assets[filename].source = () => (comment + source);
      });
      callback();
    });
  }
}

module.exports = FileSizePlugin;
```

#### 最后

总体上，提升了自己定位、思考、解决问题的能力。



### 场景三-全量加载

#### 背景

某一个页面，全量加载所有仓店数据（60000+），导致页面加载、渲染等卡顿问题。