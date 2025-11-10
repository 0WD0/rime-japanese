# `Schema.yaml` 詳解

---

## 開始之前

```yaml
# Rime schema
# encoding: utf-8
```

## 描述檔

1. `name`：方案在選單中顯示的名稱，通常使用中文。
2. `schema_id`：方案內部 ID，供其它文件引用，慣例由英文、數字、底線組成。
3. `author`：作者或維護者，若在別人的方案上修改，請在原作者後追加自己的署名。
4. `description`：簡要說明方案歷史、碼表來源與主要規則。
5. `dependencies`：此方案所依賴的其它方案（常見於反查或混合輸入）。
6. `version`：版本號；釋出新版前請遞增。

**示例**
```yaml
schema:
  name: 蒼頡檢字法
  schema_id: cangjie6
  author:
    - 發明人 朱邦復先生、沈紅蓮女士
  dependencies:
    - luna_pinyin
    - jyutping
    - zyenpheng
  description: |
    第六代倉頡輸入法
    碼表由雪齋、惜緣和 crazy4u 整理
  version: 0.19
```

## 開關（`switches`）

常見開關如下，亦可因自訂濾鏡擴充：

1. `ascii_mode`：中/西文切換，`0` 表中文，`1` 表西文。
2. `full_shape`：半角/全角切換，開啓全角時字母亦為全角。
3. `extended_charset`：字符集切換（僅 `table_translator` 可用），`0` 為基本 CJK，`1` 為擴展 CJK。
4. `ascii_punct`：中/西文標點切換。
5. `simplification`：漢字轉換開關（簡/繁）。

> 開關名稱可與 `key_binder/bindings` 的 `toggle`、`set_option`、`unset_option` 配合。`states` 可省略；若省略則界面不顯示但仍可透過快捷鍵切換。`reset` 省略時，切換方案不會重設狀態。

**單選開關示例**
```yaml
- name: simplification
  states: [ 漢字, 汉字 ]
  reset: 0
```

**多選開關示例**
```yaml
- options: [ zh_trad, zh_cn, zh_mars ]
  states:
    - 字形 → 漢字
    - 字形 → 汉字
    - 字形 → 䕼茡
  reset: 0
```

**完整示例**
```yaml
switches:
  - name: ascii_mode
    reset: 0
    states: [ 中文, 西文 ]
  - name: full_shape
    states: [ 半角, 全角 ]
  - name: extended_charset
    states: [ 通用, 增廣 ]
  - name: simplification
    states: [ 漢字, 汉字 ]
  - name: ascii_punct
    states: [ 句讀, 符號 ]
```

## 引擎

> 加粗條目為常用配置，可細調；斜體條目較少用。

### 1. `processors`

處理各類按鍵訊息：

- `ascii_composer`：處理西文模式與中/西文切換。
- **`recognizer`**：與 `matcher` 配合識別符合特定規則的輸入（如網址、反查）。
- **`key_binder`**：綁定快捷鍵或改寫按鍵行為（候選翻頁、開關切換等）。
- **`speller`**：拼寫處理器，維護輸入串。
- **`punctuator`**：單鍵輸出標點或文本。
- `selector`：處理候選選擇與翻頁。
- `navigator`：處理輸入欄光標移動。
- `express_editor`：處理空格、回車上屏與退格。
- _`fluid_editor`_：替代 `express_editor`，用於語句流類輸入法。
- _`chord_composer`_：並擊輸入組件，供多鍵同擊方案使用。

### 2. `speller`

關鍵字段：

- `alphabet`：允許輸入的字符集合。
- `delimiter`：允許視為分隔符的字符（如空格、撇號）。
- `max_code_length`、`min_code_length`：限制碼長。
- `initial_quality`：拼寫質量基礎分。
- `use_space`：`true/false`，是否以空格作輸入碼。
- `algebra`：拼寫運算規則，依序處理。

`speller` 演算指令：

```
xform   # 改寫，不保留原形
derive  # 衍生，保留原形
abbrev  # 簡拼，優先級低於上兩者
fuzz    # 略拼，只參與組詞
xlit    # 一對一大量映射
erase   # 刪除
```

