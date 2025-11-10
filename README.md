# rime-japanese

## About

This is a layout for typing in Japanese 日本語. Supports words in all 3 scripts (Kanji, Hiragana, Katakana).


## Installing

First ensure you have plum installed. For macOS this would be:

```bash
cd ~/Library/Rime
wget https://git.io/rime-install
```

Then install `gkovacs/rime-japanese` using plum:

```bash
bash rime-install gkovacs/rime-japanese
```

Finally edit `default.custom.yaml` and add `japanese` to the schema list:

```bash
patch:
  schema_list:
    - schema: japanese
```

Now reload RIME and it should appear under your layouts.

## 大西配列（Onishi layout）

If you prefer the [Onishi keyboard layout](https://o24.works/layout/) you can enable the bundled `japanese_onishi` schema:

1. Deploy this repo as usual (via plum or by copying the files into your Rime folder).
2. Edit `default.custom.yaml` so the schema list contains the variant, for example:
   ```yaml
   patch:
     schema_list:
       - schema: japanese
       - schema: japanese_onishi
   ```
3. Deploy (or restart Fcitx5/Rime). The new schema shows up as “日本語（大西配列）”.

The schema adds a transliteration rule in `speller.algebra` that remaps the physical QWERTY keys into the Onishi arrangement (vowels on the left hand, consonants on the home row, comma/period on `R`/`T`, etc.) before the Japanese dictionary lookup runs. If you use a JIS keyboard with extra keys, adjust the `xlit` rule in `japanese_onishi.schema.yaml` so that the source string reflects your hardware scan code order.
