---
title: "Cherry Pick使用隨筆"
date: 2024-04-11T03:59:35+08:00
tags:
 - git
 - cherry-pick
 - 隨筆紀錄
draft: false
---
### 前情提要
前幾天在工作的時候開發了一個feature1並且送了 Pull Request, 但是到今天為止都還沒有被 Review, 這時候我需要繼續開發下一個feature2。這個feature2需要依賴於feature1的修改, 我不想把兩個feature塞在同一個PR裡面（盡量別，體諒code reviewer 人人有責）又不想等到feature1被merge才能繼續開發feature2。  

commit 狀態假設是這樣:
```shell
A->B->C->D->E->F->G
```
我想要拆成
```git
        /->D->E # feature1
A->B->C
        \->F->G # feature2
```

### Cherry Pick
先開一個新的 branch 退版到 commit C
```shell
# dev branch A->B->C->D->E->F->G
# tmp branch A->B->C

git checkout -b tmp <hash-of-commit-C>
git cherry-pick <hash-of-commit-F> <hash-of-commit-G>
```

dev branch 再退到E, 然後 dev 跟 tmp 就可以分開送PR了