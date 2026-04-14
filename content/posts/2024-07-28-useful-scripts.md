---
title: "常用脚本集合"
date: 2024-07-28
draft: false
---

\\n

删除本目录下所有的超过一周未修改的分支

\\n\\n\\n\\n

``` wp-block-code
#!/bin/bash\n\n# 获取当前时间的Unix时间戳\ncurrent_time=$(date +%s)\n\n# 遍历所有本地分支\nfor branch in $(git branch --format='%(refname:short)'); do\n  # 获取分支最后一次编辑的时间的Unix时间戳\n  echo "Check $branch"\n  last_commit_time=$(git reflog show --date=unix "$branch" | grep -o '@{[0-9]\\+}' | head -n 1 | awk -F '[{}]' '{print $2}')\n\n  # 计算时间差\n  time_diff=$((current_time - last_commit_time))\n  \n  # 如果时间差大于一周（7天），则删除这个分支\n  if [ $time_diff -gt $((7*24*60*60)) ]; then\n    days=$((time_diff / (24*60*60)))\n    echo "Deleting branch '$branch' (last edited $days days ago)"\n    # 确认\n    confirm="n"\n    read -p "Are you sure? (y/n) " confirm\n    if [ "$confirm" != "y" ]; then\n      continue\n    fi\n    git branch -D "$branch"\n  fi\n  echo ""\ndone\n
```

\\n
