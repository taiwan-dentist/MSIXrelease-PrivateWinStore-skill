---
name: windows-msix-private-store-release
description: "將任何語言製作的 Windows 桌面程式整理成可提交 Microsoft Store 私人對象的 MSIX，並完成版本延續、簽章、審核準備、本機安裝、偵錯、更新與人工驗收。當工作進入 Windows 軟體的正式交付、發布或更新版本階段，且使用者希望或可能希望透過 Microsoft Store 私人發布時使用；一般尚未進入發布階段的功能開發不需觸發。"
---

# Windows MSIX 私人商店發布

## 目標

在 Windows 軟體進入正式交付或版本更新階段時，除了產生原本的可執行檔，也要評估並在適合時產生可供 Microsoft Store 提交的 MSIX。交付必須附有可核對的測試證據；「編譯成功」不能代替「MSIX 安裝後可以正常使用」。

MSIX 是安裝與配送封裝，不是新的程式語言或 GUI 框架。Rust、Python、C++、C# 或其他語言仍先產生 Windows 可執行檔，再把執行檔、相依元件、圖示、資源及套件資訊封裝成 MSIX。Slint、Qt、Tk、Tauri、Electron 或其他 GUI 原則上可以保留，但必須實測封裝後的行為。

## 工作邊界

- 先讀取專案中的 `AGENTS.md`、既有發布文件及封裝設定，保留使用者已經採用的建置與版本規則。
- 不因使用本 skill 而自行提交、發布或變更 Partner Center 中的產品。只有使用者已授權時才進行外部發布動作。
- 先盤點現有語言工具鏈、Windows SDK、封裝工具、簽章工具及測試環境。若已授權的發布工作缺少必要的建置或驗證依賴，應在合適的隔離環境中安裝並完成驗證，不能因工具缺少就直接把必要測試列為未測。
- 私人憑證金鑰、PFX 密碼、Partner Center 認證資訊、公司機密、客戶資料及正式工作資料不得寫入原始碼、套件、測試報告或版本庫。
- 不覆寫先前發布的 EXE、MSIX 或測試報告。使用含版本的檔名及獨立輸出目錄。
- 若程式依賴驅動程式、Windows 服務、系統層安裝、管理員權限、固定內網或無法提供給 Microsoft 審核者的環境，及早標示為 Store 審核風險，不要假設轉成 MSIX 就一定能上架。

## 缺少建置或驗證工具時

不要假定目標語言。先依原始碼、鎖定檔、專案設定及既有建置文件判斷實際工具鏈，再處理缺少的依賴。

- 發布任務已包含建置與驗證授權時，安裝完成該任務所必需的編譯器元件、語言套件、測試套件、封裝工具或驗證工具。安裝後實際執行目標測試；只確認指令存在不算完成。
- 優先使用專案隔離環境與既有套件管理方式，例如 Python 專案的 `.venv`、Node 專案的專案相依套件、Rust 的專案工具鏈與鎖定檔。不要把專案專用套件任意裝進全域環境。
- 尊重既有鎖定檔與版本限制。新增依賴前確認來源、用途及與目標平台的相容性，只安裝完成工作所需的最小集合，並在交付報告記錄新增的工具與版本。
- Windows SDK、Visual Studio Build Tools、憑證信任、驅動程式、Windows 功能等可能需要系統管理員權限或造成持續性的系統變更。若現有授權未涵蓋該變更，先完成其他工作，再向使用者說明確切缺少項目、用途及影響後取得授權。
- 若下載、授權、費用、網路政策或作業系統不相容使安裝無法完成，記錄實際錯誤與已嘗試的方法，提供對應環境的人工安裝與驗證步驟；不要改用無關工具假裝完成，也不要聲稱未執行的驗證已通過。
- 不為了執行 skill 自身的文字格式檢查而污染目標專案環境。這類輔助驗證套件也應使用現有工具或獨立的暫時環境；若缺少，依前述規則補齊或清楚記錄替代驗證方式。

## 先確定發布方式

在製作正式套件前，確認下列事項：

