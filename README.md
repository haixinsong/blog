# blog

> This is where my blog markdown files and other informations store，it's useless for other people  
> 这是我存储我博客的markdown文件和其他资料的仓库,对他人无用

## 仓库目的

同步保留所有博客内容，以期其具有高度的可迁移性

## git commit格式

```bash
<operation>(<type>): <info>
```

示例：`update(rep): update readme`

### operation

可选值如下：

1. add 增加文件
2. update 更新文件(被更新的内容无错误)
3. fix 修复错误信息
4. del 删除文件

何为错误：非时效性内容出现不正确的信息

### type

可选值如下：

1. img 图片文件
2. post 博客markdown文件
3. page 博客特殊页markdown文件
4. rep 更新仓库自身信息

### info

附加信息，备注，少于65个字符