**示例（朙月拼音）**
```yaml
speller:
  alphabet: zyxwvutsrqponmlkjihgfedcba
  delimiter: " '"
  algebra:
    - erase/^xx$/
    - abbrev/^([a-z]).+$/$1/
    - abbrev/^([zcs]h).+$/$1/
    - derive/^([nl])ve$/$1ue/
    - derive/^([jqxy])u/$1v/
    - derive/un$/uen/
    - derive/ui$/uei/
    - derive/iu$/iou/
    - derive/([aeiou])ng$/$1gn/
    - derive/([dtngkhrzcs])o(u|ng)$/$1o/
    - derive/ong$/on/
    - derive/ao$/oa/
    - derive/([iu])a(o|ng?)$/a$1$2/
```

### 3. `segmentor`

- `tag`：為輸入段落標記標籤。
- `prefix`/`suffix`：透過前綴或後綴觸發。
- `tips`/`closing_tips`：輸入提示與結束提示。
- `extra_tags`：為該段落補充額外標籤。

> 若 `affix_segmentor` 與 `translator` 同名，可合併配置，`segmentor` 條目 1–5 對應 `translator` 條目 21–25；`abc_segmentor` 只能設定 `extra_tags`。

**示例**
```yaml
reverse_lookup:
  tag: reverse_lookup
  prefix: "`"
  suffix: ";"
  tips: 【反查】
  closing_tips: 【蒼頡】
  extra_tags:
    - pinyin_lookup
    - jyutping_lookup
```

### 4. `translator`

每個方案至少有一個主 `translator`（無 `@` 前綴）。常用字段：

1. `enable_charset_filter`
2. `enable_encoder`
3. `encode_commit_history`
4. `max_phrase_length`
5. `enable_completion`
6. `enable_correction`
7. `sentence_over_completion`
8. `strict_spelling`
9. `disable_user_dict_for_patterns`
10. `enable_sentence`
11. `enable_user_dict`
12. `dictionary`
13. `prism`
14. `user_dict`
15. `db_class`
16. `preedit_format`
17. `comment_format`
18. `spelling_hints`
19. `always_show_comments`
20. `initial_quality`
21. `tag`
22. `prefix`
23. `suffix`
24. `tips`
25. `closing_tips`
26. `contextual_suggestions`
27. `max_homophones`
28. `max_homographs`

> `enable_*` 欄位填 `true/false`。`prism` 指定拼寫棱鏡；副翻譯器引入既有拼寫結果時也用此名。

**主翻譯器示例（倉頡）**
```yaml
translator:
  dictionary: cangjie6
  enable_charset_filter: true
  enable_sentence: true
  enable_encoder: true
  encode_commit_history: true
  max_phrase_length: 5
  preedit_format:
    - xform/^([a-z ]*)$/$1｜\U$1\E/
    - xform/(?<=[a-z])\s(?=[a-z])//
    - "xlit|ABCDEFGHIJKLMNOPQRSTUVWXYZ|日月金木水火土竹戈十大中一弓人心手口尸廿山女田止卜片|"
  comment_format:
    - "xlit|abcdefghijklmnopqrstuvwxyz~|日月金木水火土竹戈十大中一弓人心手口尸廿山女田止卜片・|"
  disable_user_dict_for_patterns:
    - '^z.*$'
  initial_quality: 0.75
```

**副翻譯器示例（拼音）**
```yaml
pinyin:
  tag: pinyin
  dictionary: luna_pinyin
  prism: luna_pinyin_simp
  tips: 【漢拼】
  closing_tips: 【蒼頡】
```

**拼音簡化字主翻譯器**
```yaml
translator:
  dictionary: luna_pinyin
  prism: luna_pinyin_simp
  preedit_format:
    - xform/([nl])v/$1ü/
    - xform/([nl])ue/$1üe/
    - xform/([jqxy])v/$1u/
```

**用戶短語（table_translator）**
```yaml
custom_phrase:
  dictionary: ""
  user_dict: custom_phrase
  db_class: tabledb
  enable_sentence: false
  enable_completion: false
  initial_quality: 1
```

### 5. `reverse_lookup_filter`

- `tags`：限定作用的翻譯器。
- `overwrite_comment`：是否覆蓋原提示。
- `dictionary`：反查用碼表。
- `comment_format`：提示格式。
- `apply_comment`：限制套用條件（可選）。

**示例**
```yaml
pinyin_reverse_lookup:
  tags: [ pinyin_lookup ]
  overwrite_comment: true
  dictionary: cangjie6
  comment_format:
    - xform/$/〕/
    - xform/^/〔/
    - "xlit|abcdefghijklmnopqrstuvwxyz |日月金木水火土竹戈十大中一弓人心手口尸廿山女田止卜片、|"