1. 這是第一次提交，還是既有 Store 產品的更新。
2. 目標是 Store 託管的 MSIX，而不是由開發者網站提供的傳統 EXE／MSI 安裝程式。
3. 目標處理器架構，例如 x64、x86、ARM64；需要多種架構時，評估 MSIX bundle。
4. 最低 Windows 版本及實際要支援的螢幕、印表機、周邊設備與公司環境。
5. 程式需要的執行階段、DLL、圖片、字型、設定範本及其他資源。
6. 程式資料應存放的位置，以及更新或移除程式時資料應如何處理。

Store 託管的 MSIX 與另行散布的 EXE 是兩個不同發布管道。Microsoft Store 通過審核後會重新簽署 Store 配送的 MSIX；它不會順便替網站、雲端硬碟或郵件中的原始 EXE 建立信任。

## 私人對象的必要決策

提交前重新查閱 Microsoft 官方的私人對象規則，不要只依賴舊記憶或舊畫面。現行規則中下列事項會直接影響設計與提交：

- 若產品要限制給指定人員取得，第一次提交就選擇 **Private audience／私人對象**。已經以公開對象提交的產品，可能無法在後續提交改回私人對象；私人對象則可在日後另行改為公開。
- 「無法搜尋、僅限直接連結」不是存取控制。真正限制取得者時要使用私人對象。
- 目前私人對象的取得者需要使用名單中對應的個人 Microsoft 帳號登入 Store；工作或學校的 Microsoft Entra ID 帳號目前不等同於這種取得資格。正式操作前再次確認規則是否更新。
- 私人對象只限制誰能查看與取得 Store 套件，不會替應用程式內部資料提供登入、授權、加密或離職停用機制。
- 從私人對象名單移除曾安裝程式的人，不代表已安裝的程式會被遠端刪除；必須另外設計公司資料與帳號的存取控制。
- 初次發布後，可用 package flights 將更新版本先提供給私人對象中的一小組測試者；flight 仍會經過 Store 認證流程。

若使用者尚未建立 Partner Center 產品，提醒他在最終封裝前保留產品名稱並取得 Store identity。不要猜測或自行創造正式的 Identity、Publisher 或 Application Id。

## 建置與封裝

### 1. 產生發布用執行檔

- 使用專案既有且可重現的 release 建置方式。
- 先執行與變更相稱的單元、整合及功能測試。
- 直接啟動發布用 EXE，確認核心流程正常。
- 盤點動態 DLL、語言執行階段及外部檔案。Python 類應用尤其要確認直譯器、原生擴充及資料檔案已包含；Rust 也不能假設所有依賴都一定靜態連結。
- 不把除錯資料、開發用憑證私鑰、測試帳密、真實客戶資料、暫存檔或機器專屬絕對路徑包進正式套件。

### 2. 選擇封裝方式

優先使用可從確定的輸出目錄重現套件內容的封裝方式。若專案已有 MSIX 專案或 manifest，沿用並修正它。若現有安裝程式會進行複雜的檔案、登錄或服務修改，才評估以乾淨虛擬機器使用 MSIX Packaging Tool 捕捉安裝；捕捉時避免把 Windows Update、其他程式或個人檔案的變更混入套件。

不要因工具較熟悉而改寫使用者選定的語言、GUI 框架或應用程式架構。

### 3. 檢查 manifest 與 Store identity

至少核對：

- 套件 Identity、Publisher、版本及 ProcessorArchitecture。
- Application Id、Executable 與啟動入口是否指向正確檔案。
- 顯示名稱、發行者顯示名稱、說明及圖示資產。
- TargetDeviceFamily、最低 Windows 版本及已測試版本。
- capabilities、檔案關聯、通訊協定、啟動工作或其他 extensions 是否真正需要。
- 只宣告實際需要的能力；受限制能力或外部驅動程式／服務依賴必須另行確認政策並在 certification notes 說明。

更新既有 Store 應用程式時，必須沿用 Partner Center 中的正式 identity、Publisher 與 Application Id。套件版本要高於已發布或已部署版本，並遵守目前 Partner Center 對四段版本號的規則。不要為了讓安裝通過而改成另一個 identity，否則 Windows 會把它視為另一個應用程式，舊版也無法正常更新。

### 4. 組合套件內容

