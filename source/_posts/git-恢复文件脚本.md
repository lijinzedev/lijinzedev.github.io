---
title: git 恢复文件脚本
top: false
cover: false
toc: true
mathjax: true
categories:
  - git
tags:
  - git
date: 2024-07-18 17:43:13
password:
summary:
---

使用方法：

将文件中可能包含的字符串输入进行检索

```
a.sh "java|import"
```



```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <search_strings>"
    exit 1
fi

SEARCH_STRINGS="$1"

# 运行 git fsck --lost-found 以识别丢失的对象
git fsck --lost-found

# 定义保存恢复文件的目录的绝对路径
RESTORED_DIR="$(pwd)/restored_files"
BLOBS_DIR="$RESTORED_DIR/blobs"
COMMITS_DIR="$RESTORED_DIR/commits"

# 创建目录以保存恢复的文件
mkdir -p "$BLOBS_DIR"
mkdir -p "$COMMITS_DIR"

# 检查并处理丢失的块对象
if [ -d .git/lost-found/other ]; then
    echo "Processing lost blobs..."
    cd .git/lost-found/other
    for file in *; do
        # 显示块对象内容并检查是否包含任意一个字符串
        if git show "$file" | grep -E -q "$SEARCH_STRINGS"; then
            # 如果包含任意一个字符串，则将内容保存到目标目录
            git show "$file" > "$BLOBS_DIR/$file"
            echo "Restored blob: $file"
        else
            echo "Blob $file does not contain any of the strings \"$SEARCH_STRINGS\""
        fi
    done
    cd - # 返回原始目录
else
    echo "No lost blobs found."
fi

# 检查并处理丢失的提交对象
if [ -d .git/lost-found/commit ]; then
    echo "Processing lost commits..."
    cd .git/lost-found/commit
    for file in *; do
        # 显示提交对象内容并检查是否包含任意一个字符串
        if git show "$file" | grep -E -q "$SEARCH_STRINGS"; then
            # 如果包含任意一个字符串，则将内容保存到目标目录
            git show "$file" > "$COMMITS_DIR/$file"
            echo "Restored commit: $file"
        else
            echo "Commit $file does not contain any of the strings \"$SEARCH_STRINGS\""
        fi
    done
    cd - # 返回原始目录
else
    echo "No lost commits found."
fi

echo "Filtered and restored files can be found in the restored_files directory."

```

