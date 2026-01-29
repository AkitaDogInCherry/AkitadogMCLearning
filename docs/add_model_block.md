# BlockBenchで作成したオブジェクトをブロック扱いで追加する

## どんなオブジェクトの仕様？

薄い長方形のビデオテープ。

** プレイヤーが置いた時に向いている向きに応じた向きになる。 **

## フロー

### ファイルの書き出し

BlockBenchでファイルを開いた状態から始める。

名前空間とブロック名を決める。ここではmumupoppy:vhs（ビデオテープ）とする。

大文字は使えないので注意。（暗黙の仕様）

「Project Settings」

「File」→「Export」→「Export Bedrock Geometry」と進み、「vhs.geo.json」というファイル名で作業場所に保存。
「File」→「Export」→「Export Texture」と進み、「vhs.png」というファイルで作業場所に保存。


vhs.geo.jsonを以下の位置にコピー：

(リソースパック)/models/blocks/vhs.geo.json

vhs.pngを以下の位置にコピー：

(リソースパック)/textures/blocks/vhs.png

### 普通のブロックと同じ部分

(リソースパック)/blocks.json

```blocks.json

{
  "format_version": [1, 1, 0],
  (中略)
  "mumupoppy:vhs": {
    "textures": "vhs",
    "sound": "wood"
  }
}

```

(リソースパック)/textures/terrain_texture.json

``` terrain_texture.json

{
  "resource_pack_name": "MumuPoppy",
  "texture_name": "atlas.terrain",
  "texture_data": {
    "vhs": {
      "textures": "textures/blocks/vhs"
    }
  }
}

```

### 難しい部分

#### とりあえず、サンプルコード

(ビヘイビアーパック)/blocks/vhs.jsonは以下のようにする。

```vhs.json

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "mumupoppy:vhs",
      "menu_category": {
        "category": "construction",
        "group": "itemGroup.name.items"
      },
      "traits": {
        "minecraft:placement_direction": {
          "enabled_states": ["minecraft:cardinal_direction"]
        }
      }
    },
    "components": {
      "minecraft:geometry": "geometry.vhs",
      "minecraft:material_instances": {
        "*": {
          "texture": "vhs",
          "render_method": "alpha_test"
        }
      },
      "minecraft:destructible_by_mining": { "seconds_to_destroy": 2.0 },
      "minecraft:selection_box": { "origin": [-2, 0, -4], "size": [4, 2, 8] },
      "minecraft:collision_box": { "origin": [-2, 0, -4], "size": [4, 2, 8] },
      "minecraft:loot": "loot_tables/blocks/vhs.json"
    },
    "permutations": [
      {
        "condition": "query.block_state('minecraft:cardinal_direction') == 'north'",
        "components": { "minecraft:transformation": { "rotation": [0, 270, 0] } }
      },
      {
        "condition": "query.block_state('minecraft:cardinal_direction') == 'east'",
        "components": { "minecraft:transformation": { "rotation": [0, 0, 0] } }
      },
      {
        "condition": "query.block_state('minecraft:cardinal_direction') == 'south'",
        "components": { "minecraft:transformation": { "rotation": [0, 90, 0] } }
      },
      {
        "condition": "query.block_state('minecraft:cardinal_direction') == 'west'",
        "components": { "minecraft:transformation": { "rotation": [0, 180, 0] } }
      }
    ]
  }
}

```

比較として、ただのブロックはこんな感じ。

```sample_block1.json

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "akitadog:sample_block1"
    },
    "components": {
      "minecraft:collision_box": true,
      "minecraft:selection_box": true,

      "minecraft:destructible_by_mining": {
        "seconds_to_destroy": 0.5
      },

      "minecraft:destructible_by_explosion": {
        "explosion_resistance": 0.5
      },
      "minecraft:loot": "loot_tables/blocks/sample_block1.json"
    }
  }
}

```

#### 比較しながら詳しく見ていく

** プレイヤーが置いた時に向いている向きに応じた向きになる。 **

これは、「trait」と「permutations」の記載内容によって実現している。
