# Linux文件操作



基础操作

```bash
mv  # 重命名 或 移动
rm -rf dir  # 删除整个目录
cp -r src dst # 复制整个目录
mkdir -p a/b/c # 创建多级目录
chmod -R 777 dir  # 整个目录递归修改权限
chown
cat *.js > merged.js  # 按顺序输出多个文件内容，合并到新文件
touch
nano
vim
ln -s src dst  # 创建软链接dst，指向src
```

文件大小统计

```bash
du -sh              # 当前目录大小
du -h --max-depth=1 # 当前一级目录大小
du -sh --exclude PATTERN # 排除指定文件后的大小
```

