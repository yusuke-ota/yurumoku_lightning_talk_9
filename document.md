# XR Interaction Toolkit 1.0.0-pre.1をAction-basedで使いたい

## 注

* 日本語の資料があまりなかったので書きました。品質は期待しないでください。
* タイトルの通り、XR Interaction Toolkitバージョン1.0.0-pre.1での情報です。
* Action-basedで変わるのは、XR Rig(コントローラとHMDの入力)周りです。その他の部分に関しては以前のバージョンの資料で大丈夫です。
* Qiitaはドメインパワーが強すぎるので、こういうニッチな物は載せないようにしています。ご了承ください。

## 目次

1. そもそも
   1. XR Interaction ToolkitのAction-basedって何？
   2. Input Systemって何？
2. ※HMD、コントローラの位置情報、入力関係の機能
   1. ※カメラ(todo)
   2. ※XRController
3. ※Interaction関係の機能
   1. InteractionManager
   2. ※RayInteractor / Interactable
   3. ※DirectInteractor / Interactable
   4. GrabInteractor / Interactable
   5. SocketInreractor / Interactable
4. UI関係の機能
   1. UI Canvas
   2. UI EventSystem
5. ※移動、テレポーテーション関係の機能
   1. ※Locomotion System
   2. Teleportation Area
   3. Teleportation Ancker
6. AR関係の機能(TODO)

`※`マークがついている項目はAction-based版がある機能です。

その他の機能はAction-basedとDevice-basedで共通です。
そこそこ情報が出ているので、このドキュメントを読まなくても良いです。

## 1.そもそも

Action-based以前にXR Interaction Toolkitって何？という方もいると思うので、そのあたりの説明を入れます。  
不要な方は 2.※HMD、コントローラの位置情報、入力関係の機能 以降に進んでください。

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

## 2. ※HMD、コントローラの位置情報、入力関係の機能

### 2-1. ※XR Rig

SteamVR Pluginでいうところの`[SteamVR]`プレハブです。  
以下のような構成になっています。

```txt todo: 画像に差し替え
XRRig(CameraOffsetのPosition.Yを変更するスクリプト付)
 └ CameraOffset(スクリプトはついていない。HMDの高さ反映用のオブジェクト)
    ├ XRCamera
    ├ LeftController(RayInteractor, Ray可視化用のスクリプト付)
    └ RightController(RayInteractor, Ray可視化用のスクリプト付)
```

XRCameraについている todo と、[Left / Right]Controllerについている XRControllerはAction-based版とDevice-based版があります。
XRControllerについては後で説明するので、ここではCameraについて記載します。

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

### 2-2. ※XRController

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

## 3. ※Interaction関係の機能

XR Interaction Toolkitのインタレクションは関数を呼ぶ〇〇Interactorと、呼ばれる関数を設定している〇〇Interactableの組で構成されています。
Interactorで取得したオブジェクトに対になるInteractableが設定されている場合、そのInteractableを呼ぶといった動作をします。

用意されているInteractor以外の機能が必要な場合は、XRBaseControllerInteractorやXRBaseInteractorを継承して新たにスクリプトを書くことで、オリジナルのコンポーネントを作成できます。
なお作成する場合は、DirectInteractorが一番シンプルなのでDirectInteractorのコードを参考にするのがよいと思います。

用意されているInteractableはTransformをいじる程度の機能しか用意されていません。
機能の拡張をしたい場合は、〇〇Interactableを継承して機能を追加するか、XRBaseInteractableを継承してオリジナルのコンポーネントを作成することができます。

### 3-1. InteractionManager

### 3-2. ※RayInteractor / Interactable

RayInteractorはレイキャストを飛ばし、一番手前のオブジェクトを取得する機能です。
一番手前のオブジェクトにRayInteractableコンポーネントが設定してある場合、そのRayInteractableを動作させます。

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

### 3-3. ※DirectInteractor / Interactable

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

### 3-4. GrabInteractor / Interactable

### 3-5. SocketInreractor / Interactable

公式で用意しているInreractorの中で、唯一Controllerを利用しない機能です。  
コントローラ以外でInteractableを動作させたい場合にはこのスクリプトを参考に実装すると良いと思います。

SocketInteracttorに範囲を設定しておいて、対応するInteractableが範囲に入ったとき、そのInteractableを動作させます。

## 4. UI関係の機能

### 4-1. UI Canvas

### 4-2. UI EventSystem

## 5. ※移動、テレポーテーション関係の機能

### 5-1. ※Locomotion System

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

### 5-2. Teleportation Area

todo: 画像

#### 共通部分

#### Action-based

#### Device-based

### 5-3. Teleportation Ancker

## 6. AR関係の機能(TODO)

余裕があったら書きます。
