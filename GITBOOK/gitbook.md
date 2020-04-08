
# gitbook

- 安装

```
$ npm install gitbook-cli -g
```

- 验证安装

```
$ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

- 初始化

```
$ gitbook init
warn: no summary file in this book
info: create README.md
info: create SUMMARY.md
info: initialization is finished
```

- 配置

```json
{
    "title": "learn note",
    "author": "Eric W",
    "description": "select * from learn",
    "language": "zh-hans",
    "gitbook": "3.2.3",
    "structure": {
        "readme": "README.md"
    },
    "plugins": [
        "mermaid-gb3"
    ]
}
```

- 安装plugin

```
gitbook install
```

- 启动

```
$ gitbook serve
```

- 访问

```
http://localhost:4000
```
