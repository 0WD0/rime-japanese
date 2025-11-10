# 日本語（大西配列）设计说明

本文档以 `japanese_onishi.schema.yaml` 为对象，参考 `references/Rime_description.md` 给出的 Rime 方案结构，说明如何在现有日文方案上实现大西配列（Onishi Layout）。

## 1. 目标与背景
- 在物理 QWERTY 键盘上复刻大西配列的键位排列，同时复用原 `japanese` 方案的词典和处理流程。
- 通过 Rime `speller.algebra` 做按键到罗马字的转换，避免改动字典、翻译器或滤镜，控制变更范围。
- 让用户能像切换普通 schema 一样切换到大西配列，部署步骤与原方案保持一致。

## 2. schema 描述
- `schema_id: japanese_onishi`，用独立 ID 方便在 `schema_list` 中并列配置。
- `name: 日本語（大西配列）`，在候选栏或方案菜单中能与“日本語”区分。
- `version: v0.2-onishi`，表明基于 v0.2 的派生。
- `description` 指出“在罗马字分解前插入 Onishi 键位映射”这一核心差异。
- `author`、`dependencies`（`terra_pinyin.extended`、`stroke`）继承原方案，保证反查和笔画词典仍可使用。

## 3. 开关（switches）
保留 `ascii_mode / full_shape / simplification / ascii_punct` 四个开关，并按照参考文档所述配置 `states`，以便在状态栏显示“中文/西文”“半角/全角”等人类可读字符串。大西配列只影响按键排列，不需要额外选项。

## 4. 引擎堆栈
`processors、segmentors、translators、filters` 完全沿用 `japanese.schema.yaml`，原因：
- 反查 (`putonghua_to_kanji_lookup`、`hannom_lookup`、`hanja_lookup`) 的 prefix/suffix 逻辑不受键盘映射影响。
- `simplifier`、`uniquifier` 等过滤器负责候选去重、简繁转换，仍需保持顺序与配置一致。

## 5. Speller 与大西配列
关键改动位于 `speller.algebra` 的首条 `xlit` 规则：

```
xlit|qwertyuiopasdfghjkl;zxcvbnm,./QWERTYUIOPASDFGHJKL:ZXCVBNM<>?|qlu,.fwrypeiao-ktnshzxcv;gdmjbQLU<>FWRYPEIAO_KTNSHZXCV:GDMJB|
```

- `source` 一侧列出主键区全部 QWERTY 物理键（含大小写及常用标点），`target` 一侧按大西配列顺序列出对应字符。字符数必须一致，Rime 才能逐字符完成映射。
- 该 `xlit` 放在所有 `derive` 规则之前，使后续 `derive/_/x/`、`derive/-/q/`、`derive/i_e/ye/` 等罗马字扩展仍按原逻辑执行。
- 若使用 JIS 键盘并希望覆盖 `＠` `［` `］` `￥` 等额外键，只需在 `source/target` 两串末尾按相同顺序追加字符，确保长度匹配。

## 6. 词典与译码
`translator.dictionary` 继续指向 `japanese`，沿用 `japanese.dict.yaml`、`japanese.jmdict.dict.yaml` 等现有词库。不需要额外的词典转换，候选、注释（例如 `comment_format: xlit|q|ー|`）也与原方案一致。

## 7. 键绑定
`key_binder.bindings` 未改动，仍包含：
- `Control+Shift+1` → `.next`（轮换方案）
- `Control+Shift+n/N` → `select: japanese`

一旦在 `default.custom.yaml` 中把 `japanese_onishi` 注册为 schema，就可以通过 `.next` 快速切换至大西配列。

## 8. 部署与启用
README 建议在 `default.custom.yaml` 中配置：

```yaml
patch:
  schema_list:
    - schema: japanese
    - schema: japanese_onishi
```

随后重新部署（`Control`+`Shift`+`F` 或 Fcitx5 的“重新部署”按钮）。Rime 菜单会出现“日本語（大西配列）”，选择后即可在 QWERTY 键盘上使用大西配列输入日文。

## 9. 后续扩展建议
1. **特定硬件支持**：若需要映射到 106/109 键盘的额外键位，按参考文档的方式扩展 `xlit`。
2. **物理大西键盘**：若使用已经刻印为大西布局的键帽，可移除 `xlit` 行，避免重复转换。
3. **功能滤镜整合**：如需叠加日文特有滤镜（假名直出、频率模型等），请参考 `references/Rime_description.md` 的 filters 章节，按顺序插入并配置开关。

通过以上设计，我们在保持 Rime 栈稳定的前提下，仅用一条 `xlit` 规则和少量 metadata，即可为日文方案新增可选的大西配列输入体验。
