# Windows MSIX Private Store Release Skill

這個 skill 給 Codex 與支援 `SKILL.md` 的 Agent 使用。它會在 Windows 軟體發布時，把 EXE 封裝成可提交至 Microsoft Store「私人對象」的 MSIX，並在交付前檢查安裝、執行、更新與送審準備。

程式語言與 GUI 框架不限。只要能產生 Windows 桌面執行檔，Rust／Slint、Python、C++、C#、Qt、Tauri、Electron 等專案都可以使用。

## 這個 skill 會做什麼

- Windows 軟體正式交付或更新時，除了 EXE，也準備 Store 使用的 MSIX。
- 沿用 Partner Center 裡既有的套件 identity、Publisher、Application Id 與版本資料。
- 在第一次提交時提醒選擇 **Private audience／私人對象**，避免誤用「僅限連結」。
- 區分開發測試憑證、公開信任簽章與 Microsoft Store 最終簽章。
- 分開測試直接執行的 EXE、套件環境中的偵錯，以及實際安裝後的 MSIX。
- 檢查 MSIX 唯讀安裝目錄、資源路徑、DLL、字型、資料保存、DPI、內網及硬體。
- 測試舊版升級與乾淨安裝，並檢查 Windows App Certification Kit 和 Store 的送審要求。
- 若 Agent 無法操作 GUI、取得系統權限或接觸實體設備，就逐步請使用者代為測試，並如實標明哪些項目尚未測試。
- 缺少必要的語言套件、測試套件或驗證工具時，先在專案隔離環境中安裝，再實際執行驗證。不能因專案不是 Rust 或 Python 就跳過。

## 安裝

把整個儲存庫放進 Agent 的 skills 目錄，並保留根目錄下的 `SKILL.md`。

Codex 在 Windows 上常見的使用者層級位置是：

```text
C:\Users\你的帳號\.agents\skills\windows-msix-private-store-release\SKILL.md
```

其他 Agent 或版本使用的 skills 目錄可能不同，安裝前請查看該產品的官方文件。放好後重新啟動 Agent，或開一個新工作，確認可用 skills 清單中有 `windows-msix-private-store-release`。

## 使用方式

可以直接指定：

```text
$windows-msix-private-store-release
```

也可以直接描述發布需求，例如：

- 「把這個 Rust／Slint 程式完成後，同時產生可供 Microsoft Store 私人對象提交的 MSIX。」
- 「這是 Python GUI 軟體的新版本，請保留 Store identity、測試升級並輸出 MSIX。」
- 「檢查這個 Windows App 是否符合 Microsoft Store 私人對象的上架與本機安裝測試要求。」

## 重要原則

「編譯成功」只證明建置完成，還不能證明 MSIX 可以安裝，也不能證明程式能在封裝環境中正常執行。Agent 必須分開檢查以下五件事：

1. 發布用 EXE 的直接執行測試。
2. 具有套件 identity 的開發偵錯。
3. 已簽署 MSIX 的安裝、啟動與更新測試。
4. Windows App Certification Kit 的檢查。
5. Microsoft Store 實際通過認證及配送。

完成前三項核心驗證，並列清楚所有未測事項後，才能把它稱為「可供 Store 提交的候選版本」。等 Partner Center 通過認證並開始實際配送，才算「已在 Store 發布」。

## 私人對象只控制 Store 的取得資格

Microsoft Store 的私人對象限制誰能查看及取得套件，但不會自動提供：

- 公司帳號登入與離職停用。
- 應用程式資料權限。
- 機密資料加密。
- 已安裝程式的遠端移除。

公司資料仍要由應用程式自己的登入、授權與資料安全設計保護。這個 GitHub 儲存庫雖然公開，未來在 Microsoft Store 設定的私人對象仍可維持私人。

## 工具與依賴

這個 skill 不綁定特定語言套件或 MSIX 製作工具。Agent 會先讀取目標專案的原始碼、鎖定檔與建置文件，再選擇相符的編譯、封裝和驗證方式。

缺少必要工具時，優先裝在專案隔離環境，並沿用專案既有的套件管理方式。若需要系統管理員權限、Windows SDK／Build Tools、憑證信任、驅動程式或其他會長期影響系統的變更，Agent 要先確認當次工作是否已取得相應授權。

不要把憑證私鑰、PFX 密碼、Partner Center 帳密、公司資料或審核用的測試帳密提交到任何公開儲存庫。

## 官方資料

Store 政策、費用、帳號資格和工具可能調整。正式提交前，請重新核對 Microsoft 官方文件：

- [Code signing options for Windows app developers](https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/code-signing-options)
- [App package requirements for MSIX app](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/app-package-requirements)
- [Choose visibility options for MSIX apps](https://learn.microsoft.com/en-us/windows/apps/publish/publish-your-app/msix/visibility-options)
- [Run, debug, and test an MSIX package](https://learn.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-debug)
- [MSIX troubleshooting guide](https://learn.microsoft.com/en-us/windows/msix/msix-troubleshooting-guide)
- [Windows App Certification Kit](https://learn.microsoft.com/en-us/windows/uwp/debug-test-perf/windows-app-certification-kit)
- [Microsoft Store Policies](https://learn.microsoft.com/en-us/windows/apps/publish/store-policies)
- [Package flights](https://learn.microsoft.com/en-us/windows/apps/publish/package-flights)

完整規則都寫在 [`SKILL.md`](SKILL.md)。