```

### 6. `simplifier`

1. `option_name`：對應 `switches`/`key_binder` 中的名字。
2. `opencc_config`：位於 `rime_dir/opencc/`，常見如 `t2s.json`、`t2tw.json`、`t2hk.json`、`s2t.json`。
3. `tags`：限定作用範圍。
4. `tips`：是否顯示轉換提示（`none`/`char`/`all`）。
5. `comment_format`
6. `allow_erase_comment`
7. `show_in_comment`
8. _`excluded_types`_：排除特定翻譯器。

**示例**
```yaml
zh_tw:
  option_name: zh_tw
  opencc_config: t2tw.json
  tags: [ abc ]
  tips: none
  allow_erase_comment: true
  comment_format:
    - xform/.*//
```

### 7. `chord_composer`

- `alphabet`：並擊使用的按鍵集合。
- `algebra`：將並擊編碼轉換為音節。
- `output_format`：並擊結束後套用（如追加隔音符）。
- `prompt_format`：並擊過程中的視覺提示。

**示例（宮保拼音）**
```yaml
chord_composer:
  alphabet: "swxdecfrvgtbnjum ki,lo."
  algebra:
    - 'xlit|swxdecfrvgtbnjum ki,lo.|sczhlfgdbktpRiuVaNIUeoE|'
    - xform/^zf/zh/
    - xform/^cl/ch/
    - xform/^fb/m/
    - xform/^ld/n/
    - xform/^hg/r/
    - xform/^([bpf])$/$1u/
    - xform/^([mdtnlgkh])$/$1e/
    - xform/^([zcsr]h?)$/$1i/
  output_format:
    - "xform/^([a-z]+)$/\\1'/"
  prompt_format:
    - "xform/^(.*)$/[$1]/"
```

### 8. `lua`

可定義 `lua_translator`、`lua_filter`、`lua_processor`、`lua_segmentor`。詳見 [hchunhui/librime-lua](https://github.com/hchunhui/librime-lua)。

**示例**
```lua
function get_date(input, seg, env)
  local on = env.engine.context:get_option("show_date")
  if on and input == "date" then
    yield(Candidate("date", seg.start, seg._end, os.date("%Y年%m月%d日"), " 日期"))
  end
end

function single_char_first(input, env)
  local on = env.engine.context:get_option("single_char")
  local cache = {}
  for cand in input:iter() do
    if (not on or utf8.len(cand.text) == 1) then
      yield(cand)
    else
      table.insert(cache, cand)
    end
  end
  for _, cand in ipairs(cache) do
    yield(cand)
  end
end
```

### 9. 其它引擎設置

涵蓋 `recognizer`、`key_binder`、`punctuator` 等。

- `import_preset`：統一引用外部設定。
- `grammar`：包含 `language`（如 `zh-hant-t-essay-bgw`）、`collocation_max_length`、`collocation_min_length`。
- `recognizer.patterns`：搭配 `segmentor` 的 `prefix/suffix` 分配標籤；可匹配 `reverse_lookup`、`punct` 或自定字段。
- `key_binder.bindings`：每條含 `when`、`accept`、操作（`send`、`toggle`、`send_sequence`、`set_option`、`unset_option`、`select`）。`when` 可為 `paging`、`has_menu`、`composing`、`always`。
- `editor.bindings`：自訂編輯行為，鍵名與 `key_binder` 相同，操作包括 `confirm`、`commit_comment`、`commit_raw_input`、`commit_script_text`、`commit_composition`、`revert`、`back`、`back_syllable`、`delete_candidate`、`delete`、`cancel`、`noop`。
- `punctuator.full_shape`/`half_shape`：定義全角與半角標點；`use_space` 控制空格頂字；`commit` 可直接上屛，`pair` 交替輸出。

**可用按鍵列表（節選）**
```
BackSpace, Tab, Return, Escape, Delete, Home, Left, Up, Right, Down,
Page_Up, Page_Down, End, Shift_L, Shift_R, Control_L, Control_R, Alt_L,
Alt_R, Super_L, Super_R, Caps_Lock, Num_Lock, Insert, Undo, Redo, Menu,
space, !, ", #, $, %, &, ', (, ), *, +, ',', -, ., /, :, ;, <, =, >, ?, @,
[, \, ], ^, _, `, {, |, }, ~, 以及數字鍵盤 KP_* 系列。
```

