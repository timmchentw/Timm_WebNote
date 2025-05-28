# Github Copilot

## 初始設定

1. 註冊帳號，這頁的開關打開就可以將原碼與public code做比對
  ![0.png](images/github%20copilot/16.png)
2. Visual studio安裝Github Copilot Extension
  ![1.png](images/github%20copilot/17.png)

## Visual Studio Code Extension
  
安裝完成後顯示在右下角</br>
![image](./images/github%20copilot/00.png)

* 註解產Code
  ![image](./images/github%20copilot/0.png)
  ![image](./images/github%20copilot/1.png)
  
* Copilot lab
  * 需先申請加入Lab計畫 (個人戶注意資料蒐集)，用法為圈選Code後在Tab中使用</br>
    ![image](./images/github%20copilot/2.png)
  * 解釋 (適合查詢語法用途)</br>
    ![image](./images/github%20copilot/3.png)
    ![image](./images/github%20copilot/21.png)
  * 翻譯</br>
    ![image](./images/github%20copilot/4.png)
  * 語法調整</br>
    ![image](./images/github%20copilot/5.png)
    * 原始Code
      ![image](./images/github%20copilot/6.png)
      * Fix Bugs (自動修正)
        ![image](./images/github%20copilot/7.png)
      * List Steps (註解步驟)
        ![image](./images/github%20copilot/8.png)
      * Document (文件化註解)
        ![image](./images/github%20copilot/9.png)
      * Make Robust (加入exception handler機制)
        ![image](./images/github%20copilot/10.png)
      * Chunk (職責切分)
        ![image](./images/github%20copilot/11.png)
      * Clean (縮排、去除註解等)
        ![image](./images/github%20copilot/12.png)
      * Debug (加入Logger)
        ![image](./images/github%20copilot/13.png)
      * Readable (排版與註解)
        ![image](./images/github%20copilot/14.png)
      * Add types (擴充參數?)
        ![image](./images/github%20copilot/15.png)
      * Custom (自行輸入指令)
        ![image](./images/github%20copilot/18.png)
        ![image](./images/github%20copilot/19.png)
  * Tests
    ![image](./images/github%20copilot/20.png)
  

  * 測試建置 (2023/8僅支援JS & TS)
* [Copilot for Docs](https://githubnext.com/projects/copilot-for-docs/)
  * 依據程度搜尋文件

* Copilot Inline
  * 開啟檔案可以參考，讓功能更聰明
  * Comment提供的資訊更多，產出的程式碼會更精準
  * 自動在類似的地方加入建議 (比如註解、產生測試)
  * 重複的Code適合使用

* Copilot Chat
  * VS也可以使用
  * 可用於重構程式碼
  * 給予更多語法與優化方法

### 快捷鍵
![image](./images/github%20copilot/22.png)
![image](./images/github%20copilot/23.png)

### 語法

- `@workspace`: 指定範圍
  - `#file`
  - `#selection`
  - `#editor`: 整個編輯器
- 範例
  - 前端
    - `Fixed Header`, `Fixed Footer`: 頁頭頁尾CSS+HTML範例
  - Code
    - `What the code do?`
    - `Explain this regex`
    - `Fix bugs`
    - `Create unit test`
    - `Change the variables with better name`
  - SQL
    - `Find all columns with name including "XXX"` 
      - 產出 `SELECT * FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME LIKE 'roduct'
`
    - `The MB used by the database`
    - `Optimizing the function XXX`
    - `Adding Index to table XXX`

## 心態

- 不要讓AI主掌職責 (Copilot是輔助)
  - 不一定安全
  - 一定要自己Review
  - 可讓AI指明方向
- 善用適合的工具
  - 優先用Inline chat (不要用comment promt)
    - `/fix`: 快速修BUG
    - `/explain`
    - `/fix`
    - `/tests`
  - 加速填入則ghost text較有效率
  - 詳細功能用Chat panel
    - `/api`
    - `/search`
    - `/newChat`
- 提供上下文(context)
  - 選取要問的Code範圍
  - 關聯Code要打開
- Feedback功能
  - 問答有沒有幫助按下讚/倒讚

## 相關資源

* [讓會讀心術的 Github Copilot 陪你寫程式](https://blog.darkthread.net/blog/github-copilot-demo/)
* [GitHub Copilot 開發者訓練營筆記]([https://blog.darkthread.net/blog/github-next/)