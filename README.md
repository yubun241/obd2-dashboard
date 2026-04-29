## obd2-dashboard
BLE ELM327 OBD2 ダッシュボード

Web Bluetooth API + GitHub Pages + PWA 構成

## 動作イメージ
[obd2_dashboard_mockup.html](https://github.com/user-attachments/files/27190227/obd2_dashboard_mockup.html)

<div style="background:#111827; border-radius:16px; padding:16px; overflow:hidden;">

  <div style="background:#0B1020; border-radius:12px; padding:12px 16px; font-family:'Courier New',monospace;">

    <div style="display:flex; align-items:center; justify-content:space-between; margin-bottom:8px;">
      <div style="display:flex; gap:8px; align-items:center;" id="dots-row"></div>
      <span style="color:#5A7080; font-size:10px;">Mini F56 JCW</span>
    </div>

    <div style="height:1px; background:#1C3050; margin-bottom:10px;"></div>

    <div style="display:flex; gap:0; align-items:stretch; min-height:160px;">

      <div style="flex:0 0 35%; display:flex; flex-direction:column; align-items:center; justify-content:center; border-right:1px solid #1C3050; padding-right:12px;">
        <div id="speed-val" style="font-size:56px; font-weight:bold; color:#EEF2F5; line-height:1;">87</div>
        <div style="font-size:11px; color:#AABBCC; margin-bottom:10px;">km/h</div>
        <div id="boost-val" style="font-size:44px; font-weight:bold; color:#FFD200; line-height:1;">0.82</div>
        <div style="font-size:11px; color:#FFD200;">kg/cm²</div>
      </div>

      <div style="flex:0 0 22%; display:flex; align-items:center; justify-content:center; border-right:1px solid #1C3050; padding:6px;">
        <div id="gear-box" style="width:100%; height:140px; background:#0E1E36; border-radius:12px; border:2.5px solid #EEF2F5; display:flex; align-items:center; justify-content:center;">
          <span id="gear-val" style="font-size:80px; font-weight:bold; color:#EEF2F5; line-height:1;">3</span>
        </div>
      </div>

      <div style="flex:1; display:flex; flex-direction:column; justify-content:space-evenly; padding-left:14px;">
        <div style="display:flex; gap:6px; align-items:baseline;">
          <span style="color:#5A7080; font-size:11px;">Coolant:</span>
          <span id="sv-coolant" style="color:#EEF2F5; font-size:13px; font-weight:bold;">92</span>
          <span style="color:#5A7080; font-size:10px;">°C</span>
        </div>
        <div style="display:flex; gap:6px; align-items:baseline;">
          <span style="color:#5A7080; font-size:11px;">Intake:</span>
          <span style="color:#EEF2F5; font-size:13px; font-weight:bold;">28</span>
          <span style="color:#5A7080; font-size:10px;">°C</span>
        </div>
        <div style="display:flex; gap:6px; align-items:baseline;">
          <span style="color:#5A7080; font-size:11px;">MAP:</span>
          <span id="sv-map" style="color:#EEF2F5; font-size:13px; font-weight:bold;">182</span>
          <span style="color:#5A7080; font-size:10px;">kPa</span>
        </div>
        <div style="display:flex; gap:6px; align-items:baseline;">
          <span style="color:#5A7080; font-size:11px;">Fuel:</span>
          <span style="color:#EEF2F5; font-size:13px; font-weight:bold;">378</span>
          <span style="color:#5A7080; font-size:10px;">kPa</span>
        </div>
      </div>
    </div>

    <div style="height:1px; background:#1C3050; margin-top:10px; margin-bottom:6px;"></div>

    <div style="display:flex; align-items:center; gap:8px;">
      <span style="color:#1FD060; font-size:11px; font-weight:bold;">OBD: Connected</span>
      <span id="ratio-lbl" style="color:#5A7080; font-size:9px;"></span>
      <div style="margin-left:auto; background:#FF3D1A; color:#0B1020; border-radius:4px; padding:2px 10px; font-size:11px; font-weight:bold;">切断</div>
    </div>

  </div>
</div>

<script>
const RPM_MAX = 8000, DOTS = 12, REDLINE = 0.82;
const GEARS = ['N','1','2','3','4','5','6'];
const GEAR_COLORS = { N:'#4AABFF', '1':'#FF8000', R:'#FF3D1A' };

let t = 0;

const dotsRow = document.getElementById('dots-row');
for (let i = 0; i < DOTS; i++) {
  const d = document.createElement('div');
  d.id = 'dot'+i;
  d.style.cssText = 'width:14px;height:14px;border-radius:50%;border:1.5px solid #1C3050;flex-shrink:0;';
  dotsRow.appendChild(d);
}

function updateDots(rpm) {
  const active = Math.min(Math.floor(rpm / RPM_MAX * DOTS), DOTS);
  const rd = Math.floor(DOTS * REDLINE);
  for (let i = 0; i < DOTS; i++) {
    const d = document.getElementById('dot'+i);
    if (i < active) {
      d.style.background = i >= rd ? '#FF3D1A' : '#4AABFF';
      d.style.border = 'none';
    } else {
      d.style.background = 'transparent';
      d.style.border = '1.5px solid #1C3050';
    }
  }
}

function animate() {
  t += 0.025;

  const rpm   = Math.round(2200 + Math.sin(t * 0.7) * 1400 + Math.sin(t * 2.1) * 400);
  const speed = Math.round(75 + Math.sin(t * 0.5) * 30 + Math.sin(t * 1.3) * 12);
  const boost = (0.65 + Math.sin(t * 0.8) * 0.28 + Math.sin(t * 2.2) * 0.08).toFixed(2);
  const map   = Math.round(160 + Math.sin(t * 0.8) * 35);
  const coolant = Math.round(88 + Math.sin(t * 0.1) * 6);

  const ratio = rpm / Math.max(speed, 1);
  let gear = 'N';
  const table = [[3.308,1],[1.913,2],[1.258,3],[0.943,4],[0.742,5],[0.596,6]];
  const factor = 3.647 * 1000 / (Math.PI * 616 / 1000 * 60);
  let best = 0.20;
  for (const [gr, g] of table) {
    const exp = gr * factor;
    const pct = Math.abs(ratio - exp) / exp;
    if (pct < best) { best = pct; gear = String(g); }
  }

  updateDots(rpm);
  document.getElementById('speed-val').textContent = speed;
  document.getElementById('boost-val').textContent = boost;
  document.getElementById('sv-map').textContent = map;
  document.getElementById('sv-coolant').textContent = coolant;
  document.getElementById('sv-coolant').style.color = coolant >= 105 ? '#FF3D1A' : '#EEF2F5';
  document.getElementById('ratio-lbl').textContent = speed > 2 ? (ratio).toFixed(1)+' r/v' : '';

  const gv = document.getElementById('gear-val');
  const gb = document.getElementById('gear-box');
  const col = GEAR_COLORS[gear] || '#EEF2F5';
  gv.textContent = gear;
  gv.style.color = col;
  gb.style.borderColor = col;

  requestAnimationFrame(animate);
}

animate();
</script>



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





