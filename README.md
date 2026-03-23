# ARIA v2.0 - 避難收容所地形複合風險評估系統

## 專案概述 (Project Overview)
本專案為第一代 ARIA 系統的進階升級版，導入花蓮縣 20m DEM 高程網格資料，針對花蓮縣的避難收容處所進行深度的空間地形分析。系統結合了向量資料（河川面、鄉鎮市區界、避難所點位）與網格資料（DEM），透過建立緩衝區與 Zonal Statistics，為各避難所計算「平均高程」與「最大坡度」，最終產出結合水文與地形的複合風險評估模型與視覺化地圖。

## 系統核心功能 (Core Features)
* **網格與向量資料介接**：處理不同 CRS 的統一投影（EPSG:3826），並透過空間交集（Spatial Join）精準篩選目標縣市設施。
* **複合風險分級邏輯**：整合距離河川遠近（<500m, <1000m）、坡度（>30度）與高程（<50m）等條件，進行多維度風險判定（極高、高、中、低風險）。
* **製圖與報表匯出**：
  - 產出結構化的地形風險審計清單 (`terrain_risk_audit.json`)。
  - 繪製具備海拔分層設色、行政區劃標籤與風險點位的經緯度（WGS84）高畫質地圖 (`terrain_risk_map_final.png`)。

---

## 🤖 AI 診斷日誌 (AI Diagnostic Log)

在進行 Vibecoding 開發的過程中，遭遇到以下的資料處理問題：

### 1. 沿海高程零值導致繪圖崩潰 
在繪製彩色地形圖疊加層時，原本嘗試使用 `matplotlib.colors.LogNorm` 來凸顯海拔差異。但因為花蓮為沿海縣市，DEM 中包含了高程為0的海平面，導致程式在計算色階最小值 (`vmin`) 時崩潰。
解決方案：捨棄對數規範，改用標準的線性色階，並顯式宣告 `vmin=0`，讓海平面與低海拔地區呈現自然，也讓沿海地形的視覺呈現更符合真實情況。

### 2. Zonal Stats 回傳 NaN（CRS 未對齊或像素未覆蓋）
在運用 `rasterstats.zonal_stats` 計算避難所 500m 緩衝區的地形屬性時，發現部分資料回傳了 `NaN`。
先確保所有向量交集與緩衝區建立前，皆強制投影至公尺單位的 EPSG:3826，確保疊合精準度。透過 `np.where` 將坡度矩陣中的無效區域同步挖空，並在呼叫 `zonal_stats` 時，嚴格宣告 `nodata=nodata_val` 與 `nodata=np.nan` 參數，強迫系統計算時忽略海面與邊界外的無效像素。

---

## 檔案結構 (Deliverables)
* `ARIA_v2.ipynb`：包含完整分析流程、Captain's Log 註解的 Jupyter Notebook。
* `terrain_risk_audit.json`：190 筆花蓮縣避難所的地形風險清單。
* `terrain_risk_map_final.png`：地形複合風險視覺化地圖及Top 10 高風險避難所的坡度。

## 環境與依賴套件 (Dependencies)
```bash
pip install pandas geopandas rasterio rioxarray rasterstats shapely matplotlib python-dotenv