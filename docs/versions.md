# Minecraftのバージョン表記

はい。**Minecraft Bedrock 26.45 を前提に新しいアドオンを書くなら、現時点では「1.26.40」を基準にする**と考えるのが一番分かりやすいです。

少し奇妙ですが、

> ゲーム本体の表示：**26.45**
> Creator側の基準バージョン：**1.26.40**

です。26.45は26.40系に対するHotfixで、Creator向けの新しいJSON仕様が「1.26.45」として追加されたわけではありません。実際、Mojang公式の最新 `bedrock-samples` でも `min_engine_version: [1, 26, 40]`、エンティティ等の `format_version: "1.26.40"` が使われています。([Minecraft Feedback][1])

## まず、この4種類を区別すると分かりやすいです

| 書く場所                               |   26.45向けの基本値 | 何の番号か                |
| ---------------------------------- | ------------: | -------------------- |
| `manifest.json` の `format_version` |           `2` | manifest自体の書式        |
| `min_engine_version`               | `[1, 26, 40]` | 必要なMinecraftエンジン     |
| ブロック等の `format_version`            |   `"1.26.40"` | そのJSONの仕様            |
| `header.version`                   |   `[1, 0, 0]` | **あなたのアドオン自身のバージョン** |

この4つは別物です。

### Behavior Packのmanifestはこうする

今から普通のBehavior Packを作るなら、まずこれで問題ありません。

```json
{
  "format_version": 2,
  "header": {
    "name": "My Add-On",
    "description": "My first add-on",
    "uuid": "ここにUUID",
    "version": [1, 0, 0],
    "min_engine_version": [1, 26, 40]
  },
  "modules": [
    {
      "type": "data",
      "uuid": "ここに別のUUID",
      "version": [1, 0, 0]
    }
  ]
}
```

公式ドキュメントでも、通常のBehavior Pack / Resource Packのmanifestは現在も

```json
"format_version": 2
```

が推奨されています。`format_version: 3` という新仕様もありますが、現在の公式資料ではまだPreview扱いなので、普通のアドオンでは**2を使っておくのが安全**です。([Microsoft Learn][2])

そしてMojang公式の現在のサンプルそのものが、

```json
"format_version": 2,
...
"min_engine_version": [1, 26, 40]
```

になっています。([GitHub][3])

## `version: [1,0,0]` はMinecraftのバージョンではない

ここは特に間違えやすいです。

```json
"version": [1, 0, 0]
```

は、

> My Add-On version 1.0.0

という意味です。

Minecraft 26.45だから、

```json
"version": [26, 45, 0]
```

にするわけではありません。

たとえば自分のアドオンを修正したら、

```text
最初
[1, 0, 0]

小修正
[1, 0, 1]

機能追加
[1, 1, 0]

大幅更新
[2, 0, 0]
```

など、自分で管理します。

`header.version` と `modules[].version` は基本的には同じ値にしておけばよいです。

---

## ブロックJSONなら `"1.26.40"`

たとえば独自ブロックなら、

```json
{
  "format_version": "1.26.40",
  "minecraft:block": {
    "description": {
      "identifier": "mymod:my_block"
    },
    "components": {
      ...
    }
  }
}
```

です。

Mojangの現在の公式サンプルもブロックについて `"1.26.40"` を使用しています。([GitHub][4])

エンティティも同様に、

```json
{
  "format_version": "1.26.40",
  "minecraft:entity": {
    ...
  }
}
```

とできます。公式サンプルの `egg.json` も現在 `"1.26.40"` です。([GitHub][5])

ただし注意点が一つあります。`1.26.40` ではエンティティJSONの検証が以前より厳しくなっています。古いネット記事をそのままコピーして `"1.26.40"` にすると、不正な古い書式がエラーになるケースがあります。([Microsoft Learn][6])

---

## ただし「すべてのJSONを1.26.40」にするわけではない

ここがMinecraft Add-Onのやや面倒なところです。

公式の現在の指針では、おおむね次のようになっています。([Microsoft Learn][7])

| ファイル                           | 基本的な考え方                     |
| ------------------------------ | --------------------------- |
| `manifest.json`                | `2`                         |
| `blocks/*.json`                | `"1.26.40"`                 |
| `items/*.json`                 | `"1.26.40"`                 |
| `entities/*.json`              | 新規作成なら `"1.26.40"` でよい      |
| `recipes/*.json`               | 新規なら現行形式を使う                 |
| `spawn_rules/*.json`           | 現行形式を使う                     |
| `animations/*.json`            | **`"1.10.0"` が一般的**         |
| `animation_controllers/*.json` | **`"1.10.0"`**              |
| `render_controllers/*.json`    | **`"1.10.0"`**              |
| `models/*.geo.json`            | **`"1.12.0"`**              |
| `loot_tables/*.json`           | そもそも共通の `format_version` なし |
| `trading/*.json`               | 同様に独自形式                     |
| `textures/item_texture.json` 等 | 通常この意味の `format_version` なし |

