# README

此分支是作为构建GitHub page的hexo-starter仓库的fork版本，自定义修改hexo的配置。

## 注意事项

发布时，需要把本仓库克隆下来，并且重命名为`.deploy_git`，因为[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git#how-it-works)在发布时是强制push的，所以为了维持提交的连续性，需要把本仓库克隆下来作为发布时的仓库使用。如果在不同的电脑上进行发布，一定要记得及时更新仓库。

## 常用命令

根据`scoffolds`里设置的模板，在`source/_posts`的目录里生成同名`.md`文件和同名文件夹：

```sh
npx hexo new galgame -p "galgame/Making Lovers FD1+2" "Making Lovers FD1+2"
```

生成：

```sh
npx hexo g
```

预览：

```sh
npx hexo s
```

发布：(注意每次发布前最后手动生成一次，避免文件未及时更新)

```sh
npx hexo d
```

## Node.js

当前版本：**v16.20.0**

本地初始化和升级，当你手动修改`package.json`升级一些模块时，需要重新安装，会自动根据`package.json`安装模块到本地，不要加`-g`就不会是全局安装

```sh
npm install
```

### hexo-asset-img

这是一个很重要的一个Nodejs模块，用于将md风格的图片插入转换成hexo风格的图片插入，当时有些特殊字符的转换有问题，个人修改了部分，把代码贴下避免丢失。源仓库地址：https://github.com/yiyungent/hexo-asset-img

```js index.js
const log = require('hexo-log')({ 'debug': true, 'slient': false });

/**
 * md文件返回 true
 * @param {*} data 
 */
function ignore(data) {
    // TODO: 好奇怪，试了一下, md返回true, 但却需要忽略 取反!
    var source = data.source;
    var ext = source.substring(source.lastIndexOf('.')).toLowerCase();
    return ['md',].indexOf(ext) > -1;
}

function action(data) {
    var reverseSource = data.source.split("").reverse().join("");
    var fileName = reverseSource.substring(3, reverseSource.indexOf("/")).split("").reverse().join("");

    // ![example](postname/example.jpg)  -->  {% asset_img example.jpg example %}
    var fileName = fileName.replace(RegExp("\\+", "g"), "\\+");
    var regExp = RegExp("!\\[(.*?)\\]\\((" + fileName + "|" + fileName.replace(RegExp(" ", "g"), "%20") + ')/(.+?g)\\)', "g");
    // hexo g
    data.content = data.content.replace(regExp, function (matchStr, group1, group2, group3) {
        var imgname = group3.replace(RegExp("%20", "g"), " ");
        return `{% asset_img "${imgname}" "${group1}" %}`;
    });

    log.info(`hexo-asset-img: filename: ${fileName}, title: ${data.title.trim()}`);
    //log.info(`hexo-asset-img: filename: ${fileName}, title: ${data.content}`);

    return data;
}

hexo.extend.filter.register('before_post_render', (data) => {
    if (!ignore(data)) {
        action(data)
    }
}, 0);
```

## Links

- https://github.com/hexojs/hexo-starter
- https://github.com/hexojs/hexo-deployer-git
