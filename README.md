# riot-pkg

[Riot](https://github.com/caiwuu/Riot) 的可下载能力包。

Riot 主程序里不带 Python、Node、LibreOffice 这些东西 —— 装机量最大的那部分用户
根本用不到文档处理，为它们把安装包撑到几百 MB 不划算。需要的人在设置里点一下
安装，运行时按平台下载对应的包。

## 仓库布局

```
packs.json                     发布清单，Riot 运行时读的就是它
doc-runtime/
  darwin-arm64/
    packs.json                 这个平台构建时产出的清单片段
    doc-runtime-0.1.0-darwin-arm64.tar.zst
  win-x64/
    packs.json
    doc-runtime-0.1.0-win-x64.tar.zst
```

包名在平台上面 —— 这个仓库以后不止装文档一个能力，按平台分在最外层的话，一个包
的东西会散在各平台目录里，想知道仓库里有哪些能力得把每个平台目录都翻一遍。

`.tar.zst` **不在 git 里**（见 `.gitignore`）。GitHub 拒收超过 100MB 的单个文件，
而一个包两百多 MB；Git LFS 的免费额度也撑不住这个量级的下载。包体走 Releases，
仓库里只留清单。

包必须在对应平台的机器上制作 —— `skia.node` 之类的原生绑定按平台编译，没法交叉
产出。两台机器各写各的平台目录，`publish.mjs` 再把两份清单并成根目录那一份。

## 现有的包

| 包 | 内容 | 压缩后 | 安装后 |
| --- | --- | --- | --- |
| `doc-runtime` | Python + Node + LibreOffice + Poppler，以及 docx/xlsx/pptx/pdf 四个 skill | ~235MB | ~900MB |

## 发布

在 Riot 仓库里：

```bash
# macOS 上
node scripts/build-doc-pack.mjs

# Windows 上
pwsh scripts/build-doc-pack.ps1

# 任一台机器上，等两个平台的产物都到齐之后
node scripts/doc-pack/publish.mjs
```

`publish.mjs` 会合并清单、比对每个包的 sha256、把 `.tar.zst` 传到 Releases，最后
提交 `packs.json`。清单最后推：Riot 一读到新清单就会去下对应的资产。