- 包含啟動 EXE、必要 DLL、執行階段、Slint／其他 GUI 資源、字型、圖片、授權文字及 manifest 宣告的所有資產。
- 讓程式使用明確且適當的資源路徑。不要依賴「目前工作目錄等於 EXE 所在目錄」。
- MSIX 安裝目錄是唯讀的。設定、紀錄、快取、暫存檔及使用者資料不得寫在程式安裝目錄。
- 對公司共用資料夾、網路磁碟、使用者選取的輸入／輸出位置及外部程式，確認封裝後仍有合理的存取方式與錯誤訊息。
- 多架構套件要確認各架構內容一致且只包含相符的執行檔與相依元件。

### 5. 區分測試簽章與 Store 簽章

- 真正安裝 `.msix` 檔前，套件必須有有效簽章，而且測試電腦必須信任該簽章。
- 本機開發可以使用自簽測試憑證，但憑證 Subject 必須與 manifest 的 Publisher 完全一致。
- 自簽憑證只適合受控測試電腦。不要把它描述成公開信任或 SmartScreen 信譽，也不要要求一般使用者把它加入受信任的根憑證。
- 若需要讓測試電腦信任自簽憑證，使用正確的 **Trusted People** 範圍；不要把非根 CA 的測試憑證匯入 Trusted Root Certification Authorities。
- 匯入信任憑證會改變電腦的信任狀態。只有已有授權且能確認憑證來源時才代為執行；否則提供人工步驟並說明影響。
- Store 提交套件依目前官方流程準備。Store 通過審核後會替 MSIX/AppX 重新簽署；本機測試憑證不是 Store 最終簽章。
- 絕不提交、分享或納入版本庫的憑證私鑰與密碼。

## 三層測試迴圈

### A. 直接 EXE 快速測試

每次修改程式邏輯或 GUI 後，先重新編譯並直接執行 EXE。檢查啟動、主要流程、錯誤處理及介面排版。這一層速度最快，但不能證明 MSIX 安裝後正常。

### B. 展開套件或封裝專案偵錯

如果現有工具支援，使用封裝專案的開發部署／F5，或註冊展開的套件內容，讓程式以套件 identity 執行。需要程式層偵錯時，把適合該語言的原生偵錯器連接到套件內的實際處理程序；若程式在啟動前期就失敗，使用 Windows SDK 的套件生命週期偵錯能力。

這一層用來快速找出啟動入口、套件 identity、資源路徑及封裝情境問題，不等同於完整安裝測試。

### C. 已簽署 MSIX 的端到端安裝測試

使用測試簽章建立可安裝的 MSIX，在測試電腦或乾淨虛擬機器上實際安裝，再從開始功能表啟動。至少測試：

#### 安裝與身分

- 標準使用者能否依預期安裝；只有確實需要時才要求系統管理員權限。
- 安裝畫面顯示的名稱、發行者、版本及圖示是否正確。
- 開始功能表捷徑、啟動入口與解除安裝項目是否正確。
- 套件簽章有效，檔案雜湊與交付檔一致。

#### 核心功能

- 用小型、可回復的測試資料走完一個完整工作流程。
- 測試開啟、建立、修改、儲存、匯出、列印及關閉後重開；只測程式實際具備的功能。
- 測試取消、無效輸入、檔案被占用、目的地不可寫、網路中斷及外部資源不存在時的反應。
- 程式若依賴內網或伺服器，離線時不可無說明地當機。

#### 資源、路徑與資料

- 圖片、字型、翻譯、範本及 DLL 都能從已安裝套件載入。
- 程式未嘗試寫入唯讀的 MSIX 安裝目錄。
- 使用者設定、紀錄、快取及輸出寫到合理且可存取的位置。
- 相對路徑、目前工作目錄與直接執行 EXE 時不同仍可正常運作。
- 更新程式後，既有設定與資料能保留或正確遷移。

#### GUI 與可用性

- 在實際支援的解析度與 Windows 顯示縮放比例測試，至少涵蓋常見的 100%、125%、150%；若軟體會在高 DPI 或小螢幕使用，再測更高比例與最低支援解析度。
- 檢查文字截斷、控制項重疊、視窗超出螢幕、對話框被遮住、鍵盤焦點、Tab 順序及必要的鍵盤操作。
- 多螢幕或不同 DPI 間移動視窗時，確認縮放及位置合理。
- Slint 或其他 GUI 直接執行正常，不代表封裝後資源與 DPI 行為一定相同；保留安裝版畫面或明確人工驗收紀錄。

