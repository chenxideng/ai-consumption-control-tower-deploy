---
title: AI Consumption Control Tower 安裝指南
description: 使用 Azure VM 內建 DNS 及自動 HTTPS，從公開通用編譯包安裝，不需要自購網域或網站 SSL 證書。
---

## 先準備

* 一台有公網 IP 的 Ubuntu 22.04 或 24.04 x64 VM，建議至少 2 vCPU、4 GB RAM
* 可以使用 SSH 及 `sudo` 的管理帳號
* Azure NSG 及 VM 防火牆允許公網 TCP 80、443；SSH 22 只開放給管理 IP
* VM 可連線 Ubuntu／Caddy 官方套件、HTTPS 憑證服務及提供者授權網址
* `/tmp` 允許執行校驗後的編譯安裝工具，不能掛載為 `noexec`

建議使用新 VM。安裝器不會覆蓋其他程式的端口或未受管理的既有安裝。
不要公開後端端口 8096。

## 第一步：設定 Azure 內建網址

不需要購買網域或建立 Azure DNS zone。

1. Azure Portal → VM → Properties，按 Public IP address／DNS name label 的公網 IP 連結。
2. 在公網 IP 設定中填寫可用的 DNS name label，按 Save。
3. 返回 VM Overview，複製 Azure 顯示的完整 DNS name。

例如 `your-company-aict.eastasia.cloudapp.azure.com`。
不同區域和 DNS name label scope 可能有不同格式，直接複製 Azure
傳回的完整值，不要自行拼接。不要使用範例名稱或只填公網 IP。

VM、公網 IP 和流量仍依 Azure 正常價格收費。
安裝器不會替你更改 NSG、DNS label、雲端權限或建立資源。

## 第二步：提交公司申請並立即下載

