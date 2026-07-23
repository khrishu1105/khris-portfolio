# Three.js 捲動背景動畫範本

一個純 WebGL、即時運算的網站背景動畫，可捲動驅動，適合放在作品集或個人網站首頁。
風格參考自 [sites.amping.io](https://sites.amping.io)，並照其實際使用的技術複製而成。

作者：Khris

---

## 怎麼打開

雙擊 `index.html` 用瀏覽器開啟即可，不需安裝任何東西、不需網路以外的環境。
（Three.js 透過 CDN 載入，第一次開啟需要連網。）

---

## 這是用什麼做的

| 技術 | 用途 |
|------|------|
| **Three.js r160**（WebGL 3D 引擎） | 即時繪製整個 3D 場景 |
| **importmap + ES Module** | 直接從 CDN 載入，免打包工具、純 HTML |
| **`InstancedMesh`** | 一次繪製 800 個方塊，效能極高 |
| **`MeshStandardMaterial` + `DirectionalLight`** | PBR 材質與光影，做出立體反光質感 |
| **`Clock` + `lerp`（線性插值）** | 時間驅動動畫 + 平滑慣性手感 |
| **捲動進度映射** | 把頁面捲動換算成 0~1，驅動鏡頭與方塊聚散 |

沒有影片、沒有 GIF——所有畫面都靠顯示卡即時繪製，因此無論縮放都清晰，檔案也極小。

---

## 這個範本是怎麼「還原」出來的

還原度來自 **先偵測、再對照**，而不是憑印象重做：

### 1. 抓原始 HTML，偵測技術指紋
WebFetch 轉 markdown 會過濾掉 script，所以改用 `curl` 抓原始 HTML 再 grep：

```
21 THREE      ← Three.js 被提及 21 次
 4 WebGL
 3 rive
```

確定核心是 Three.js（WebGL），不是影片。

### 2. 挖出它實際用的 API 與出現次數
```
9 group   8 lerp   4 clock   2 DirectionalLight   1 InstancedMesh
```
把這份清單當成「技術指紋」，一項一項對照著搭，所以手感才會接近。

### 3. 關鍵領悟：高級感的祕密是 `lerp`
原始碼裡 `lerp` 出現特別多次。它的質感不在 3D 本身，而在**所有動作都不是瞬間到位，而是慢慢逼近目標**：

```js
smooth += (target - smooth) * 0.04;  // 每幀只移動 4%，產生慣性延遲的絲滑感
```

拿掉這行，方塊照樣動，但會變得生硬廉價。這是「像不像」的分水嶺。

---

## 運作原理

### 捲動 → 動畫進度
整頁捲動位置被換算成 `0~1` 的進度值 `p`：

```js
scrollTarget = window.scrollY / max;
scrollSmooth += (scrollTarget - scrollSmooth) * 0.06; // 平滑慣性
```

`p` 用來驅動：
- **鏡頭前進**：`camera.position.z` 從 18 推進到 4，像穿過方塊群
- **方塊聚散**：每顆方塊有「散開座標」與「聚合座標」兩組，用 `p` 在兩者間插值
- **整團旋轉**：捲動時多轉半圈

### 滑鼠視差
鏡頭隨滑鼠輕微擺動，同樣用 `lerp` 平滑，產生慣性延遲感。

---

## 可以自己調的地方

| 想改什麼 | 改哪裡（`index.html`） |
|---------|----------------------|
| 方塊數量 | `const COUNT = 800` |
| 換形狀（球、環…） | `new THREE.BoxGeometry` → `SphereGeometry` / `TorusGeometry` |
| 配色 | 第 3 段的三個 `color.setHex(...)` 色碼 |
| 鏡頭飛多遠 | `camera.position.z = 18 - p * 14`（改 14） |
| 捲動反應快慢 | `scrollSmooth += (...) * 0.06`（調大=跟得緊，調小=更飄） |
| 滑鼠跟隨快慢 | `smooth.x += (...) * 0.04` |
| 聚合緊密度 | seeds 裡 `gx/gy/gz` 的 `* 8` |
| 背景色 | CSS `--ink` 與 `scene.fog` 色碼要一起改 |
| 加頁面文字 | 複製一個 `<section>` 即可 |

---

## 檔案結構

```
three_bg_demo/
├── index.html   ← 全部程式碼都在這（HTML + CSS + JS 單檔）
└── README.md    ← 本說明
```
