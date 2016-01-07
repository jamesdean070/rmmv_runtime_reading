# Sceneクラスを理解する
## ３行で
- ツクールで作られたゲームは、Sceneオブジェクトがゲームの状態を毎フレームごとに更新することで進行する。
- Sceneオブジェクトは現在の場面に沿った処理を行い、次にどのSceneに遷移するかを知っている。
- ゲーム中にアクティブになれるSceneは同時にただ１つだけで、SceneManagerが切り替え処理などを管理する。

## SceneManger
グローバル名前空間に定義されたシングルトンオブジェクト.
### 責務
- 現在有効なSceneの保持
- Sceneの切り替え
- 毎フレームごとにSceneに対して更新メッセージを送る
- スクリプトがエラーを起こした時にダイアログを表示する

### 主なメソッド
- ```goto(Scene): void``` Sceneを切り替える。
- ```push(Scene): void``` Sceneを切り替える。gotoと異なり新しいSceneをスタックに積むので遷移先のSceneがpopメソッドで直前のSceneに戻ることができる.
- ```pop(Scene): void``` 直前のSceneに遷移する。
- ```snap(): Bitmap``` 現在のSceneのスクリーンショットを返す
- ```snapForBackground(): Bitmap``` snap()で取得したスクリーンショットにブラーエフェクトをかけたものを返す

## Sceneオブジェクト
ゲームを場面にそって駆動し、次にどのSceneに遷移するかを管理するオブジェクト。

### Sceneオブジェクトのライフサイクル
SceneManagerの実装を読んだところ、Sceneオブジェクトのライフサイクルは下図のようになっているらしい。
コード上に明示的に書かれているわけではないのでステート名は適当なものをつけた。
しかしSceneオブジェクトはready,busy,activeの３つのプロパティしかもっていないため自身がライフサイクルのうちどの状態にあるのかを知ることができない。
プラグインを作るときには要注意。

![scene_state](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/scene_state.svg)

- ```initialized``` 初期化直後の状態。
- ```ready``` 開始に必要なリソースの読み込みなどが完了した状態。
- ```start_transition``` Sceneを切り替えるアニメーションを行っている状態
- ```running``` Sceneを実行している状態
- ```stop_transition``` Sceneを切り替えるアニメーションを行っている状態
- ```stopped``` 新しいSceneに画面が置き換わり破棄される準備が整った状態
- ```terminated``` Sceneの破棄に必要な処理（ゲームウィンドウオブジェクトの破棄など）が終わった状態。

### Sceneどうしの遷移図
Scene同士のつながりをイメージしやすくするため、ざっくりとした遷移図を作った。ユーザー定義イベントで任意のSceneに遷移することができるため、全ての遷移を書ききれているわけではない。
![scene_transition_diagram](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/scene_transition_diagram.svg)

- ※ (prev)とある遷移は「直前のSceneに戻る」という処理。
- ※ "game event"はユーザーが定義したイベントによって発生する遷移。

### Sceneの各クラスと拡張指針
Scene関連のクラスの概要とプラグインを作るときにどのクラスを拡張すればよいかの指針を紹介する。
#### Scene_Base
Sceneの抽象クラス。全Sceneで共通の処理がまとめてある。
#### 拡張できる機能
- 全てのSceneで毎フレームごとの処理
- 全てのScene切替時のフェードイン・フェードアウト処理
- ゲームオーバー判定

#### Scene_Boot
htmlのエントリポイントから呼び出されるScene。
システム画像の読み込み・データベースの読み込み・フォントの読み込み・重要な音声データの読み込みを待ったあとでScene_Titleに遷移させる。
ただしオプション付きで起動された場合は戦闘やマップのデバッグのために直接Scene_BattleまたはScene_Mapに遷移させる。

#### 拡張できる機能
- ゲーム初期化時の追加処理
- 画像・データベース・フォント・音声データ以外のデータを先読みさせる機能
- デバッグオプションから遷移することができるSceneを増やす

#### Scene_Title
タイトル画面。
![Scene_Title](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Title.png)
#### 拡張できる機能
- タイトル画面に独自の情報を表示させる
- タイトル画面のメニュー項目


#### Scene_Map
マップ画面。
マップ上の具体的な処理（マップ描画・イベント発生判定処理・乗り物処理など）はGame_Mapに移譲している。
![Scene_Map](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Map.png)
#### 拡張できる機能
- 戦闘画面に遷移するときのエフェクト（画面点滅・BGMを停止する）
- メニュー画面に遷移するときのエフェクト

#### Scene_MenuBase
メニュー画面の抽象クラス。背景設定と「パーティーからアクターを選ぶ機能」の実装をもつ。
##### 拡張できる機能
- メニュー画面の背景

#### Scene_Menu
メニュー画面。
![Scene_Menu](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Menu.png)
##### 拡張できる機能
- メニューに表示される項目
- 所持金ウィンドウの表示・非表示
- ステータスウィンドウの表示・非表示


#### Scene_ItemBase
Scene_ItemとScene_Skillの親クラス。選択された「アクター」にGame_Itemを適用する実装がある。
読んでいる時にちょっと混乱したのだが、ツクールでは「道具（アイテム）」「スキル」「武器」「防具」をまとめてGame_Itemクラス１つで表現している。

---
ここから下で紹介するクラスは機能が薄くプラグインを作る上で見どころがないので簡単に紹介する。
#### Scene_Item
アイテム画面
![Scene_Item](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Item.png)
#### Scene_Skill
スキル画面
![Scene_Skill](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Skill.png)
#### Scene_Equip
装備画面
![Scene_Equip](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Equip.png)
#### Scene_Status
ステータス画面
![Scene_Status](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Status.png)
#### Scene_Options
オプション画面
![Scene_Options](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Options.png)
#### Scene_File
Scene_Save・Scene_Loadの親クラス。セーブ画面を表示して選択する処理が定義されている。
#### Scene_Save
セーブ画面
![Scene_Save](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Save.png)
#### Scene_Load
ロード画面
![Scene_Load](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Load.png)
#### Scene_Gameend
ゲーム終了画面
![Scene_GameEnd](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Gameend.png)
#### Scene_Shop
ショップ画面
![Scene_Shop](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Shop.png)
##### 拡張できる機能
- ショップでの購入・売却処理（質問に答えるとおまけで安くする機能とか？）

#### Scene_Name
名前設定画面
![Scene_Name](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Name.png)
#### Scene_Debug
デバッグ画面
![Scene_Debug](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Debug.png)
#### Scene_Battle
戦闘画面。実際の戦闘処理はBattleManagerに移譲している。
![Scene_Battle](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Battle.png)
#### Scene_Gameover
ゲームオーバー画面
![Scene_Gameover](http://ryiwamoto.github.io/rmmv_runtime_reading/scene/images/Scene_Gameover.png)
##### 拡張できる機能
- ゲームオーバー時に表示する情報（死因の表示など）