#### 公司環境與硬體

- 實測公司共用資料夾、網路磁碟、檔案權限、印表機、掃描器、USB 裝置及其他實際使用的周邊。
- 實體印表機、掃描器或受管理網路的成功，不能用一般建置測試代替。
- 若 Agent 無法接觸這些環境，清楚標記為未測，並啟動下方的人工對話測試流程。

#### 更新與回復

- 保留舊版，從目前實際部署版本直接安裝候選更新版。
- 確認 identity 與 Publisher 未改變、版本號提高、更新不會被當成另一套程式。
- 更新後重測資料、設定、捷徑、檔案關聯及核心工作流程。
- 若產品允許降版或回復，另外驗證新資料格式是否能被舊版安全讀取；不能安全降版時要明確阻止或說明。
- 另做一次乾淨安裝，避免開發電腦既有的 DLL、憑證或設定掩蓋缺漏。

## 診斷順序

先比較「同一個發布 EXE 直接執行」與「MSIX 安裝後執行」：

- 兩者都失敗：優先檢查程式邏輯、GUI、資料或原生相依元件。
- 只有 MSIX 失敗：優先檢查 manifest、套件 identity、簽章、資源是否漏包、安裝目錄唯讀、目前工作目錄、檔案／登錄虛擬化、capabilities 及架構相符性。
- MSIX 無法安裝：記錄完整錯誤碼，檢查簽章信任、Publisher 是否完全一致、版本是否降級、套件是否毀損、架構及相依套件。
- 可以安裝但無法啟動：檢查 Executable、Application Id、入口點、缺少 DLL 及啟動即當機。

安裝或更新錯誤優先查看事件檢視器中 `Applications and Services Logs > Microsoft > Windows > AppxDeployment-Server` 的相關事件；封裝資料錯誤再查看 AppxPackagingOM。執行期錯誤使用該語言適合的偵錯器、Windows Error Reporting、應用程式事件及程式自身紀錄。紀錄檔必須寫在使用者可寫位置，不能寫入套件根目錄。

修正後重新建置 EXE、重建 MSIX、重新簽署、提高測試版本並安裝更新，再重跑受影響測試。不要透過關閉 SmartScreen、停用防毒或降低整台電腦的安全政策來讓測試表面通過。

## Store 審核準備

在宣稱「可提交 Store」前完成：

1. 使用目前版本的 Windows App Certification Kit 測試候選套件或已安裝應用程式，保存完整報告。通過 WACK 只是送審前檢查，不保證 Store 一定核准。
2. 查閱當下有效的 Microsoft Store Policies、MSIX package requirements 及 restricted capabilities 規則。
3. 確認產品名稱、說明、畫面截圖、功能、價格、支援資訊及系統需求與實際版本一致。
4. 判斷程式是否存取、收集或傳輸個人資訊；需要時提供有效且與實際行為一致的隱私權政策。
5. 若程式需要登入、內網、伺服器、隱藏功能、外部驅動程式或特殊硬體，在 Partner Center 的 certification notes 提供審核者可完成測試的簡短步驟。
6. 需要登入時提供專用、可撤銷、權限最小的測試帳號，不要提供正式管理者或員工帳號。帳密只放在適當的審核欄位，不寫進套件或公開商店文字。
7. 審核期間必須讓必要伺服器保持可用。若程式只能在 Microsoft 無法進入的公司內網工作，明確告知使用者這可能導致「產品無法測試」而審核失敗；在未經同意前不要擅自新增後門、公開公司服務或改變安全架構。
8. 第一次提交時確認私人對象與已核對的測試者 Microsoft 帳號名單。不要用「僅限連結」代替私人對象。

## Agent 無法親自安裝或操作時

若缺少系統權限、GUI、實體硬體、公司網路、乾淨測試機或適當 Windows 版本，仍要先完成能完成的建置、靜態檢查、簽章檢查、雜湊及 WACK 準備。接著用對話引導使用者實測，不要只丟出一大串指令。

採用以下方式：

