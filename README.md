# my-sno-rime

本仓库是本地 Rime 配置（`~/Library/Rime`，Squirrel）中日语方案 `sno_japanese` 及其全部依赖的公开镜像，供 [My RIME](https://my-rime.vercel.app/)（浏览器版 Rime）在线部署，目标是使网页端日语输入与本地完全一致：罗马字输入假名与汉字、拼音输入日语汉字（候选附罗马音注释）、主段混输联想。文件平铺于仓库根目录，以适配 My RIME 的 `?plum=` 一键部署参数。

使用分为两个阶段：先将仓库构建并推送至 GitHub（一次性），此后在任意浏览器中一键部署。

## 方案一览

| 方案 | 说明 |
|---|---|
| `sno_japanese`（JPN 雪星日語） | 主力方案：主段同时给出日语罗马字、拼音和字（附罗马音注释）、五笔、英→日翻译、颜文字候选；前缀反查 P（拼音和字）、W（五笔汉字）、J（五笔日本汉字）、M（Emoji） |
| 依赖方案 | japanese、japanese_pinyin、pinyin_simp、sno_wubi、sno_wubi_kanji、sno_emoji、translate_en2jp。经主方案的 `web_deps` 适配节自动下载，无须列入部署参数、不占用方案菜单；如需作为独立方案选用，追加到部署参数即可 |

## 一、仓库构建

### 1. 从本地 Rime 目录拷贝文件

仓库内容直接取自本地配置，非上游仓库下载，以保证与本地行为一致：

```bash
R=~/Library/Rime
cp $R/{sno_japanese,japanese,japanese_pinyin,pinyin_simp,sno_wubi,sno_wubi_kanji,sno_emoji,translate_en2jp}.schema.yaml .
cp $R/{japanese,japanese_pinyin,japanese_pinyin_simp,japanese_chars,pinyin_simp,sno_wubi,sno_wubi_kanji,sno_emoji,translate_en2jp}.dict.yaml .
cp $R/japanese.{mozc,jmdict,kana}.dict.yaml .
mkdir -p opencc
cp $R/opencc/{zh_emoji,zhjp}.json opencc/
cp $R/opencc/{zh_emoji_word,zh_emoji_category,zhjp_word,zhjp_daily,zhjp_names}.txt opencc/
```

合计约 90MB，单文件最大约 30MB，低于 GitHub 单文件 100MB 上限。

### 2. 网页适配改动

仓库相对本地文件共两处改动，均不影响输入行为，同步本地文件后须保留：

其一，`sno_japanese.schema.yaml` 追加 `web_deps` 节。micro-plum（My RIME 的下载器）不解析 schema 的 `dependencies` 字段，但会追踪 `__include` 引用并递归下载；藉此让单个 schema id 触发全部依赖 schema 及词库的下载，部署参数因而只需列出主方案，方案菜单也只显示一项：

```yaml
# web 適配：micro-plum 不解析 dependencies，藉 __include 引用觸發依賴 schema
# 及其詞庫的遞歸下載；引擎僅在此節得到幾個無用的字符串，不影響輸入行為
web_deps:
  a: { __include: "japanese.schema:/schema/schema_id" }
  b: { __include: "japanese_pinyin.schema:/schema/schema_id" }
  c: { __include: "pinyin_simp.schema:/schema/schema_id" }
  d: { __include: "sno_wubi.schema:/schema/schema_id" }
  e: { __include: "sno_wubi_kanji.schema:/schema/schema_id" }
  f: { __include: "sno_emoji.schema:/schema/schema_id" }
  g: { __include: "translate_en2jp.schema:/schema/schema_id" }
```

其二，`sno_emoji.schema.yaml` 追加引擎不引用的顶层 `translator` 节。micro-plum 仅解析顶层 `translator.dictionary`，而该方案的词典声明位于 `sno_emoji_lookup` 节内，缺少此适配则词库文件不会被下载：

```yaml
# web 適配：引擎未引用此節，僅供 micro-plum 解析以下載詞典文件
translator:
  dictionary: sno_emoji
```

### 3. 提交并推送

```bash
git init -b main
git add .
git commit -m "Mirror local sno_japanese schemas for My RIME web deploy"
```

在 GitHub 创建空的 Public 仓库 `my-sno-rime`——micro-plum 经 raw.githubusercontent.com 匿名拉取文件，私有仓库无法访问；README、gitignore、license 均不初始化。随后关联推送：

```bash
git remote add origin git@github.com:221bmogl/my-sno-rime.git
git push -u origin main
```

词库文件超过 GitHub 网页上传的 25MB 限制，故须经 git push 提交。

## 二、网页部署

### 一键部署（推荐）

将下述链接存为书签，打开即自动完成下载、部署并选中 `sno_japanese`：

```
https://my-rime.vercel.app/?plum=221bmogl/my-sno-rime:sno_japanese
```

依赖 schema 与词库经 `web_deps` 适配节自动下载，方案菜单仅显示「JPN 雪星日語」一项。部署产物保存在浏览器站点存储中；站点数据被清空后，重新打开书签即可恢复。首次部署需下载约 90MB 并在浏览器内编译词库（日语 mozc 词库最大，约 30MB），耗时约数分钟，期间不应关闭标签页；此后走缓存即时可用。

### 手动安装

在 My RIME 界面打开 Micro Plum，选择 plum 目标方式，输入目标 `221bmogl/my-sno-rime` 与 schema id `sno_japanese`，Install 后 Deploy。

### 部署说明

- 部署过程中个别文件下载失败提示属预期现象：`jpzh.json`（日→汉注释，本地同样缺失）、`terra_pinyin.extended`、`stroke`、`default.yaml`、`symbols.yaml` 等（后两者由 My RIME 预置包提供），均不影响主体功能。
- 若浏览器中已存在同名方案，一键部署将跳过下载、直接使用现有文件；切换至仓库新版本须先清除站点数据。

## 三、与本地体验的已知差异

- 方案切换快捷键（Ctrl+comma / Ctrl+period / Ctrl+slash）定义于本地 `default.custom.yaml`，而网页部署会以方案列表覆盖该文件，故不可用，须以页面按钮切换方案。方案内快捷键 Shift+E / Shift+C（切至 sno_english / sno_chinese）因目标方案未部署而无效果。
- 用户词典与输入习惯（userdb）不随仓库迁移，网页端从零累积，且存于站点存储、随站点数据清空而丢失。

## 四、更新与维护

本地配置变更后，重复「一、1」的拷贝命令同步对应文件（注意保留「一、2」所述两处网页适配节），提交推送；浏览器端清除站点数据、重新打开一键部署书签即可生效。

## 来源与致谢

- 方案与词库：[snomiao/rime-snomiao](https://github.com/snomiao/rime-snomiao)（本地快照）
- 日语方案上游：[gkovacs/rime-japanese](https://github.com/gkovacs/rime-japanese)，词库源自 mozc 与 JMdict
- 拼音方案：Rime 朙月拼音（佛振）
- 运行环境：[LibreService/my_rime](https://github.com/LibreService/my_rime)

本仓库仅作个人在线部署之用。
