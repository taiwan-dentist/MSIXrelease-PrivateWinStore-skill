# Windows MSIX Private Store Release Skill

這是一個供 Codex 與支援 `SKILL.md` 的 Agent 使用的發布工作流程，協助將 Windows 桌面程式的 EXE 整理成可提交 Microsoft Store「私人對象」的 MSIX，並要求在交付前完成可核對的安裝、執行、更新及審核準備。

它不限定程式語言或 GUI 框架，可用於 Rust／Slint、Python、C++、C#、Qt、Tauri、Electron 及其他能產生 Windows 桌面執行檔的專案。

## 這個 skill 解決什麼

- 在 Windows 軟體正式交付或更新時，同時考慮 Store 用 MSIX，不只輸出 EXE。
- 保留 Partner Center 的套件 identity、Publisher、Application Id 與版本延續性。
- 在第一次提交時提醒選擇 **Private audience／私人對象**，避免誤用「僅限連結」。
- 區分開發測試憑證、公開信任簽章與 Microsoft Store 最終簽章。
- 分別驗證直接執行 EXE、套件情境偵錯及完整安裝 MSIX。
- 檢查 MSIX 唯讀安裝目錄、資源路徑、DLL、字型、資料保存、DPI、內網及硬體。
- 驗證舊版升級、乾淨安裝、Windows App Certification Kit 與 Store 審核需求。
- Agent 沒有 GUI、系統權限或實體設備時，改用逐步對話帶領使用者人工驗收，並忠實標示未測項目。
- 需要的語言套件、測試套件或驗證工具缺少時，優先在專案隔離環境中安裝並實際執行驗證，不因語言不同而略過。

## 安裝

將整個倉庫放進 Agent 使用的 skills 目錄，並保留根目錄的 `SKILL.md`。

Codex 在 Windows 上常見的使用者層級位置是：

```text
C:\Users\你的帳號\.agents\skills\windows-msix-private-store-release\SKILL.md
```

不同 Agent 或不同版本可能使用其他 skills 目錄，請依該產品當時的官方文件為準。複製後重新啟動 Agent 或建立新工作，確認 `windows-msix-private-store-release` 出現在可用 skills 清單。

## 使用方式

可以直接指定：

```text
$windows-msix-private-store-release
```

也可以提出符合其描述的發布要求，例如：

- 「把這個 Rust／Slint 程式完成後，同時產生可供 Microsoft Store 私人對象提交的 MSIX。」
- 「這是 Python GUI 軟體的新版本，請保留 Store identity、測試升級並輸出 MSIX。」
- 「檢查這個 Windows App 是否符合私人 Store 上架及本機安裝測試要求。」

## 重要原則

「編譯成功」只證明建置完成，不代表 MSIX 已經能安裝或程式能在封裝環境正常工作。本 skill 要求 Agent 清楚區分：

1. 發布用 EXE 的直接執行測試。
2. 具有套件 identity 的開發偵錯。
3. 已簽署 MSIX 的安裝、啟動與更新測試。
4. Windows App Certification Kit 的檢查。
5. Microsoft Store 實際通過認證及配送。

只有完成前三項核心驗證並揭露所有未測事項後，才能稱為「可供 Store 提交的候選版本」。只有 Partner Center 通過認證並實際配送後，才能稱為「已在 Store 發布」。

## 私人對象不是程式內部權限控制

Microsoft Store 的私人對象限制誰能查看及取得套件，但不會自動提供：

- 公司帳號登入與離職停用。
- 應用程式資料權限。
- 機密資料加密。
- 已安裝程式的遠端移除。

需要保護公司資料時，應用程式仍要有自己的授權與資料安全設計。GitHub 倉庫是公開的，也不會使未來 Store 上架的私人對象設定變成公開。

## 工具與依賴

此倉庫不綁定特定語言套件或 MSIX 製作工具。Agent 應先讀取目標專案的原始碼、鎖定檔及建置文件，再選擇相符的編譯、封裝與驗證方式。

若缺少必要工具，應優先使用專案隔離環境並遵守既有套件管理方式。需要系統管理員權限、Windows SDK／Build Tools、憑證信任、驅動程式或其他持續性系統變更時，必須依當次工作的授權範圍處理。

憑證私鑰、PFX 密碼、Partner Center 帳密、公司資料及審核測試帳密不得提交到這個或任何公開倉庫。

## 官方資料

Store 政策、費用、帳號資格和工具會改變，實際提交前應重新查閱 Microsoft 官方文件：

- [Code signing options for Windows app developers](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/code-signing-options)
- [App package requirements for MSIX app](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/app-package-requirements)
- [Choose visibility options for MSIX apps](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/visibility-options)
- [Run, debug, and test an MSIX package](https://learn.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-debug)
- [MSIX troubleshooting guide](https://learn.microsoft.com/en-us/windows/msix/msix-troubleshooting-guide)
- [Windows App Certification Kit](https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/windows-app-certification-kit)
- [Microsoft Store Policies](https://learn.microsoft.com/en-us/windows/apps/publish/store-policies)
- [Package flights](https://learn.microsoft.com/en-us/windows/apps/publish/package-flights)

完整執行規則請閱讀 [`SKILL.md`](SKILL.md)。
