訓練流路管理系統 (Training Flow Management System)
一個基於 Web 的訓練排程與流路管理工具，專為訓練中心設計。本系統採用 Serverless 架構，前端使用 HTML/Tailwind CSS，後端結合 Google Apps Script (GAS) 與 Google Sheets 進行資料儲存與同步。
![alt text](https://img.shields.io/badge/license-MIT-blue.svg)

![alt text](https://img.shields.io/badge/version-1.0.0-green.svg)
✨ 主要功能
多維度資料管理：
消防人員訓練：管理梯次、人數、時數及跨縣市參訓需求（宜基北桃）。
非消防人員訓練：管理一般公務或外單位訓練課程。
重大活動：追蹤大型演習或活動的人力調用。
視覺化甘特圖：
依據輸入的日期區間自動產生全年度甘特圖。
支援智慧日期解析（如 1/5-8, 2/10）。
每日人力統計預覽。
報表輸出與匯入：
PDF 輸出：支援 A3 橫向排版，最佳化列印效果。
Excel 匯出：產生帶有格式的 .xlsx 報表。
CSV 匯入：支援批次匯入舊資料。
進階操作介面：
響應式設計 (RWD)，支援手機與電腦操作。
自定義日期選擇器（支援排除週末或特定日期）。
即時雲端同步。
🛠️ 技術棧
Frontend: HTML5, JavaScript (ES6+), Tailwind CSS (CDN)
Backend: Google Apps Script (GAS)
Database: Google Sheets
Libraries:
ExcelJS: 產生 Excel 報表
html2pdf.js: HTML 轉 PDF
PapaParse: CSV 解析
FileSaver.js: 檔案下載處理
Remix Icon: 圖標庫
🚀 安裝與部署指南
本系統由兩個部分組成：前端網頁 (index.html) 與 後端腳本 (Google Apps Script)。
步驟 1：設定 Google Sheets (資料庫)
建立一個新的 Google Sheet。
將試算表重新命名（例如：「訓練流路資料庫」）。
建立三個工作表 (Sheets)，分別命名為：
Sheet1 (對應消防人員)
Sheet2 (對應非消防人員)
Sheet3 (對應重大活動)
步驟 2：設定 Google Apps Script (後端 API)
在 Google Sheet 中，點選 擴充功能 > Apps Script。
將以下程式碼複製並貼上到 程式碼.gs (Code.gs) 中：
code
JavaScript
function doGet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  // 讀取三個工作表的資料 (假設有標題列，從第二行開始讀)
  const d1 = ss.getSheetByName('Sheet1').getDataRange().getValues().slice(1);
  const d2 = ss.getSheetByName('Sheet2').getDataRange().getValues().slice(1);
  const d3 = ss.getSheetByName('Sheet3').getDataRange().getValues().slice(1);
  
  return ContentService.createTextOutput(JSON.stringify({
    status: 'success',
    d1: d1,
    d2: d2,
    d3: d3
  })).setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    
    // 儲存資料的輔助函式
    const saveSheet = (sheetName, rows) => {
      const sheet = ss.getSheetByName(sheetName);
      sheet.clearContents(); // 清除舊資料
      // 重寫標題列 (請根據您的實際需求修改標題)
      const header = sheetName === 'Sheet1' 
        ? ['月份','承辦單位','課程名稱','對象','日期','地點','梯數','每梯人數','總人數','日數','總時數','合計時數','調用','跨縣市','新北','基隆','桃園','宜蘭','備註','排除日期'] 
        : sheetName === 'Sheet2'
        ? ['月份','承辦單位','課程名稱','對象','日期','地點','梯數','每梯人數','總人數','日數','總時數','合計時數','教官','跨縣市','備註','排除日期']
        : ['承辦單位','名稱','對象','日期','天數','時數','調用','地點','排除日期'];
        
      sheet.appendRow(header);
      
      if (rows && rows.length > 0) {
        // 將資料轉換為字串以避免格式錯誤
        const stringRows = rows.map(r => r.map(c => String(c)));
        sheet.getRange(2, 1, stringRows.length, stringRows[0].length).setValues(stringRows);
      }
    };

    saveSheet('Sheet1', data.d1);
    saveSheet('Sheet2', data.d2);
    saveSheet('Sheet3', data.d3);

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success',
      message: '資料儲存成功！'
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
點擊右上角的 「部署」 > 「新增部署作業」。
選擇類型：「網頁應用程式」。
設定如下：
執行身分：我 (您的帳號)
誰可以存取：所有人 (這是關鍵，否則前端無法存取)
點擊部署，並複製產生的 網頁應用程式網址 (Web App URL)。
步驟 3：設定前端
開啟專案中的 index.html。
找到約第 20 行的配置區域：
code
JavaScript
// ==========================================
// ⚠️ 1. 請填入您的流路管理 GAS 網址
const GAS_API_URL = "您的_GAS_WEB_APP_URL_貼在這裡";
// ==========================================
將您在步驟 2 取得的網址貼入 GAS_API_URL。
步驟 4：執行
直接用瀏覽器開啟 index.html，或將其上傳至 GitHub Pages 即可開始使用。
📖 使用說明
資料輸入：
點擊「新增資料列」開始輸入。
日期格式支援：1/5 (單日), 1/5-8 (連續), 1/5, 1/8 (多日), 1/5-8, 2/1 (混合)。
日期排除：
點擊日期欄位旁的小月曆圖示，可開啟進階選擇器，排除週末或特定不開課日期。
甘特圖：
切換至「甘特圖」頁籤，系統會自動根據輸入的日期繪製圖表。
若需列印，請點擊「下載 PDF」，系統會自動調整為 A3 格式。
儲存：
點擊上方工具列的 「儲存資料」 (黑色按鈕) 才會將變更寫入 Google Sheets。
⚠️ 注意事項
瀏覽器相容性：建議使用 Chrome、Edge 或 Safari 以獲得最佳列印與操作體驗。
列印設定：列印 PDF 時，請確保勾選「列印背景圖片 (Background graphics)」，否則甘特圖顏色可能無法顯示。
API 權限：若遇到 CORS 錯誤，請確認 GAS 部署時的存取權限是否設為「所有人」。