つまり、

> **「最新Minecraftだから全部 `1.26.40`」ではない**

ということです。

`format_version` は「ゲームの最低バージョン」というより、

> **このファイルを、どの世代のJSON仕様として解釈してください**

という指定に近いものです。

---

## Script APIを使う場合は、さらに別の番号が出てくる

たとえばJavaScriptを使うBehavior Packなら、manifestにはscript moduleを追加します。

26.45系でStable APIを使うなら、現在の安定版は **`@minecraft/server 2.9.0`** です。1.26.40 Creator Updateで2.9.0がStableとして搭載されました。([Microsoft Learn][6])

たとえば、

```json
{
  "format_version": 2,
  "header": {
    "name": "My Script Add-On",
    "description": "Test",
    "uuid": "PACK-UUID",
    "version": [1, 0, 0],
    "min_engine_version": [1, 26, 40]
  },
  "modules": [
    {
      "type": "data",
      "uuid": "DATA-UUID",
      "version": [1, 0, 0]
    },
    {
      "type": "script",
      "language": "javascript",
      "uuid": "SCRIPT-UUID",
      "version": [1, 0, 0],
      "entry": "scripts/main.js"
    }
  ],
  "dependencies": [
    {
      "module_name": "@minecraft/server",
      "version": "2.9.0"
    }
  ]
}
```

となります。

この

```json
"version": "2.9.0"
```

はさらに別で、

> **Minecraft Script APIのバージョン**

です。本体の26.45とも、JSONの1.26.40とも関係ありません。Script APIは独立したSemVerで管理されています。([Microsoft Learn][8])

---

## したがって、26.45で開発するならこう覚える

いま作り始めるなら、まずこのセットを基準にしてよいです。

```text
Minecraft本体
    26.45

manifest.json
    "format_version": 2

manifest.json
    "min_engine_version": [1, 26, 40]

自分のパック
    "version": [1, 0, 0]

ブロック・アイテム・新しいエンティティ等
    "format_version": "1.26.40"

Script APIを使う場合
    "@minecraft/server": "2.9.0"

animation
    "format_version": "1.10.0"

geometry
    "format_version": "1.12.0"
```

**特に「26.45なのになぜ `1.26.40`？」という点は正常です。** 26.45は8月20日のHotfixで、Creator側で新しい1.26.45形式が作られたわけではありません。現行Mojang公式サンプルも1.26.40基準のままです。([Minecraft Feedback][1])

そして、今後アドオン作成について私に質問する際は、**「Bedrock 26.45／Creator format 1.26.40／Script API 2.9.0 Stable」**を基本前提にすると、古い1.21系の記事との混同をかなり防げます。

[1]: https://feedback.minecraft.net/hc/en-us/articles/48149564061965-Minecraft-Bedrock-Edition-26-44-45-Hotfix-Changelog?utm_source=chatgpt.com "Minecraft: Bedrock Edition 26.44/45 Hotfix Changelog – Minecraft Feedback"
[2]: https://learn.microsoft.com/en-us/minecraft/creator/reference/content/addonsreference/packmanifest?view=minecraft-bedrock-stable "Add-Ons Reference: manifest.json | Microsoft Learn"
[3]: https://github.com/Mojang/bedrock-samples/blob/main/behavior_pack/manifest.json?utm_source=chatgpt.com "bedrock-samples/behavior_pack/manifest.json at main · Mojang/bedrock-samples · GitHub"
[4]: https://github.com/Mojang/bedrock-samples/blob/main/documentation/Blocks.html?utm_source=chatgpt.com "bedrock-samples/documentation/Blocks.html at main · Mojang/bedrock-samples · GitHub"
[5]: https://github.com/Mojang/bedrock-samples/blob/main/behavior_pack/entities/egg.json?utm_source=chatgpt.com "bedrock-samples/behavior_pack/entities/egg.json at main · Mojang/bedrock-samples · GitHub"
[6]: https://learn.microsoft.com/en-us/minecraft/creator/documents/update1.26.40?view=minecraft-bedrock-stable "1.26.40 Update Notes | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/minecraft/creator/documents/practices/latestplatformversion?view=minecraft-bedrock-stable "Latest Platform Version Guidance | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/minecraft/creator/documents/scripting/versioning?view=minecraft-bedrock-stable&utm_source=chatgpt.com "Script Module Versioning | Microsoft Learn"
