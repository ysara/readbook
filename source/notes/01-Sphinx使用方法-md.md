# Sphinx-md

[Docs](http://www.sphinx-doc.org/en/1.4.8/contents.html)

## 安裝

```shell
pip install Sphinx
```

### 支持markdown

[http://www.sphinx-doc.org/en/stable/markdown.html](http://www.sphinx-doc.org/en/stable/markdown.html)

安装 `recommonmark`

```shell
pip install recommonmark
```

添加 `Markdown parser` 到 Sphinx 配置文件变量 `source_parsers` 中 :

```python
from recommonmark.parser import CommonMarkParser

source_parsers = {
    '.md': CommonMarkParser,
}
```

我们也可设置 `.md` 后缀为其他名字.

添加 `Markdown filename extension` 到配置文件前缀 `source_suffix` 配置变量中:

```python
source_suffix = ['.rst', '.md']
```

You can further configure recommonmark to allow custom syntax that standard CommonMark doesn’t support. Read more in the recommonmark documentation.

### sphinx-autobuild

自动生成html, 并启动server

```shell
# 安装sphinx-autobuild
pip install sphinx-autobuild
# 监听 source 目录, 生成的文件放置于 build/html
sphinx-autobuild source build/html
```

### theme

```shell
# 安装
pip install sphinx_rtd_theme

# 配置
import sphinx_rtd_theme
html_theme = 'sphinx_rtd_theme'
```

[HTML theming support](http://www.sphinx-doc.org/en/stable/theming.html)

- [sphinx_rtd_theme](https://pypi.python.org/pypi/sphinx_rtd_theme)
- [sphinx-bootstrap-theme](https://github.com/ryan-roemer/sphinx-bootstrap-theme)
