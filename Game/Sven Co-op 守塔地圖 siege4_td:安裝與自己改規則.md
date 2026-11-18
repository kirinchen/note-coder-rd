# Sven Co-op 守塔地圖 siege4_td:安裝與自己改規則

> 我在 CS 1.6 打 bot 打到膩,把 dust2 的感覺搬進 **Sven Co-op**(免費),做了一張四線守塔圖 **siege4_td**:基地在中間,怪從四條路衝進來,守住兩座金礦撐過 15 波。
> 這篇是影片裡「教你裝」和「自己改規則」兩段的文字版,指令可以直接複製。

- 🎬 影片:(上架後補)
- 📦 地圖包:[GitHub Release v1.0.0-siege4-td](https://github.com/kirinchen/svencoop-skills-toolkit/releases/tag/v1.0.0-siege4-td)(`siege4_td-v1.0.0.zip`)
- 🎮 Sven Co-op(Steam,免費):<https://store.steampowered.com/app/225840/Sven_Coop/>

---

## 安裝五步

### ① Steam 裝 Sven Co-op

免費。需要 **5.26 以上**。

### ② 找到 Sven 的資料夾

Steam 資料庫 → 對 **Sven Co-op** 按右鍵 → **管理** → **瀏覽本機檔案**。

裡面有一個 `svencoop_addon`,等一下東西全丟這裡。Windows 預設路徑:

```
C:\Program Files (x86)\Steam\steamapps\common\Sven Co-op\svencoop_addon\
```

### ③ 裝 CS 1.6 武器包(必要)

武器包是 KernCore 的 [Counter-Strike 1.6 Weapons Project](https://github.com/KernCore91/-SC-Counter-Strike-1.6-Weapons-Project),授權不允許重新打包,所以地圖 zip 裡**沒有**。
沒裝的話地圖能載入,但 script 編譯失敗:沒有商店、沒有怪、沒有金礦。**不需要有 CS 1.6。**

**方法 A:用腳本(最簡單)**

1. 裝好 Python 3.8+ 與 git
2. 下載 [`install_cs16_weapons.py`](https://github.com/kirinchen/svencoop-skills-toolkit/blob/v1.0.0-siege4-td/tools/install_cs16_weapons.py)
3. 在它所在的資料夾開終端機,執行:

```
python install_cs16_weapons.py --src D:\sc-cs16-weapons
```

它會自己下載武器包、複製進 `svencoop_addon`。Sven 不在 Steam 預設路徑時,加 `--sven "<Sven Co-op 路徑>"`。

**方法 B:手動複製**

下載武器包 zip、解壓縮,把這六樣複製進 `svencoop_addon\`:

```
models/cs16
sound/cs16
sprites/cs16
events
scripts/maps/cs16
cs16_resources.res
```

### ④ 地圖包 zip → 解壓縮

從 [Release 頁](https://github.com/kirinchen/svencoop-skills-toolkit/releases/tag/v1.0.0-siege4-td) 下載 `siege4_td-v1.0.0.zip`,解壓縮後裡面的 `maps`、`scripts`、`sprites` 整個丟進 `svencoop_addon\`,跟剛才的武器包合在一起。

### ⑤ 開圖

兩種都可以:

- 遊戲主選單 → 開新伺服器 → 地圖清單選 **siege4_td**
- 或主控台輸入:

```
map siege4_td
```

第一次載入比較久,等一下。

---

## 怎麼玩

| 操作 | 做什麼 |
|---|---|
| 軍火商按 **E** | 買槍、雇礦工 / 運彈兵 |
| 按 **G** | 把槍丟在地上,長成一座會開那把槍的塔 |
| 打完一波 | 去按按鈕叫下一波 |
| 撐過 **15 波** | 贏 |

已知:第 9 波 Race X 很硬(我就卡在這),第 10 波有 Gargantua。

---

## 自己改規則(叫 AI 幫你改)

這張圖的規則全在一個文字檔:`scripts/maps/siege4_td.as`(repo 裡是 `maps/siege4_td/siege4_td.as`)。不用會寫程式,打開 [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) 用講的就好。影片裡示範的是這句:

```
siege4_td 第一波 72 隻怪太多了,幫我改成一半。直接改 maps/siege4_td/siege4_td.as 裡的 WAVE_COUNT_MULT 就好。改完用一句話告訴我數字從多少變多少。
```

它把 `WAVE_COUNT_MULT` 從 6 改成 3。改完的檔案丟回:

```
svencoop_addon\scripts\maps\siege4_td.as
```

重開地圖就生效 → 第 1 波從 72 隻變 36 隻。

怪太硬、錢太少、想加一把槍,都這樣改。改出什麼好玩的,影片底下留言告訴我。

---

## 授權

地圖、script、sprite:MIT。材質用 Sven Co-op 內附的 `halflife.wad`。武器包依原作者授權。
