# BlockBenchで作成したオブジェクトをブロック扱いで追加する

## フロー

### ファイルの書き出し

BlockBenchでファイルを開いた状態から始める。

名前空間とブロック名を決める。ここではminopoppy:vhs（ビデオテープ）とする。

大文字は使えないので注意。（暗黙の仕様）

「Project Settings」

「File」→「Export」→「Export Bedrock Geometry」と進み、「vhs.geo.json」というファイル名で作業場所に保存。
「File」→「Export」→「Export Texture」と進み、「vhs.png」というファイルで作業場所に保存。



## Geminiの回答メモ

Blockbenchで作ったモデルを統合版（Bedrock Edition）に実装するのは、パズルを組み立てるような楽しさがありますね！

「原木と同じ設定」や「右クリックで回転」というリソースパック・ビヘイビアパックの両方にまたがる具体的な実装方法を解説します。

---

## 1. Blockbenchでの書き出し

まず、モデルを正しく書き出す必要があります。

* **File > Export > Export Bedrock Geometry**: `.geo.json`として保存。
* **File > Export > Export Texture**: `.png`として保存。
* **注意点**: `Project Settings`で「Identifier」を `namespace:model_name`（例: `my:fancy_block`）に設定しておくと、後の作業がスムーズです。

---

## 2. リソースパックの設定

モデルの外見を定義します。

### ジオメトリとテクスチャの配置

1. `models/blocks/` 内に `.geo.json` を配置。
2. `textures/blocks/` 内に `.png` を配置。

### blocks.json の編集

ブロックの見た目と名前を紐付けます。

```json
{
  "my:fancy_block": {
    "textures": "fancy_block_texture_name"
  }
}

```

---

## 3. ビヘイビアパックの設定（ここが本番！）

「原木と同じ硬さ」「回転」などのロジックをここに書きます。

`entities`ではなく、**`blocks/fancy_block.json`** を作成します。

### 基本構成（破壊速度とドロップ）

「原木と同じ」にするには、`minecraft:destructible_by_mining` の `seconds_to_destroy` を **2.0**（素手の場合。斧だと早くなる）に設定します。

```json
{
  "format_version": "1.20.10",
  "minecraft:block": {
    "description": {
      "identifier": "my:fancy_block",
      "properties": { "my:rotation": [0, 1, 2, 3, 4, 5, 6, 7] } // 1/8回転用
    },
    "components": {
      "minecraft:destructible_by_mining": { "seconds_to_destroy": 2.0 },
      "minecraft:loot": "loot_tables/blocks/fancy_block.json",
      "minecraft:geometry": "geometry.my_model",
      "minecraft:material_instances": { "*": { "texture": "fancy_block_texture_name", "render_method": "alpha_test" } },
      "minecraft:on_interact": {
        "condition": "query.is_sneaking == false",
        "event": "update_rotation"
      }
    },
    "events": {
      "update_rotation": {
        "set_block_property": {
          "my:rotation": "query.block_property('my:rotation') == 7 ? 0 : query.block_property('my:rotation') + 1"
        }
      }
    },
    "permutations": [
      {
        "condition": "query.block_property('my:rotation') == 1",
        "components": { "minecraft:transformation": { "rotation": [0, 45, 0] } }
      }
      // ...これを7まで繰り返す（45度ずつ加算）
    ]
  }
}

```

---

## 4. 1/8回転（45度ずつ）の実装ポイント

上記の `permutations`（置換）を使うのがもっとも確実です。

* **Property**: `my:rotation` という数値を定義。
* **Event**: `on_interact`（右クリック）されたときに、その数値を +1 する。
* **Transformation**: 数値が 1 なら 45度、2 なら 90度... とモデルを回転させる。
* **設置時の向き**: 設置した瞬間のプレイヤーの向きに合わせる場合は、`minecraft:on_placed` イベントで `query.cardinal_facing_2d` を取得する処理が必要になりますが、まずは手動回転から試すのがおすすめです。

---

## 5. 次のステップ

モデルが表示されない、あるいはピンク色のチェック模様（テクスチャエラー）になる場合は、`terrain_texture.json` の設定が漏れている可能性が高いです。

もしよろしければ、**「terrain_texture.json」の書き方や、ドロップアイテムを定義する「Loot Table」の具体的な書き方**についても詳しく解説しましょうか？