1. 先說明哪一項尚未實測、缺少什麼條件，以及這會限制哪個結論。
2. 一次只請使用者進行一個小步驟，使用符合其語言與程度的說法。
3. 每一步先描述預期畫面與正常結果，再請使用者回報實際結果、完整錯誤文字或截圖。
4. 收到結果後判定通過、失敗或資訊不足，記錄下來，再給下一步。
5. 先使用測試資料；涉及真實公司資料、批次列印、覆寫檔案或不可回復操作前要停下來提醒。
6. 不要求使用者停用 Windows 安全功能。涉及匯入測試憑證時，先核對憑證來源、發行者及用途，並說明它只信任由該憑證簽署的測試套件。

建議的人工對話順序：

- 核對 MSIX 檔名、版本、架構、大小及 SHA-256。
- 開啟安裝畫面，請使用者確認顯示名稱、發行者及圖示；尚未按安裝前先回報異常。
- 安裝後由開始功能表啟動，確認視窗能完整出現且沒有立即錯誤。
- 用最小測試資料完成核心流程，逐項確認輸入、輸出、儲存與錯誤提示。
- 關閉並重開，確認設定與資料是否如預期保留。
- 測試實際解析度、縮放比例、印表機及公司共用資源。
- 保留舊版的測試環境並安裝新版，確認更新、資料保留及版本顯示。
- 最後測試解除安裝；是否刪除工作資料必須符合產品設計且事先說明。

使用者回報的人工結果要標為「使用者在指定電腦確認」，不能改寫成 Agent 親自測試。若只完成編譯或套件驗證，不能聲稱 GUI、列印、硬體或公司環境已驗證。

## 更新既有版本

- 先取得目前發布與實際部署版本的 identity、Publisher、Application Id、架構、版本號及更新來源。
- 保留版本相容性；先測「舊版直接升級至候選版」，再測乾淨安裝。
- 更新後重新執行受影響的功能測試、完整 MSIX 安裝測試及 WACK。
- 若 capabilities、隱私行為、登入方式、硬體需求、支援網址或商店說明改變，同步更新 Partner Center 資料及 certification notes。
- 可先透過私人對象中的 package flight 提供給少數測試者；它仍需送交認證，不能取代本機測試。

## 完成交付標準

最終回報至少包含：

- EXE、MSIX／MSIX bundle／MSIX upload 及 WACK 報告的完整路徑；只列實際產生的檔案。
- 應用程式版本、套件 identity、Publisher、架構、最低 Windows 版本及封裝方式。
- 每個交付檔的 SHA-256。
- 簽章狀態：未簽、開發測試簽章、公開信任簽章或已由 Store 配送。未經 Store 下載驗證，不得稱為 Store 已簽署版本。
- 直接 EXE、本機封裝偵錯、已安裝 MSIX、升級、乾淨安裝、不同 DPI、硬體／內網及 WACK 的結果矩陣。
- 明確列出沒有權限或環境而未測的項目，以及使用者可以接續的人工測試步驟。
- 已知限制、Store 審核風險、需要準備的 certification notes、私人對象名單及隱私／支援資料。

只有在候選 MSIX 已成功產生、簽章狀態清楚、能安裝啟動、核心流程完成，而且所有未測事項均明確揭露時，才能稱為「可供 Store 提交的候選版本」。只有 Partner Center 通過認證並實際配送後，才能稱為「已在 Store 發布」。

## 官方資料

政策、費用、帳號資格與工具會變動。需要提交、購買或變更帳號前，重新查閱 Microsoft 官方資料：

- [Code signing options for Windows app developers](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/code-signing-options)
- [App package requirements for MSIX app](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/app-package-requirements)
- [Choose visibility options for MSIX apps](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/visibility-options)
- [Run, debug, and test an MSIX package](https://learn.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-debug)
- [Understanding how packaged desktop apps run on Windows](https://learn.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-behind-the-scenes)
- [MSIX troubleshooting guide](https://learn.microsoft.com/en-us/windows/msix/msix-troubleshooting-guide)
- [Windows App Certification Kit](https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/windows-app-certification-kit)
- [Microsoft Store Policies](https://learn.microsoft.com/en-us/windows/apps/publish/store-policies)
- [Manage submission options for MSIX apps](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/manage-submission-options)
- [Package flights](https://learn.microsoft.com/en-us/windows/apps/publish/package-flights)
