---
title: vscode配置
date: 2020-07-23 17:06:35
tags: [vscode]
categories: 软件配置
toc: true
---

# phpcs

## 1. 安装

这里使用 `composer` 安装<!-- more -->

### 全局安装

```
composer global require squizlabs/php_codesniffer
```

### 局部安装

```
composer require --dev squizlabs/php_codesniffer
```

- 局部安装是在项目的根目录

## 2. 安装 vscode 插件

打开 Visual Studio Code 搜索 `phpcs` 进行安装。

安装成功后点击启用就行了，或者在配置文件中配置

```json
"phpcs.enable" : "true"
```

> 安装后重启 VSCode 如果还提示需要安装或者全局安装`phpcs`，检查一下环境变量，将`phpcs`添加到环境变量

## 3. 配置参数

在 `setting.json` 中配置参数，配置代码的验证标准

```json
"phpcs.standard": "PSR2"
```

- 代码的标准有：MySource`, `PEAR`, `PHPCS`, `PSR1`, `PSR2`, `Squiz`, `Zend 等
- 可以在自己 phpcs 的安装目录中看自己安装了哪些 phpcs 标准
  - 使用 Composer 全局安装的话，phpcs 目录也是在 composer 目录下的
  - 文件目录在`xxx\Composer\vendor\squizlabs\php_codesniffer\src\Standards`

这样基本配置就完成了

---

其他的配置标准：

- 没有

  ```json
  "phpcs.standard": null
  ```

- 使用默认的标准

  ```json
  "phpcs.standard": ""
  ```

  - 设置默认标准，执行

    ```
    phpcs --config-set default_standard <标准名>
    ```

更多配置去官网看文档。[pear](https://pear.php.net/package/PHP_CodeSniffer/)

## 4. vscode 中其他的 PHP 配置

### 配置 PHP 执行路径

```json
"php.validate.executablePath": "xxx/php/php7.3.4nts/php.exe",
```

- 这里使用的是绝对路径
