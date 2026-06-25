# UED-vibe 设计陈列

公网链接:**https://zhangwan-ued-vibe.surge.sh/**
仓库源:https://github.com/zn453713682-glitch/ued-vibe

## 目录约定

```
vibe-gallery/
├── index.html              # 壳子,不要手改 MANIFEST
├── manifest.js             # 由 build-manifest.mjs 自动生成
├── build-manifest.mjs      # 扫子目录生成 manifest 的 node 脚本
└── {系统}/{版本}/          # 设计师产物落这里
    ├── *.html              # *.inline.html 也行(inline = 资源已内联)
    └── *.md                # 同名 md 自动配对(`foo.html` ↔ `foo.md`)
```

例:
```
灵翼/v1.0/作品管理列表-老黑-v3.inline.html
灵翼/v1.0/作品管理列表-老黑-v3.md
灵翼/v1.1/...
项目B/v0.1/...
```

> 顶层散落的 *.html(没有放进 {系统}/{版本}/ 子目录的)兜底归到 `灵翼/v1.0`。

## 本地预览 / 手动发布

```bash
node build-manifest.mjs   # 重写 manifest.js
open index.html           # 本地看效果(file://)
surge . zhangwan-ued-vibe.surge.sh   # 发布到公网
git add -A && git commit -m "xxx" && git push   # 推仓库做版本历史
```

## 多人协作

- 每个设计师 clone 仓库,在自己分支(或直接 main)放产物到 `{系统}/{版本}/` 下。
- 推送前 `git pull --rebase`,文件级 merge 不冲突。
- 完整发布流程已封装在 vibe-design skill 末尾,跑 skill 自动完成。