在[提供者首頁](https://charlesdeng.eastasia.cloudapp.azure.com/ai-finops/)
選擇英文／繁體／簡體，填寫公司、所在地、AI 場景、預計開始時間及聯絡資料。
提交後立即下載公司專屬 ZIP，不需等待審批。ZIP 包含：

* `ai-consumption-control-tower-v1.5.0-linux-x64.run`
* `install-guide.txt`，與下載時頁面語言一致
* `company-bootstrap.json`，你的安裝身分及初始語言
* `superadmin-credentials.txt`，公司獨立的初始帳密
* `activation-code.txt`，32 字元啟用碼

只使用實際 ZIP 中的版本及檔名，不要混用不同公司的檔案。
每個公司包都有獨立帳密及啟用碼，不會放在公開 GitHub Release。
私密下載自提交起七天有效；啟用綁定後不再提供初始帳密下載。

把 ZIP 解壓至 VM 的私人目錄，透過 SSH 進入該目錄。
不需要把 GitHub Token、源碼或任何其他公司的檔案放到 VM。

## 第三步：校驗並執行一個安裝命令

以下在 VM 的 SSH 終端執行，不是在 Windows PowerShell 執行：

```bash
sudo bash ai-consumption-control-tower-v1.5.0-linux-x64.run \
  --bootstrap company-bootstrap.json
```

使用 ZIP 內實際的 `.run` 檔名。安裝器會驗證內嵌 SHA-256 及每個部署檔案；
校驗失敗請從原私密連結重新下載，不要繞過。

安裝時依提示：

1. 貼上 Azure VM 的完整 DNS name，不加 `https://`、端口或路徑。
2. 等待安裝完成，開啟輸出的 HTTPS 網址；不必在命令列輸入密碼或啟用碼。

Caddy 會自動申請和續期瀏覽器信任的網站 SSL 證書。
不用購買或上傳證書，也不用安裝根證書或略過瀏覽器安全警告。
請持續保留 TCP 80、443 可達，以便網站存取及證書驗證。

安裝器會設定必要套件、獨立服務帳號、受保護的 SQLite、
開機自動啟動和只監聽 loopback 的後端。啟動或驗證失敗時會回滾，
不會以測試檔案代替真實安裝包。

## 首次登入

1. 開啟安裝器輸出的網址，登入頁是 `/ai-finops/login/`。
2. 使用 `superadmin-credentials.txt` 的帳密登入，按要求更改密碼。
3. 在安裝恢復頁或 Admin → Installation 輸入 `activation-code.txt` 的啟用碼。
4. 成功連網驗證後可使用儀表板，並在 Admin → Users & roles 管理本機權限。

新申請的授權預設啟用，但提供者可隨時停用或重新啟用。
Demo 數字不是客戶 Azure 用量；Azure Live、Fabric 及告警資料庫
必須另外配置及授權。應用身分憑證與網站 SSL 是不同事項，
前者不是安裝 Demo 的先決條件。

## 強制連網授權與選用回傳

每 60 秒驗證一次授權，最長有效租期 90 秒。回應由提供者簽署並綁定
每次隨機挑戰，不能重播舊回應續期，重啟 VM 也不能恢復離線使用。
授權被停用通常於下次檢查鎖定；連不到授權服務且無法續期，
最遲於有效租期結束時鎖定，阻止受保護 API、Dashboard 及新的背景監控讀取。
只保留登入、改密碼及安裝恢復入口，提示重新申請並提供更多證明。

必要驗證只包含安裝身分、隨機實例識別、隨機驗證挑戰及版本。
每五分鐘的運行使用回傳為另一個選用功能，可關閉；
關閉回傳不會停止必要的授權驗證。授權鎖定不會關閉 Azure 工作負載或停止計費。
不要在同一安裝上混用不同公司的引導檔。

這不是不可破解的 DRM，不能保證阻止管理員刻意修改程式或複製已登記資料庫。
舊版客戶包須升級後，才可依賴遠端鎖定。

## 公開通用包與更新

[Releases](https://github.com/chenxideng/ai-consumption-control-tower-deploy/releases)
只提供已驗證通用 `.run` 及 `SHA256SUMS`，沒有公司帳密、啟用碼或 bootstrap。
通用包供已登記安裝更新使用；沒有公司引導及有效授權時不能使用私人模組。
下載同一版本的兩個檔案，在既有安裝執行：

```bash
sha256sum --check SHA256SUMS &&
  sudo bash ai-consumption-control-tower-v1.5.0-linux-x64.run
```

使用實際版本的檔名。已管理安裝會保留原本的公司身分、帳戶及密碼。

## 驗證、更新與備份

在 VM 執行：

```bash
sudo systemctl status ai-consumption-control-tower.service --no-pager
```

再用瀏覽器確認 HTTPS 沒有安全警告、登入及儀表板可用。
健康檢查在站點的 `/ai-finops/healthz`。
`database: not_configured` 表示 Fabric 未配置，不是 Fabric 健康驗證成功；
本機帳戶 SQLite 則看 `controlPlane.database`。

更新前，先停止本服務並一致地備份以下資料，再啟動服務：

* `/var/lib/ai-consumption-control-tower/` 整個狀態目錄，包含 SQLite 與匹配的 `.key`
* `/etc/ai-consumption-control-tower.env`

備份含敏感資料，不可上傳 GitHub 或公開下載目錄。
下載新版本並重新校驗、安裝，使用原來的 DNS name。
已管理安裝會保留帳戶、已更改密碼及公司身分，不重新發出共用密碼。
若需回滾，使用安裝輸出的完整 `sudo aict-deploy --rollback …` 命令。

## 常見問題

| 問題 | 處理 |
|------|------|
| 沒有 Release 安裝包 | 尚無已驗證可交付版本，不把 GitHub「Source code」壓縮檔當安裝包 |
| DNS 不解析 | 確認 Azure DNS name label 已儲存，複製完整 DNS name 並等待生效 |
| HTTPS 證書失敗 | 檢查 DNS、TCP 80／443、NSG、防火牆及對外連線，不略過證書錯誤 |
| 端口或既有安裝衝突 | 使用新 VM，或由管理員先規劃既有反向代理，不強制覆蓋 |
| 舊申請仍待處理 | 透過新的即時下載流程重新申請；舊的拒絕或撤銷紀錄不會自動啟用 |
| 授權未啟用 | 依頁面指示重新申請並提供更多證明，或聯絡提供者重新啟用 |
| 授權服務失聯 | 檢查提供者網址及對外 HTTPS；恢復連線後在安裝恢復頁重新驗證 |
| Live 沒有資料 | 配置客戶自己的 Entra、Azure／Fabric 權限，不以 Demo 數字代替 |

Azure 操作參考：[設定 VM 內建 DNS 名稱](https://learn.microsoft.com/en-us/azure/virtual-machines/create-fqdn)。
HTTPS 原理：[Caddy 自動 HTTPS](https://caddyserver.com/docs/automatic-https)。
