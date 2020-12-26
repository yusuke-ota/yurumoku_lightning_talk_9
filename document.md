# XR Interaction Toolkit 1.0.0-pre.1をAction-basedで使いたい

## 注

* 日本語の資料があまりなかったので書きました。品質は期待しないでください。
* タイトルの通り、XR Interaction Toolkitバージョン1.0.0-pre.1での情報です。
* Action-basedで変わるのは、XR Rig(コントローラとヘッドマウントディスプレイの入力)周りです。その他の部分に関しては以前のバージョンの資料で大丈夫です。
* Qiitaはドメインパワーが強すぎるので、こういうニッチな物は載せないようにしています。ご了承ください。
* 質問、わかりにくい記載内容の指摘等あれば、気軽にIssueください。(Pull Requestもウェルカムです)

## 目次

1. そもそも
   1. XR Interaction ToolkitのAction-basedって何？
   2. Input Systemって何？
2. ※ヘッドマウントディスプレイ、コントローラの位置情報、入力関係の機能
   1. ※TrackedPoseDriver
   2. ※XRController
3. ※Interaction関係の機能(TODO)
   1. InteractionManager
   2. ※RayInteractor / Interactable
   3. ※DirectInteractor / Interactable
   4. GrabInteractor / Interactable
   5. SocketInreractor / Interactable
4. UI関係の機能(TODO)
   1. UI Canvas
   2. UI EventSystem
5. ※移動、テレポーテーション関係の機能(TODO)
   1. ※Locomotion System
   2. Teleportation Area
   3. Teleportation Ancker
6. AR関係の機能(TODO)

`※`マークがついている項目はAction-based版がある機能です。

その他の機能はAction-basedとDevice-basedで共通です。
そこそこ情報が出ているので、このドキュメントを読まなくても良いです。

## 1.そもそも

Action-based以前にXR Interaction Toolkitって何？という方もいると思うので、そのあたりの説明を入れます。  
不要な方は 2.※ヘッドマウントディスプレイ、コントローラの位置情報、入力関係の機能 以降に進んでください。

### 1-1. XR Interaction ToolkitのAction-basedって何？

XR Interaction Toolkit 0.10.0-preview.1から、Input Systemと連携させることができるようになりました。  
Input Systemを利用する機能は機能名の後ろに`(Action-based)`がつきます。

Input Systemを使用することで、VRコントローラ以外も使用できるようにしたり、特定のコントローラのみキーマッピングを変えるといったことができるようになります。

なお、バージョン0.10.0-preview.1以前の機能は`(Device-based)`が付きます。  
XR Rig周りが変わるだけなので、それ以外の機能に関しては以前のバージョンの資料でも大丈夫です。

## 1-2. Input Systemって何？

新しい入力管理機能です。

