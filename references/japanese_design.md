# 日本語 schema 设计说明

本文档针对 `japanese.schema.yaml`，按照 `references/Rime_description.md` 的结构梳理默认日文方案的构成与设计考量，便于后续维护或派生变种。

## 1. 目标与定位
- 提供基于罗马字的日语输入体验，候选覆盖汉字、平假名、片假名（来自 JMdict/Japanese 字典）。
- 保留与中文方案类似的自定义开关、反查、简繁转换等 Rime 能力，方便多语言用户切换。
- 通过额外的普通话/汉喃/韩文反查，满足跨语言检索需求。

## 2. schema 元信息
- `schema_id: japanese`：供 `schema_list` 引用的内部名称。
- `name: 日本語`：显示在 Rime 菜单/状态栏。
- `version: v0.2`：与词库版本一致。
- `author`：原作者 `ensigma96`，若有人修改应在列表中追加。
- `dependencies`：
  - `terra_pinyin.extended`：供普通话反查 (`putonghua_to_kanji_lookup`) 使用。
  - `stroke`：用于笔画反查。

## 3. 开关配置（switches）
- `ascii_mode`：中/西文切换，默认中文（`reset: 0`）。
- `full_shape`：半角/全角符号。
- `simplification`：汉字简繁转换（默认繁体）。
- `ascii_punct`：中西文标点。
- 其余注释掉的 `options` 示范了如何扩展字形切换，可按 Rime_description 的指引启用。

## 4. 引擎堆栈
- **processors**：`ascii_composer`、`recognizer`、`key_binder`、`speller`、`punctuator`、`selector`、`navigator`、`express_editor`；覆盖按键捕获、拼写、候选选择与上屏。
- **segmentors**：
  - 基础段器：`ascii_segmentor`、`matcher`、`abc_segmentor`、`punct_segmentor`、`fallback_segmentor`。
  - 反查段器：`affix_segmentor@putonghua_to_kanji_lookup / hannom_lookup / hanja_lookup`，借由前缀（如 `` ` ``）进入对应反查流程。
- **translators**：
  - 主体：`script_translator`（罗马字→日语词典）。
  - 反查：`reverse_lookup_translator` 及三种 `script_translator@...` 搭配各自字典。
  - `punct_translator` 负责单键标点直出。
- **filters**：
  - `simplifier`：字形/简繁转换（与 `simplification` 开关联动）。
  - `uniquifier`：去除重复候选。
  - `reverse_lookup_filter@...`：为候选附加来自普通话/汉喃/韩文的提示。

## 5. Speller 与按键代数
- `alphabet: 'zyxwvutsrqponmlkjihgfedcba-_'`：允许的输入字符集合。
- `delimiter: " '"`：允许空格或 `'` 断词。
- `algebra`：
  - `derive/_/x/`：把 `_` 映射为 `x`，便于处理带下划线的输入。
  - `derive/-/q/`：`-` 映射为 `q`，用于长音/促音处理。
  - `derive/i_e/ye/`（可扩展至 `derive/fu/hu/` 等）：解决特定罗马字到假名的变体。
- 这些代数规则遵循参考文档建议：先对按键序列做正则替换，再交由 `script_translator` 查字典。

## 6. 译码配置
- `translator.dictionary: japanese`：主词典，来自 `japanese.dict.yaml`（含基本词汇）及 `japanese.jmdict.dict.yaml`（JMdict 大词库）。
- `spelling_hints: 5`：为前 5 个候选显示拼写提示。
- `comment_format` 与 `preedit_format` 使用 `xlit|q|ー|` —— 在候选与预编辑区将字母 `q` 显示为长音符号 `ー`，提示用户其特殊含义。

## 7. 反查模块
- `putonghua_to_kanji_lookup`：
  - `prefix: "`"`，使用 `terra_pinyin.extended` 字典。
  - `prism: td_pinyin_flypy` 及一组 `xform` 规则，用于将速记拼音还原为标准拼音并标调。
  - 对应的 `reverse_lookup_filter@putonghua_to_kanji_reverse_lookup` 把拼音注释显示在日语候选上。
- `hannom_lookup` / `hanja_lookup`：分别基于 `hannomPS` 与 `hangyl` 字典，prefix 为 `` `V``/`` `K``，支持越南语汉喃及朝鲜汉字反查。
- `reverse_lookup`（笔画）：使用 `stroke` 字典，`prefix: "`H"`，`xlit/hspnz/一丨丿丶乙/` 把笔画序列可视化。

## 8. 键绑定
- `key_binder.bindings`：
  - `Control+Shift+1` → `.next`：在 schema 间循环。
  - `Control+Shift+n` / `Control+Shift+N` → `select: japanese`：直接切回日语方案。
- 绑定遵循参考文档中的 `key_binder` 配置方式，可在自定义文件中覆盖或扩展。

## 9. 部署与自定义
- README 建议通过 `plum` 安装后在 `default.custom.yaml` 中追加：

```yaml
patch:
  schema_list:
    - schema: japanese
```

- 若要并列多个方案，可按 Rime_description 的做法把 `schema_list` 写成数组并指定顺序。
- `simplifier@jp_variants`、`zh_simp`、`zh_tw` 等可选滤镜已在 schema 中注释示例，启用时需同步在 `switches` 中添加对应 `options` 与 `states`。

## 10. 维护要点
- 新增词汇：编辑 `*.dict.yaml` 后运行 `rime_dict_manager` 或重新部署以编译 `*.bin`。
- 自定义罗马字规则：在 `speller.algebra` 追加 `derive`/`abbrev`/`xlit`，遵循参考文档中“按键代数”规范，避免破坏现有规则顺序。
- 反查扩展：仿照既有 `affix_segmentor@...` + `script_translator@...` + `reverse_lookup_filter@...` 套件，确保 `prefix` 不冲突并在 `tips`/`comment_format` 中说明用途。

通过上述结构，`japanese` 方案实现了“罗马字 → 日语”主路径与多语言辅助反查，用户可在此基础上进一步定制键位或词库，同时保持与 Rime 生态中其它方案一致的组织方式。
