# SEKIROのキャラクター置き換えMOD作成のメモ

## 初めに

何があっても自己責任でお願いします。  
SteamでBANされても責任は取れません。  

以下のwikiに色々のってるので初めに呼んでおけば楽です。  
[Soulsborne-Modding-Wiki/wiki/Game-Engine-and-File-Formats-Overview](https://github.com/soulsmods/Soulsborne-Modding-Wiki/wiki/Game-Engine-and-File-Formats-Overview)  

詰まった部分のみ記載します。

## 必要なもの

* SEKIRO  

ゲーム本体。  
Steamで購入する。  

* SEKIRO MOD ENGINE  

MODエンジン。
ファイルの読み込みをフックしたりする。
ダウンロードにはnexusmodsのアカウントが必要。  
https://www.nexusmods.com/sekiro/mods/6

ソースはここ。  
https://github.com/katalash/ModEngine  

ビルドには他にもファイルが必要で面倒なので素直にnexusmodsから取得するのが楽。

* UXM

データアンパックツール。  
必須ではない。  
ダウンロードにはnexusmodsのアカウントが必要。  
https://www.nexusmods.com/sekiro/mods/26  

ソースはここ。  
https://github.com/JKAnderson/UXM

* Yabber 

データアンパック/パックツール。  
UXMで展開して出てきたファイルはさらに独自形式でパックされているのでそれをアンパック/パックする。  
こちらは必須。  
ダウンロードにはnexusmodsのアカウントが必要。  
https://www.nexusmods.com/sekiro/mods/42 

ソースはここ。
https://github.com/katalash/Yabber

* FLVER_Editor

モデルファイル(flverファイル)編集ツール。  
これはソースからのビルドが楽なのでソースからのビルド推奨。  
(ソースに修正が必要な個所あり。)  

ソースはここ。  
https://github.com/asasasasasbc/FLVER_Editor

* MTD Editor

マテリアルの設定を編集するツール。  
必須ではなく自分は使用していないので詳細不明。  
ダウンロードにはnexusmodsのアカウントが必要。  
https://www.nexusmods.com/darksouls/mods/1455

* Costume Pack Mod

色んなコスチュームが入ったパック。  
使用するのは中に入っている透明なパーツのみ。  
ダウンロードにはnexusmodsのアカウントが必要。  
https://www.nexusmods.com/sekiro/mods/93

* Blender/Maya

Mayaを持ってないのでBlenderを使いました。

* GIMP/PhotoShop

PhotoShopを持ってないのでGIMPを使用しました。  
GIMPは微妙にDDSフォーマットに対応していないのでPhotoShopのほうがいいです。

## MOD導入

MOD Engineをreadme.txtを参考にして導入してください。  
modengine.iniの中身を設定を以下のように変更するのがおすすめです。  

```
; タイトルに戻るをしてゲーム再開すればファイルの更新が反映されるようにする。
cacheFilePaths=0

; デバッグログを表示する。
showDebugLog=1
```

## FLVER_EDITOR モデル読み込み

基本的には以下の動画を参考にしてください。  
[SEKIRO Modding Tutorial - How to Import 3d Models (FLVER Editor 1.42)](https://www.youtube.com/watch?v=KcXyABspyMM)  

一からモデルを作って適応する場合は上記の動画の概要に書いてある`DS3 Skeleton & Invisible parts`を取得して参考にしながらリギングしてください。  
既存モデルに差し替える場合はFLVER_Editorのボーン差し替え機能を使用します。  
FLVER_Editor(MySFformat.exe)と同一のフォルダに`boneConvertion.ini`というファイルが置いてあります。  
このファイルにはインポートするモデルのボーンをSEKIRO用のボーンに対応させるための情報が書いてあります。  
フォーマットは簡単なので中身を見て察してください。  
ボーン構造が全然違う場合はどうしようもないのでモデリングツールでいい感じのボーンを作ってください。  

この状態でFLVER_EditorのImportModelすればいい感じにメッシュの設定がされると行きたいところですがモデルによってはうまくいかない場合があります。  
特に指先や髪の先がボーンの設定が行われない可能性があります。  

なのでFLVER_Editorのソースの以下の部分を修正します。  
https://github.com/asasasasasbc/FLVER_Editor/blob/80002645a426db24200e2af6a827ab8906644bbe/MySFformat/ProgramFbxImport.cs#L242  

```patch
-                                     if (boneParentList.ContainsValue(boneName))
+                                     if (boneParentList.ContainsKey(boneName))
```

https://github.com/asasasasasbc/FLVER_Editor/blob/80002645a426db24200e2af6a827ab8906644bbe/MySFformat/ProgramFbxImport.cs#L237  

```patch
-                             for (int bp = 0; bp < 5; bp++)
+                             for (int bp = 0; bp < 20; bp++)
```

修正内容についてはソースの周辺個所を見れば大体何をしているかわかります。  

これでモデルの読み込みは完了です。


## サイズの微調整

FLVER ViewerでBキーを押すとボーンが表示されます。  
メッシュとボーンがいい感じの位置に来るようにスケールを設定してModifyをクリックします。  
undoできないので注意してください。  
どうしても会わない場合はXZのみ、Yのみのスケールをかけて多少モデルを変形させましょう。  
どうしようもない場合は再モデリングです。  

## マテリアルの調整  

ExportJsonして中身を書き換えてLoadJsonします。  
ポイントは`MTD`に`N:\\NTC\\data\\Material\\mtd\\character\\c9990_dummy.mtd`を設定すること、  
`Textures`に適当に必要なテクスチャを設定することです。  
`MTD`はマテリアルの設定でほかにも`M[ARSN]_e.mtd`が設定できます。  
マテリアルの設定を変更した場合は必要なテクスチャタイプも変わるので注意してください。  
どのマテリアルにどのテクスチャが必要かはMTD Editorで確認できます。  
`c9990_dummy.mtd`の場合は以下の用に設定します。  

```json
{
		"Name":	"hoge",
		"MTD":	"N:\\NTC\\data\\Material\\mtd\\character\\c9990_dummy.mtd",
		"Flags":	1342,
		"Textures":	[
			{
				"Type":	"Character_AMSN_snp_Texture2D_2_AlbedoMap_0",
				"Path":	"body.tif",
				"ScaleX":	1,
				"ScaleY":	1,
				"Unk10":	1,
				"Unk11":	true,
				"Unk14":	0,
				"Unk18":	0,
				"Unk1C":	0
			},
			{
				"Type":	"SAT_Equip_snp_Texture2D_1_BlendMask",
				"Path":	"N:\\FDP\\data\\Other\\SysTex\\SYSTEX_DummyBurn_m.tif",
				"ScaleX":	1,
				"ScaleY":	1,
				"Unk10":	0,
				"Unk11":	false,
				"Unk14":	0,
				"Unk18":	0,
				"Unk1C":	0
			},
			{
				"Type":	"SAT_Equip_snp_Texture2D_0_EmissiveMap_0",
				"Path":	"N:\\FDP\\data\\Other\\SysTex\\SYSTEX_DummyBurn_em.tif",
				"ScaleX":	1,
				"ScaleY":	1,
				"Unk10":	0,
				"Unk11":	false,
				"Unk14":	0,
				"Unk18":	0,
				"Unk1C":	0
			},
			{
				"Type":	"SAT_Equip_snp_Texture2D_2_DamageNormal",
				"Path":	"N:\\FDP\\data\\Other\\SysTex\\SYSTEX_DummyDamagedNormal.tif",
				"ScaleX":	1,
				"ScaleY":	1,
				"Unk10":	0,
				"Unk11":	false,
				"Unk14":	0,
				"Unk18":	0,
				"Unk1C":	0
			},
			{
				"Type": "Character_AMSN_snp_Texture2D_7_NormalMap_4",
				"Path": "body_n.tif",
				"ScaleX": 1,
				"ScaleY": 1,
				"Unk10": 1,
				"Unk11": true,
				"Unk14": 0,
				"Unk18": 0,
				"Unk1C": 0
			},
			{
				"Type":	"g_SpecularTexture",
				"Path":	"body_r.tif",
				"ScaleX":	1,
				"ScaleY":	1,
				"Unk10":	1,
				"Unk11":	true,
				"Unk14":	0,
				"Unk18":	0,
				"Unk1C":	0
			}
		],
		"GXBytes":	[...省略],
		"Unk18":	0
	}
```

上記の例ではアルベド、ノーマル、スペキュラのテクスチャを設定しています。  
テクスチャファイル自体は`.tpf`ファイルをアンパックしてその中にファイルを格納し、再度パックすることで使用できるようになります。  
ファイルの中身を追加してパックする場合は`_yabber-tpf.xml`に対応するエントリを追加してからパックする必要があります。 

マテリアルの設定ができれば作業完了です。  

### 追記

とりあえずモデルの色を出したい場合は`M[ARSN]_e.mtd`にしてdiffuseだけ設定するとよい。

```json
{
	"Name":	"negi.fbx_Material.003",
	"MTD":	"M[ARSN]_e.mtd",
	"Flags":	1342,
	"Textures":	[
		{
			"Type":	"g_DiffuseTexture",
			"Path":	"tx_negi.tif",
			"ScaleX":	1,
			"ScaleY":	1,
			"Unk10":	1,
			"Unk11":	true,
			"Unk14":	0,
			"Unk18":	0,
			"Unk1C":	0
		}
	],
	"GXBytes":	[
		71,
		88,
		77,
		68,
		242,
		0,
		0,
		0,
		28,
		0,
		0,
		0,
		1,
		0,
		0,
		0,
		31,
		1,
		0,
		0,
		1,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		71,
		88,
		48,
		48,
		100,
		0,
		0,
		0,
		44,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		71,
		88,
		48,
		52,
		100,
		0,
		0,
		0,
		64,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		160,
		64,
		0,
		0,
		128,
		63,
		0,
		0,
		0,
		63,
		0,
		0,
		0,
		0,
		0,
		0,
		160,
		64,
		0,
		0,
		0,
		63,
		0,
		0,
		128,
		63,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		71,
		88,
		56,
		48,
		100,
		0,
		0,
		0,
		28,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		71,
		88,
		56,
		49,
		100,
		0,
		0,
		0,
		56,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		128,
		63,
		0,
		0,
		128,
		63,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		0,
		255,
		255,
		255,
		127,
		100,
		0,
		0,
		0,
		12,
		0,
		0,
		0
	],
	"Unk18":	0
}
```

## ファイル配置

`\mods\parts`に作成したファイルとCostume Packから取得した空パーツを配置します。  
この状態でゲームプレイを開始すればMODが適用されているはずです。  

## トラブルシューティング

### メッシュが表示されない

マテリアルが間違ってると表示すらされないので`M[ARSN]_e.mtd`などに変更して表示されるかどうかを確認する。


## 最後に

パック/アンパックの作業が結構発生するのでバッチファイルを作成して自動でpartsフォルダに配置されるようにしておいた方が効率がいいです。  
今回記載した手法はとりあえず差し替えるだけなのでちゃんとしたかったらもっとソースを読んだりする必要がありますがそこまで必要なかったのでこれで終わりとします。  

