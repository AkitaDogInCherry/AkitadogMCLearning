# フルブロック（全面同じ画像）を追加する

## 作成するブロック

作成するブロックは以下のような特徴を持つものとする。

- フルブロックで不透明のブロック。
- 土と同じ柔らかさ。
- 破壊すると自身をドロップする。

特徴を持たない一般的なフルブロックである。

命名について、ここでは、以下のようにする。

- リソースパックのフォルダ名は、MyApp_RP
- 作成するブロックのIDは、ma:sample_block1
- テクスチャは、sample_block1_texture.png

## 画像を保存

画像を作成し、

MyApp_RP/textures/blocks/sample_block1_texture.pngに保存する。

## terrain_texture.jsonの作成

terrain_texture.jsonは、ブロック用テクスチャ名簿（＝辞書）。

テクスチャは直接画像ファイルを指定するのではなく、「この名前のテクスチャで、この画像を参照する」というような辞書を作る。

MyApp_RP/textures/terrain_texture.jsonファイルを作成する。

ここでは以下のようにする。

```

{
  "texture_name": "atlas.terrain",
  "resource_pack_name": "MyApp_RP",
  "texture_data": {
    "sample_block1": { "textures": "textures/blocks/sample_block1_texture" }
  }
}

```

「textures/blocks/sample_block1_texture」に拡張子はつけない。

## リソースパック側の作業

ブロックが、フルブロックで不透明である、といった情報をjsonファイルに記録する。

一般的に、ブロックの情報（形状など）はビヘイビアーパックにJSONファイルで記録する。

特殊なブロックなどは1ブロック当たり1つのJSONファイルを作成する（後述の「方法2」）が、「不透明なフルブロック」についてはblocks.jsonというファイルにまとめて書く（「方法1」）決まりになっている。

この挙動は混乱しやすいものであり、将来は後述の「方法2」でも動くようになるらしいが、現状は「方法2」では動かない。

方法2も今後の理解の上では見ておいた方が良いので、ここでは方法1, 2の両方を書いておく。

### 「方法1」blocks.jsonの作成

方法1では、このように書く。

```blocks.json

{
  "format_version": "1.1.0",
  "ma:sample_block1": {
    "textures": "sample_block1",
    "sound": "dig.gravel"
  }
}

```

### 「方法2」blocks/sample_block1.jsonの作成（この方法では動かない）

方法2では、このように書く。

```

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "ma:sample_block1" 
    },
    "components": {
      "minecraft:geometry": "geometry.full_block",
      "minecraft:material_instances": {
        "*": { "texture": "sample_block1", "render_method": "opaque", "sound": "dig.gravel" }
      }
    }
  }
}

```

方法2の方は、

```

"identifier": "ma:sample_block1" 

```

識別子が「ma:sample_block1」であるブロックであることが明記されている。

format_versionが、「方法1」は1.1.0、「方法2」は1.21.0となっているが、これは間違いではない。

1.1.0を使うのはblocks.json限定で、基本的には1.21.0（Minecraftのバージョン）の数値を使用する。

### トラブルシューティング

- terrain_texture.jsonが違う場所（例えばリソースパックのルート）にあると、テクスチャが黒と紫のグリッドになる。
- 「方法2」で記述すると、テクスチャが土に「？」マークのようになる。

## ビヘイビアーパック側の作業

### 基本情報の設定

**blocks/sample_block1.json**というファイルを作成し、以下のように書く。

```

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "ma:sample_block1"
    },
    "components": {
      "minecraft:collision_box": true,
      "minecraft:selection_box": true,

      "minecraft:destructible_by_mining": {
        "seconds_to_destroy": 0.5
      },

      "minecraft:destructible_by_explosion": {
        "explosion_resistance": 0.5
      }
    }
  }
}

```

### 破壊すると自身をドロップする仕組みの作成

**loot_tables/blocks/sample_block1.json**というファイルを作成し、以下のように書く。

先ほどとフォルダが異なるので注意。

```

{
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "item",
          "name": "ma:sample_block1"
        }
      ]
    }
  ]
}

```

さらに、先ほどのファイルの末尾に以下のように追記する。

```

{
  "format_version": "1.21.0",
  "minecraft:block": {
    "description": {
      "identifier": "ma:sample_block1"
    },
    "components": {
      (中略)
      "minecraft:destructible_by_explosion": {
        "explosion_resistance": 0.5
      },
      "minecraft:loot": "loot_tables/blocks/sample_block1.json"
    }
  }
}

```

カンマと、「minecraft:loot」の行を追加した。

これで、このブロックを破壊した時にloot_tables/blocks/sample_block1.jsonの中身を参照することになる。

## 確認

Minecraftを起動してリソースパック・ビヘイビアーパックを入れ、

```

/give @s ma:sample_block1 

```

とブロック入手のコマンドを入力して、指定したテクスチャのブロックが入手できればOK。

破壊するとそのブロック自身が落ちるかどうかも確認。



