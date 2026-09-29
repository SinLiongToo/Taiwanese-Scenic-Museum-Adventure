---
name: taiwanese-cultural-webapp
description: "Guidelines and architecture for curating, developing, and auditing Taiwanese Hokkien cultural, educational, and geography WebApps (四界行 series). Covers authentic MOE Tâi-lô Romanization, Tone Sandhi Engine (連音變調) algorithms, Leaflet NLSC Taiwan map integration (zero API key), ChhoeTaigi KipSutian audio mirroring, and dual-mode data synchronization."
---

# Taiwanese Cultural & Educational WebApp Skill

This skill provides guidelines, architectural patterns, and auditing procedures for building, maintaining, and scaling Taiwanese Hokkien (臺灣台語) educational and cultural WebApps (including the **四界行 Adventure series**).

---

## 1. Core Architecture & Philosophy

Every "四界行" (Adventure) application is a lightweight, zero-backend, client-side interactive WebApp designed for learners, educators, and children.

- **Clean Client-Side Stack**: Vanilla HTML5, CSS3, and JavaScript in a standalone or modular structure.
- **Authentic Linguistic Authority**: All Romanization and audio mappings must strictly follow the Ministry of Education's dictionary (教育部《臺灣閩南語常用詞辭典》) and the ChhoeTaigi community standards. Never use mechanical machine translations.
- **Zero API-Key Free Maps**: Use the National Land Surveying and Mapping Center (NLSC 內政部國土測繪中心) map tiles for 100% free, watermark-free, official Traditional Chinese mapping.
- **Dual-Mode Audio & Tone Sandhi**: Real human audio clips played with authentic Taiwanese Tone Sandhi (連音變調), backed up by offline browser speech synthesis.

---

## 2. Linguistic Standards (Tâi-lô & ChhoeTaigi)

### 2.1 Standard Orthography
- **Tâi-lô (台灣閩南語羅馬字拼音方案)**:
  - Tones 1 through 8 (Tone 6 merged with 2 in standard prestige accent).
  - Codas: Entering stop codas (`-p`, `-t`, `-k`) and glottal stop coda (`-h`).
  - Nasalization: use `nn` or superscript `ⁿ` consistently (e.g. `soaⁿ` / `suann`).
  - Open-o: use `oo` or `o͘`.
- **Verified Common Pronunciations**:
  - `二二八`: `Jī-jī-pat` (`ji7-ji7-pat4`)
  - `紀念碑`: `Kì-liām-pi` (`ki3-liam7-pi1`)
  - `和平`: `Hô-pîng` (`ho5-ping5`)
  - `人權`: `Jîn-khoân` (`jin5-khuan5`)
  - `億載金城`: `It-tāi-kim-siânn` (`it4-tai7-kim1-siann5`)
  - `門`: `mn̂g` (`mng5`)
  - `城`: `siânn` (`siann5`)
  - `森林`: `sim-lîm` (`sim1-lim5`)
  - `歌劇`: `koa-ke̍k` (`kua1-kiok8`)

---

## 3. Tone Sandhi Engine (連音變調轉換引擎)

### 3.1 The Fundamental Law
In spoken Taiwanese:
> **「前字變調，後字本調」 (Non-final syllables change tones; the final syllable retains citation tone).**

### 3.2 Standard Sandhi Rules (教育部優勢腔)

| 原始調號 (本調) | 變調調號 | 規則說明 | 範例 |
|---|---|---|---|
| **第 1 調 (陰平)** | ➔ **第 7 調** | 高平 ➔ 中平 (55 ➔ 33) | 金 `kim1` ➔ `kim7` |
| **第 2 調 (陰上)** | ➔ **第 1 調** | 高降 ➔ 高平 (51 ➔ 55) | 古 `kóo (2)` ➔ `koo1` |
| **第 3 調 (陰去)** | ➔ **第 2 調** | 低降 ➔ 高降 (21 ➔ 51) | 紀 `kì (3)` ➔ `ki2` |
| **第 4 調 (陰入 -h)** | ➔ **第 2 調** | 喉塞音脫落變高降 | 樹仔 `chhiū-á` |
| **第 4 調 (陰入 -p/-t/-k)** | ➔ **第 8 調** | 短促入聲變高入 | 億 `it4` ➔ `it8` |
| **第 5 調 (陽平)** | ➔ **第 7 調** | 低升 ➔ 中平 (24 ➔ 33) | 和 `hô (5)` ➔ `ho7` |
| **第 7 調 (陽去)** | ➔ **第 3 調** | 中平 ➔ 低降 (33 ➔ 21) | 二 `jī (7)` ➔ `ji3` |
| **第 8 調 (陽入 -h)** | ➔ **第 3 調** | 喉塞音脫落變低降 | 白 `pe̍h (8)` ➔ `pe3` |
| **第 8 調 (陽入 -p/-t/-k)** | ➔ **第 4 調** | 短促入聲變低入 | 達 `ta̍t (8)` ➔ `tat4` |

### 3.3 Phrase Chunks & 2+2 Compounds
- **2-Syllable Phrase**: Word 1 (sandhi) + Word 2 (base tone). E.g., `和平` ➔ `ho7-ping5`.
- **3-Syllable Phrase**: Word 1 (sandhi) + Word 2 (sandhi) + Word 3 (base tone). E.g., `二二八` ➔ `ji3-ji3-pat4`.
- **4-Syllable Compound**: Default to `2+2` split sandhi. E.g., `億載金城` ➔ `[it8-tai7] [kim7-siann5]`.
- **Long Place Names**: Decompose into semantic phrase chunks. E.g., `基隆二二八紀念碑` ➔ `[ki7-liong5] [ji3-ji3-pat4] [ki2-liam3-pi1]`.

### 3.4 Phonological Fallbacks
- Free variations: Support `j` ↔ `l` (日母與來母自由互換)，使 `ji3` 在資料庫中查無時，能無縫調用教育部 `li3.mp3`。

---

## 4. Map Integration (NLSC Taiwan Electronic Map)

### 4.1 Tile Provider Configuration
Never use CARTO (which requires API keys or adds watermarks) or raw OpenStreetMap (which may 403 rate-limit). Use the official NLSC Taiwan General Electronic Map:

```javascript
const nlscLayer = L.tileLayer('https://wmts.nlsc.gov.tw/wmts/EMAP/default/GoogleMapsCompatible/{z}/{y}/{x}', {
    maxZoom: 19,
    attribution: '© <a href="https://maps.nlsc.gov.tw/" target="_blank">國土測繪圖資服務雲 (NLSC)</a>'
});
```

### 4.2 County Normalization Standard
Always standardize county names to `台` (`台北市`, `台中市`, `台南市`, `台東縣`) instead of `臺` to prevent duplicate tag split and missing coordinates, while registering `臺` aliases in `countyCenters` and `countyZooms` for backward compatibility.

---

## 5. Verification & Quality Assurance Suite

Run these automated checks before any commit or push:
1. `fix_escapes.py`: Ensure template literals `${...}` are not escaped to `\${...}`.
2. `diagnose.js`: Verify 100% location and vocabulary coverage in `wordSyllableMap`, check for duplicate IDs, missing fields, or invalid coordinates.
3. `verify_syntax.js`: Verify zero syntax errors in embedded JavaScript.
4. `generate_static_files.js`: Regenerate `vocabulary.json` and `vocabulary.md` to guarantee exact count and schema parity.
