# フルブロック（面により画像が異なる）を追加する

## 置く方向によって面の向きが変わらないタイプのブロック

置く方向によって面の向きが変わらないタイプのブロックの場合。

以下の仕様で作成する。

- ブロックのIDはsample_block2とする。
- 上下面はsample_block2_top_texture、側面はsample_block2_side_textureとする。

```terrain_texture.json

{
  "texture_name": "atlas.terrain",
  "texture_data": {
    "sample_block1": { "textures": "textures/blocks/sample_block1_texture" },
    "sample_block2_top": { "textures": "textures/blocks/sample_block2_top_texture" },
    "sample_block2_side": { "textures": "textures/blocks/sample_block2_side_texture" }
  }
}

```

面ごとのテクスチャを以下のように指定することができる。

```blocks.json

{
  "format_version": "1.1.0",
  "ma:sample_block1": {
    "textures": "sample_block1",
    "sound": "dig.gravel"
  },
  "ma:sample_block2": {
    "textures": {
      "up": "sample_block2_top",
      "down": "sample_block2_top",
      "north": "sample_block2_side",
      "south": "sample_block2_side",
      "east": "sample_block2_side",
      "west": "sample_block2_side"
    },
    "sound": "dig.wood"
  }
}

```

前節の方法2の方法で記載するならこのようになる。（現在のバージョンではこの記載では動かない）

```sample_block2.json

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "ma:sample_block2" 
    },
    "components": {
      "minecraft:geometry": "geometry.full_block",
      "minecraft:material_instances": {
        "*": { 
          "textures": {
                "up": "sample_block2_top",
                "down": "sample_block2_top",
                "north": "sample_block2_side",
                "south": "sample_block2_side",
                "east": "sample_block2_side",
                "west": "sample_block2_side"
              },
          "render_method": "opaque" 
        }
      }
    }
  }
}

```