**綜合示例**
```yaml
key_binder:
  import_preset: default
  bindings:
    - { accept: semicolon, send: 2, when: has_menu }
    - { accept: apostrophe, send: 3, when: has_menu }
    - { accept: "Control+1", select: .next, when: always }
    - { accept: "Control+2", toggle: full_shape, when: always }
    - { accept: "Control+3", toggle: simplification, when: always }
    - { accept: "Control+4", toggle: extended_charset, when: always }
editor:
  bindings:
    Return: commit_comment
punctuator:
  import_preset: symbols
  half_shape:
    "'": { pair: [ "「", "」" ] }
    "(": [ "〔", "［" ]
    .: { commit: "。" }
recognizer:
  import_preset: default
  patterns:
    email: "^[a-z][-_.0-9a-z]*@.*$"
    url: "^(www[.]|https?:|ftp:|mailto:).*$"
    reverse_lookup: "`[a-z]*;?$"
    pinyin_lookup: "`P[a-z]*;?$"
    jyutping_lookup: "`J[a-z]*;?$"
    pinyin: "(?<!`)P[a-z']*;?$"
    jyutping: "(?<!`)J[a-z']*;?$"
    punct: "/[a-z]*$"
```

## 外觀與選單

- `menu` 通常定義於 `default.yaml`/`default.custom.yaml`。
- `style` 通常定義於 `squirrel.yaml`、`weasel.yaml` 及其 `.custom.yaml`。

**示例**
```yaml
menu:
  alternative_select_labels: [ ①, ②, ③, ④, ⑤, ⑥, ⑦, ⑧, ⑨ ]
  alternative_select_keys: ASDFGHJKL
  page_size: 5
style:
  font_face: HanaMinA, HanaMinB
  font_point: 15
  label_format: '%s'
  horizontal: false
  line_spacing: 1
  inline_preedit: true
```

---

# `Dict.yaml` 詳解

## 開始之前

```yaml
# Rime dict
# encoding: utf-8
# 可在此記錄字典來源與版本
```

## 描述檔

1. `name`：字典內部名稱，需與文件名吻合並供 schema 引用。
2. `version`：版本號；每次發布請更新。

**示例**
```yaml
name: cangjie6.extended
version: "0.1"
```

## 配置

1. `sort`：初始排序方式，`original` 或 `by_weight`。
2. `use_preset_vocabulary`：是否引入八股文（詞頻庫）。
3. `vocabulary`：引入其他詞庫（與 `use_preset_vocabulary` 互斥）。
4. `max_phrase_length`：導入詞條的最長詞長。
5. `min_phrase_weight`：導入詞條的最低權重。
6. `columns`：以 Tab 分隔的列順序，常見 `text`/`code`/`weight`/`stem`。
7. `import_tables`：載入其它字典文件。
8. `encoder`：形碼造詞規則。
   - `exclude_patterns`
   - `rules`：`length_equal`/`length_in_range` + `formula`（大寫代表字序，小寫代表對應字碼位置）。
   - `tail_anchor`

**示例**
```yaml
sort: by_weight
use_preset_vocabulary: false
import_tables:
  - cangjie6
columns:
  - text
  - weight
encoder:
  exclude_patterns:
    - '^z.*$'
  rules:
    - length_equal: 2
      formula: "AaAzBaBbBz"
    - length_equal: 3
      formula: "AaAzBaYzZz"
    - length_in_range: [4, 5]
      formula: "AaBzCaYzZz"
  tail_anchor: "'"
```

## 碼表

- 依 `columns` 定義的順序，以 Tab 分隔即可。

**示例**
```yaml
columns:
  - text
  - code
  - weight
  - stem
```

**資料例子**
```
個	owjr	246268	ow'jr
看	hqbu	245668
中	l	243881
呢	rsp	242970
來	doo	235101
嗎	rsqf	221092
爲	bhnf	211340
會	owfa	209844
她	vpd	204725
與	xyc	203975
給	vfor	193007
等	hgdi	183340
這	yymr	181787
用	bq	168934	b'q
```

---

> 雪齋 · 2013‑11‑09
