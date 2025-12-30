# 開発中フォルダでの開発準備

## 一番最初の準備

### com.mojangフォルダへのショートカットを作る

以下のフォルダがMinecraftの「開発用フォルダ」にあたる。

```

C:\Users\(ユーザ名)\AppData\Roaming\Minecraft Bedrock\Users\Shared\games\com.mojang

```

少し前の情報（公式Wikiや生成AI含む）だと

```

%localappdata%\Packages\Microsoft.MinecraftUWP_8wekyb3d8bbwe\LocalState\games\com.mojang

```

と書かれているが、アップデートで変更になった。

### com.mojangフォルダの中身

フォルダ内は以下のようになっている。

```

├─behavior_packs
├─development_behavior_packs
├─development_resource_packs
├─development_skin_packs
└─resource_packs

```

アドオンを入れるには.mcaddonファイルをダブルクリックする場合が多いが、.mcaddonでインポートしたアドオンなどは

- behaviour_packs
- resource_packs

フォルダに振り分けられて入る。（と言うか、.mcaddonはリソースパック・ビヘイバーパックをまとめてzip化したもので、単純にそれを解凍して振り分けている）

さらに、

- development_behavior_packs
- development_resource_packs

は

- behavior_packs
- resource_packs

の「開発用」フォルダである。

だいたい同じ挙動であるが、緩いホットリロードなどに対応している。

## 空のアドオンを作成する

### 方針

- ファイルはGit管理したいが、com.mojangフォルダ内にリポジトリを置くべきではない（関係ないファイルも入る）
- あるフォルダを見かけ上別のフォルダと認識させる、「ジャンクション」という仕組みを使う

### 開発用フォルダの作成

#### フォルダ構成

アドオンを開発する際のフォルダ名を決め、適当な場所（com.mojang内ではなく、マイドキュメントなど）にフォルダを作成する。
ここではフォルダ名はMyAppとする。

以下のようにフォルダを作る。

```

MyApp
┝ packs
│　┝ MyApp_RP
│　└ MyApp_BP
┝ tools

```

MyApp_RPがリソースパック（Resource Pack）用のフォルダ、MyApp_BPがビヘイバーパック（Behavior Pack）用のフォルダ。

#### uuid.json

UUIDは、Minecraftがアドオンを識別するための、アドオン固有のIDである。

リソース・ビヘイバーごとに2種類のUUIDがある。

- パック自体のUUID
- パック内の**モジュール**のUUID

モジュールについて、詳しくはまだ理解しなくてよい。

基本は1パックあたり1モジュールだが、複数モジュールを入れることもできるようになっている。

スクリプトを使う場合はビヘイバーパックに「data」のモジュールと「script」のモジュールを作成する。

バージョンアップしてもUUIDの値は変えない。

そのようなUUIDを最初にまとめて作成・保存しておくため、以下のような中身のuuid.jsonをtoolsフォルダ内に作成する。

```tools/uuid.json

{
  "rp_uuid": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
  "bp_uuid": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "rp_module_uuid": "dddddddd-dddd-dddd-dddd-dddddddddddd",
  "bp_module_uuid": "cccccccc-cccc-cccc-cccc-cccccccccccc"
}

```

UUIDは[このサイト](https://www.uuidgenerator.net/version4)で作ることができる。

4回実行して、それぞれを上記のuuidを入れ替えるように記述する。

### リソースパック、ビヘイバーパックとして認識させるためのファイルを作る

リソースパックフォルダ、ビヘイバーパック用フォルダは、それぞれ中にmanifest.jsonというファイルが入っていることでリソースパック・ビヘイバーパックとして認識される。

```MyApp_RP/manifest.json

{
  "format_version": 2,
  "header": {
    "name": "(リソースパック名)",
    "description": "(リソースパックの説明)",
    "uuid": "(uuid.jsonのrp_uuidの値)",
    "version": [1, 0, 0],
    "min_engine_version": [1, 21, 0]
  },
  "modules": [
    {
      "type": "resources",
      "uuid": "(uuid.jsonのrp_module_uuidの値)",
      "version": [1, 0, 0]
    }
  ]
}

```

```MyApp_BP/manifest.json

{
  "format_version": 2,
  "header": {
    "name": "(ビヘイバーパック名)",
    "description": "(ビヘイバーパックの説明)",
    "uuid": "(uuid.jsonのbp_uuidの値)",
    "version": [1, 0, 0],
    "min_engine_version": [1, 21, 0]
  },
  "modules": [
    {
      "type": "data",
      "uuid": "(uuid.jsonのbp_module_uuidの値)",
      "version": [1, 0, 0]
    }
  ]
}

```

### com.mojang内から作業フォルダへの**ジャンクション**を作る

ジャンクションとは、あるフォルダを、特定の場所に置かれているかのように見せかけることができる機能である。

現在、実際には

```

(適当な場所)
└ MyApp
  ┝ packs
  │　┝ MyApp_RP ← リソースパックのフォルダ
  │　└ MyApp_BP ← ビヘイビアーパックのフォルダ
  ┝ tools

```

というフォルダ構成だが、

見た目上、

```

com.mojang
┝ development_resource_packs
│ └ MyApp_RP ← リソースパックのフォルダ
└ development_behavior_packs
　└ MyApp_BP ← ビヘイビアーパックのフォルダ

```

という形にしたい。従って、

- com.mojang/development_resource_packsに、MyApp_RPという名前で、MyApp/packs/MyApp_RPへのリンクを作る
- com.mojang/development_behavior_packsに、MyApp_BPという名前で、MyApp/packs/MyApp_BPへのリンクを作る

という対応を行えば良い。

ジャンクションを作るにはmklinkというコマンドを使えばよい。AppDataというシステム的なフォルダの中なので、管理者権限で起動したコマンドプロンプトでmklinkを行う必要がある。

プログラム一覧から、コマンドプロンプトを「管理者として実行」で起動。

```

mklink /J "(com.mojang)/development_resource_packs/MyApp_RP" "(適当な場所)/MyApp/packs/MyApp_RP" 
mklink /J "(com.mojang)/development_behavior_packs/MyApp_BP" "(適当な場所)/MyApp/packs/MyApp_BP" 

```

- development_resource_packs/MyApp_RPは存在しない状態で実行する。
- packsは複数形なので入力ミスに注意。

```

(com.mojang)\development_behavior_packs\MyApp_BP <<===>> (MyApp)\packs\AkitadogLearning_BP のジャンクションが作成されました

```

と出る。

これでショートカットのようなものが出来ればOK。

## 空のアドオンをインポートする。

Minecraftを起動する。

ワールド新規作成画面の「リソースパック」「ビヘイビアーパック」の中に作った名前のアドオンがあればOK。

