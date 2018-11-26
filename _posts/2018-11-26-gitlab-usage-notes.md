---
layout: default
title: gitlab 使用笔记
---

Gitlab 使用笔记
===============


CI/CD
-----

### CI / CD 遇到使用`$` 变量

**Settings** > **CI / CD** > **Variables** 设置变量如果值存在 `$`, 需要使用 `$$` 替换.
  (`MY_$_VARIABLE` => `MY_$$_VARIABLE`)

### CI 配置文件, 可使用配置合并

CI 配置 如果遇到, 可重复的配置, 可使用引用继承方式

```yaml
.tags: &tags
  tags:
   - node
   - php
.cache_paths: &cache_paths
  paths:
    - vendor/
    - node_modules/
    - yarn.lock

deploy:prod:
  <<: *tags
  stage: deploy
  cache:
    key: prod
    <<: *cache_paths
```
