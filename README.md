## obd2-dashboard
BLE ELM327 OBD2 ダッシュボード

Web Bluetooth API + GitHub Pages + PWA 構成

## 表示項目
| 項目 | 説明 |
|------|------|
| 速度 | km/h |
| ブースト圧 | kg/cm² |
| ギア | 物理演算による推定（1〜6・N・R） |
| 水温 | °C（105°C以上で赤色警告） |
| 吸気温 | °C |
| MAP | kPa |
| 燃圧 | kPa |
| RPMドット | レッドライン付きインジケーター |

※現在はJCWのギア比に合わせて開発しています

　他の車種にする場合は変更が必要です


 
## 必要なもの
- Android スマートフォン
  
- Google Chrome（最新版）
  
- BLE ELM327 ドングル（推奨: **Vgate iCar Pro BLE 4.0**）

- GitHub アカウント（無料）



## セットアップ手順

### 1. リポジトリ作成

1. [github.com](https://github.com) をChromeで開いてログイン
   
3. 右上 `+` → `New repository`
4. Repository name: `obd2-dashboard`
   
6. **Public** を選択
  
7. `Create repository` をクリック



### 2. ファイルをアップロード
リポジトリ作成後、以下の4ファイルをアップロード：

- `index.html`
  
- `manifest.json`
  
- `sw.js`
  
- `icon.png`

手順：

1. リポジトリページで `Add file` → `Upload files` 

2. 4ファイルをドラッグ＆ドロップ

3. `Commit changes` をクリック



### 3. GitHub Pages を有効化

1. リポジトリの `Settings` → `Pages`

2. Source: `Deploy from a branch`

3. Branch: `main` / `(root)` → `Save`

4. 数分後に以下のURLで公開される：



### 4. ホーム画面にアプリとして追加（PWA）

1. AndroidのChromeで上記URLを開く

2. Chrome右上 `⋮` → `ホーム画面に追加`

3. `追加` をタップ

4. ホーム画面にアイコンが出現 → アプリ化完成


## 使い方

1. エンジンを**ON**にする（必須）

2. アプリを起動

3. 「接続」ボタンをタップ

4. デバイス選択画面でELM327ドングルを選択

5. `Connected`（緑）になればデータ取得開始



## 対応ドングル

| ドングル | 動作確認 |
|----------|----------|
| Vgate iCar Pro BLE 4.0 | ◎ 推奨 |
| KONNWEI BLE | ○ |
| OBDLink CX | ◎ |
| 格安クローン品 | △ 動作不安定な場合あり |


**注意:** 
BLE ELM327はiOS/Androidの「設定 → Bluetooth」からはペアリング不要。アプリ内の「接続」ボタンから直接接続する。



## 車両設定（Mini F56 JCW デフォルト値）


`index.html` 内の `CFG` オブジェクトで変更可能：


```javascript
const CFG = {
  finalDrive:   3.647,
  tireDiamMm:   616,
  gearRatios:   [3.308, 1.913, 1.258, 0.943, 0.742, 0.596],
  tolerancePct: 0.18,
  hysteresis:   3,
  minSpeed:     4,
  minRpm:       500,
  maxRpm:       8000,
  redlinePct:   0.82,
};
```




## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| デバイスが見つからない | Bluetoothが未許可 | Chrome → サイト設定 → Bluetooth を許可 |
| 接続はできるがデータが来ない | エンジンOFFのまま | エンジンをかけてから接続 |
| `Match failed` と表示 | 未知のService UUID | ドングルの型番を確認 |
| ギアが正しく表示されない | 車両設定が違う | `CFG` のギア比・タイヤ径を調整 |

GitHubのリポジトリで Add file → Create new file → ファイル名を README.md にして貼り付けてください。