ここではざっくり説明します。  
細かい話は、Unity Leraning Materialsの[新しいInputSystemの使い方](https://learning.unity3d.jp/4080/)を見てください。

今までの入力管理機能Input Managerでは入力を得るとき

```C#
using UnityEngine;

class ReadButton: MonoBehaviour{
    private void Update()
    {
        bool isPushedOkButton = input.GetKeyDown(KeyCode.A);
    }
}
```

と書いていました。
この記法では入力デバイスが増えたときに、増えた分だけ入力の取得と必要に応じて条件分岐を書く必要があります。

Input Systemでは、マッピング用のファイル(Input Actions)を作成することで、入力デバイスとコードを分けて管理することができるようになります。

```C#
using UnityEngine;
using UnityEngine.InputSystem;

// 前もってPlayerInputコンポーネントに作成したInput Actionsを設定しておく
RequireComponent[typeof(PlayerInput)]
class ReadButton: MonoBehaviour
{
    private InputActions _inputActions;

    private void Start()
    {
        // Okという名前のbool型を返すActionを設定した前提
        _inputActions = GetComponent<PlayerInput>().actions["Ok"];
    }

    private void Update()
    {
        bool isPushedOkButton = _inputActions.ReadValue<bool>()
    }
}
```

[参考]【Unity】出来るだけ簡単にNew Input Systemを使いたい - テラシュールブログ
URL: <http://tsubakit1.hateblo.jp/entry/2019/10/14/215312>

上記のコードでは、Input Actionsの中に隠ぺいでき、どのキーをOkボタンに割り当てているかといった情報が出てきません。  
また、1つのInput Actionsファイルに複数のデバイスを設定することができるため、デバイスの増加に対して、原則コードの書き換えが不要になります。

### 1-2-1. はまったところ

**設定でデバイスを追加したら、マウスの入力を受け付けなくなった。**  
※注: ちゃんと**説明を読んでいなかった**私が悪いです。

`Project Settings` / `Input System Package`に、`Supported Devices`という項目があります。

![SupportedDevicesを強調した画像](./Images/SupportedDevices.png)

Supported Devicesの項目ですが、上の写真の英文の通り、

* 空の場合：全てのデバイスの入力を取得
* 何か設定してある場合:設定しているデバイスのみの入力を取得

となっています。

よく読まずに、ここにデバイスを追加し、追加漏れがあった場合、特定のデバイスで上手く動かないといった現象がおきます。

対応デバイスにこだわりがない場合は、Supported Devicesをいじらない方が良いです。

特定のデバイスのみに限定したい場合、特にエディタから動かす時とビルドして動かす時でデバイスが違う場合は、片方の設定が漏れやすいので、それ単体のタスクとして切り分けて行った方が良いでです。(1敗)

---

## 2. ※ヘッドマウントディスプレイ、コントローラの位置情報、入力関係の機能

### 2-1. XR Rig

SteamVR Pluginでいうところの`[SteamVR]`プレハブです。  
以下のような構成になっています。

```txt todo: 画像に差し替え
XRRig(XRRig.cs: カメラのY座標を変更するスクリプト 付)
 └ CameraOffset(スクリプトはついていない。ヘッドマウントディスプレイの高さ反映用のオブジェクト)
    ├ Main Camera(TrackedPoseDriver.cs: ヘッドマウントディスプレイの位置、角度を取得、反映するスクリプト 付)
    ├ LeftHand Controller(RayInteractor, Ray可視化用のスクリプト付)
    └ RightHand Controller(RayInteractor, Ray可視化用のスクリプト付)
```

Main Cameraについている`TrackedPoseDriver`と、[Left / Right]Controllerについている `XRController`はAction-based版とDevice-based版があります。

既存のメインカメラを勝手に削除して、XR Rigを生成するというゲームオブジェクトの作り方をします。  
もとからあるメインカメラを消されて困る場合は、退避させるなどの対策をしてください。

#### 内容

XR Rig自体はAction-based、Device-basedともに共通です。  
ここの設定はほとんどいじることがないので、比較的いじることがあるTrackingOriginModeの項目についてだけ説明します。

![トラッキングの形態](Images/TrackingOriginMode.png)

XR Rig.csは`Camera Floor Offcet Object`のLocalPositionを

* Floorモードの時は、(0, 0, 0)
* Deviceモードの時は、(0, `Camera Y Offset`, 0)  
  `Camera Y Offset`はDeviceモードの時のみ設定可能

にするので、プレイヤーを丸ごと移動させたいときには、XR Rigごと移動させる必要があります。  
**`Camera Floor Offcet Object`に設定したオブジェクトの位置はいじらない**ようにしましょう

ヒエラルキーウィンドウ右クリック / XR でXR Rigを作成する際、Room-ScaleとStationaryの2種類があってどっちを選べばいいかわからないことがあるかもしれません。  
2つの違いは

* Room-Scale XR RigはFloorモード
* Stationary XR RigはDeviceモード

だけです。なので、あとから切り替えするのも楽です。

なお、他にもモードがあるように見えますが、XRRig.cs内では全く触りません。  
(触れられていないので、わかりません)

### 2-2. TrackedPoseDriver

ヘッドマウントディスプレイのPosition、Rotationを読み込むコンポーネントです。
実はXR Interaction Toolkitの依存パッケージの機能なので、XR Interaction Toolkitを探しても見つかりません。

#### 共通部分 TrackedPoseDriver

![ActionBasedTrackedPoseDriverの画像](./Images/ActionBasedTrackedPoseDriver.png)
![DeviceBasedTrackedPoseDriverの画像](./Images/DeviceBasedTrackedPoseDriver.png)

上の図のTrackingType(6DoF or 3DoF)とUpdateType(Updateのタイミング)は共通しています。
どのデバイスのどの部分を使用するか設定する部分が変わっています。

このコンポーネントはXR対応しており、VRアプリ開発ではVRヘッドマウントディスプレイの、AR Foundationを利用したARアプリ開発ではARグラス、スマートフォンのPosition、Rotationを読み込むことができます。  
また、TrackingTypeで3Dofで使用するか、6DoFで使用するかも設定できます。

XR RigでCameraOffsetのPositionを操作し、TrackedPoseDriverでMainCameraのPosition, Rotationを操作するので、実際どういう動きになるんだ？と思う方もいると思うので、解説します。

XR RigはFloor Mode, Device Mode、TrackedPoseDriverはRotationAndPosition, RotationOnly, PositionOnlyとあります。
ここではPositionについて話すので、TrackedPoseDriverのRotationAndPositionとPositionOnlyはひとまとめにして説明します。

todo: 説明用画像

概要は上の図で理解してもらえると思います。

XR RigはCameraOffset(MainCameraの親)のPositionをFloor Modeで(0, 0, 0)に、Device Modeで(0, `Camera Y Offset`, 0)にします。  
つまり、MainCameraの土台の高さを6DoFを前提として0mとするか、3DoFを前提として`Camera Y Offset`mとするかを設定します。  
TrackedPoseDriverはその土台の上でどうふるまう(3DoFか6DoF)かを設定することになります。

もちろん意図してXR Rig Device ModeとTrackedPoseDriver RotationAndPositionで使用することもできますが、逆に意図していない場合は、カメラの位置が意図した場所からずれるといった問題の原因にもなります。

#### Action-based TrackedPoseDriver

![ActionBasedTrackedPoseDriverの画像](./Images/ActionBasedTrackedPoseDriver.png)

Action-baseのTrackedPoseDriverは``InputSystem.XR`内で実装されています。[リンク](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/api/UnityEngine.InputSystem.XR.TrackedPoseDriver.html)  
ファイルの位置は`Packeages/Input System/InputSystem/Plugin/XR/TrackedPoseDriver.cs`です。

Input Actionsをシリアライズすることができないので、手動で設定していく必要があります。  
マウス等でシミュレーションしたい場合は、Package ManagerのXR Interaction ToolkitのサンプルからSimuratorサンプルをインポートすると参考になると思います。

#### Device-based TrackedPoseDriver

![DeviceBasedTrackedPoseDriverの画像](Images/DeviceBasedTrackedPoseDriver.png)

Device-basedのTrackedPoseDriverは`UnityEngine.SpatialTracking`内で実装されています。[リンク](https://docs.unity3d.com/Packages/com.unity.xr.legacyinputhelpers@2.1/api/UnityEngine.SpatialTracking.TrackedPoseDriver.html)  
ファイルの位置は`Packeages/XR Regacy Input Helper/Runtime/TrackedPoseDriver/TrackedPoseDriver.cs`です。

特に説明することないです。

### 2-3. ※XRController

todo: 画像

コントローラのPosition, Rotation, ボタン入力を読み込むコンポーネントです。

#### 共通部分 XRController

#### Action-based XRController

Action-basedのXRControllerはInputSystem.XR内で実装されています。(TrackedPoseDriverを継承しています) [リンク](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/api/UnityEngine.InputSystem.XR.XRController.html)

コントローラのキーマッピングを行う機能がついていますが、そのためにInput Actionsファイルを要求します。  
Input Actionsファイルの作成は結構面倒(単純作業だけど量が...)かつ、バグを埋め込みやすい(経験談)ので、Package ManagerのXR Interaction ToolkitのサンプルからDefault Input Actionsサンプルをインポートして、改変していくことをお勧めします。

#### Device-based XRController

---

## 3. ※Interaction関係の機能

XR Interaction Toolkitのインタレクションは関数を呼ぶ〇〇Interactorと、呼ばれる関数を設定している〇〇Interactableの組で構成されています。
Interactorで取得したオブジェクトに対になるInteractableが設定されている場合、そのInteractableを呼ぶといった動作をします。todo: 確認

用意されているInteractor以外の機能が必要な場合は、XRBaseControllerInteractorやXRBaseInteractorを継承して新たにスクリプトを書くことで、オリジナルのコンポーネントを作成できます。
なお作成する場合は、DirectInteractorが一番シンプルなのでDirectInteractorのコードを参考にするのがよいと思います。

用意されているInteractableはTransformをいじる程度の機能しか用意されていません。
機能の拡張をしたい場合は、〇〇Interactableを継承して機能を追加するか、XRBaseInteractableを継承してオリジナルのコンポーネントを作成することができます。

### 3-1. InteractionManager

### XRBaseInteractor / Interactable

### 3-2. ※RayInteractor / Interactable

RayInteractorはレイキャストを飛ばし、一番手前のオブジェクトを取得する機能です。
一番手前のオブジェクトにRayInteractableコンポーネントが設定してある場合、そのRayInteractableを動作させます。

todo: 画像

#### 共通部分 RayInteractor / Interactable

#### Action-based RayInteractor / Interactable

#### Device-based RayInteractor / Interactable

### 3-3. ※DirectInteractor / Interactable

todo: 画像

#### 共通部分 DirectInteractor / Interactable

#### Action-based DirectInteractor / Interactable

#### Device-based DirectInteractor / Interactable

### 3-4. GrabInteractor / Interactable

### 3-5. SocketInreractor / Interactable

公式で用意しているInreractorの中で、唯一Controllerを利用しない機能です。  
コントローラ以外でInteractableを動作させたい場合にはこのスクリプトを参考に実装すると良いと思います。

SocketInteracttorに範囲を設定しておいて、対応するInteractableが範囲に入ったとき、そのInteractableを動作させます。

---

## 4. UI関係の機能

### 4-1. UI Canvas

### 4-2. UI EventSystem

---

## 5. ※移動、テレポーテーション関係の機能

### 5-1. ※Locomotion System

todo: 画像

#### 共通部分 Locomotion System

#### Action-based Locomotion System

#### Device-based Locomotion System

### 5-2. Teleportation Area

todo: 画像

#### 共通部分 Teleportation Area

#### Action-based Teleportation Area

#### Device-based Teleportation Area

### 5-3. Teleportation Ancker

---

## 6. AR関係の機能(TODO)

余裕があったら書きます。
