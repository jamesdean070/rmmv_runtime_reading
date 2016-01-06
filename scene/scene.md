# Sceneを理解する
## ３行で
- RPGメーカーで作られたゲームはSceneオブジェクトの更新を繰り返すことで進行する。
- ゲーム中にアクティブになれるSceneはただ１つだけで、SceneManagerがこれを保持している。
- Sceneオブジェクトの責務はフレームごとの処理を実行することと、次に遷移するSceneをSceneManagerに通知すること。

## SceneManger
グローバルに定義されたシングルトンオブジェクト. 
### 責務
- 現在有効なSceneの保持
- Sceneの切り替え
- 毎フレームごとにSceneを更新する
- スクリプトがエラーを起こした時にダイアログを表示する

### 主なメソッド
- ```goto(Scene): void``` Sceneを切り替える。
- ```push(Scene): void``` Sceneを切り替える。gotoと異なり新しいSceneをスタックの上に積むので遷移先のSceneがpopメソッドで直前のSceneに戻ることができる.
- ```pop(Scene): void``` 直前のSceneに遷移する。スタックが空の場合はゲームを終了する。

## Sceneオブジェクト
Sceneはゲームを場面にそって駆動し、次にどのSceneに遷移するかを管理するオブジェクト。

### 主なメソッド

### Sceneオブジェクトのライフサイクル
SceneManagerの実装を読んだところ、Sceneオブジェクトのライフサイクルは下図のようになっている。
コード上に明示的に書かれているわけではないのでステート名は適当なものをつけた。  
しかしSceneオブジェクトはready,busy,activeの３つのプロパティしかもっていないため自身がライフサイクルのうちどの状態にあるのかを知ることができない。
プラグインを作るときには要注意。
![scene_state](images/scene_state.svg)

### Sceneごとの遷移
![scene_transition_diagram](https://raw.githubusercontent.com/ryiwamoto/rmmv_runtime_reading/master/scene/images/scene_transition_diagram.svg)
※ (return prev scene)とある遷移は「直前のSceneに戻る」という処理。
※ game eventによる遷移はユーザーが指定したイベント処理によって発生するのでScene_MapだけでなくScene_Battleから遷移することもできそう？

### 各クラス
Scene関連のクラスの概要とプラグインを作るときにどのクラスを拡張すればよいかの指針を紹介する。
#### Scene_base
Sceneの抽象クラス。全Sceneで共通の処理がまとめてある。
#### いつ拡張する？
- 全てのSceneで毎フレームごとの処理を変更したい
- 全てのScene切替時のフェードイン・フェードアウト処理を変更したい
- ゲームオーバー判定を変更したい

#### Scene_Boot
htmlのエントリポイントから呼び出されるScene。
システム画像の読み込み・データベースの読み込み・フォントの読み込み・重要な音声データの読み込みを待ったあとでScene_Titleに遷移させる。
ただし、戦闘SceneやマップSceneのデバッグのために、オプション付きで起動された場合はいきなりScene_BattleまたはScene_Mapに遷移させる。

#### いつ拡張する？
- ゲーム初期化時に追加の処理させたい場合
- 画像・データベース・フォント・音声データ以外のデータを先読みさせたい場合
- デバッグオプションから飛ぶことができるSceneを増やしたい場合

#### Scene_Title
タイトル画面。
![Scene_Title](./images/Scene_Title.png)
#### いつ拡張する？
- タイトル画面に独自の情報を表示したい場合。
- タイトル画面のメニューを増やしたい場合。


#### Scene_Map
マップ場面のゲーム部分を動かし、他Sceneへの遷移を管理する。
マップ上の具体的な処理（マップ描画・イベント発生判定処理・乗り物処理など）はGame_Mapに移譲している。
![Scene_Map](./images/Scene_Map.png)
#### いつ拡張する？
- 戦闘画面とメニュー画面以外にマップ画面から遷移する画面を増やしたい場合
- 戦闘画面に遷移するときの処理（画面点滅エフェクト・BGMを停止する処理）を変更したい場合
- メニュー画面に遷移するときのエフェクトを変更したい場合

#### Scene_MenuBase
メニュー画面の抽象クラス。背景設定処理とアイテム画面などでつかう「ゲームパーティーからコマンド対象を選ぶ機能」の実装をもつ。
##### いつ拡張する？
- メニュー画面の背景を変更したい

#### Scene_Menu
メニュー画面。
![Scene_Menu](./images/Scene_Menu.png)
##### いつ拡張する？
- メニューに表示される項目を変更したい
- 所持金ウィンドウを非表示にしたい
- ステータスウィンドウを非表示にしたい


#### Scene_ItemBase
Scene_ItemとScene_Skillの親クラス。選択された「アクター」にGame_Itemを適用する実装がある。
読んでいる時にちょっと混乱したのだが、ツクールでは「道具（アイテム）」「スキル」「武器」「防具」をまとめてGame_Itemクラスで表現している。

---
ここから下はほとんどWindowを起動しているだけでプラグインを作る上で見どころがないので省略。
#### Scene_Item
アイテム画面
![Scene_Item](./images/Scene_Item.png)
#### Scene_Skill
スキル画面
![Scene_Skill](./images/Scene_Skill.png)
#### Scene_Equip
装備画面
![Scene_Equip](./images/Scene_Equip.png)
#### Scene_Status
ステータス画面
![Scene_Status](./images/Scene_Status.png)
#### Scene_Options
オプション画面
![Scene_Options](./images/Scene_Options.png)
#### Scene_File
Scene_Save・Scene_Loadの親クラス。セーブ画面を表示して選択する処理が定義されている。
#### Scene_Save
セーブ画面
![Scene_Save](./images/Scene_Save.png)
#### Scene_Load
ロード画面
![Scene_Load](./images/Scene_Load.png)
#### Scene_Gameend
ゲーム終了画面
![Scene_GameEnd](./images/Scene_Gameend.png)
#### Scene_Shop
ショップ画面
![Scene_Shop](./images/Scene_Shop.png)
##### いつ拡張する？
- ショップでの購入・売却処理を変更したい（質問に答えるとおまけで安くする機能とか？）

#### Scene_Name
名前設定画面
![Scene_Name](./images/Scene_Name.png)
#### Scene_Debug
デバッグ画面
![Scene_Debug](./images/Scene_Debug.png)
#### Scene_Battle
戦闘画面。Scene_Battleはウィンドウの初期化とコールバック処理の登録の実行しかしておらず、実際の戦闘処理はBattleManagerに移譲している。
![Scene_Battle](./images/Scene_Battle.png)
#### Scene_Gameover
ゲームオーバー画面
![Scene_Gameover](./images/Scene_Gameover.png)
##### いつ拡張する？
- ゲームオーバー時に表示する情報を変更したい（死因の表示など）
