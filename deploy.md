---
title: AI Consumption Control Tower 安裝指南
description: 使用 Azure VM 內建 DNS 及自動 HTTPS，從公開通用編譯包安裝，不需要自購網域或網站 SSL 證書。
---

## 先準備

* 一台有公網 IP 的 Ubuntu 22.04 或 24.04 x64 VM，建議至少 2 vCPU、4 GB RAM
* 可以使用 SSH 及 `sudo` 的管理帳號
* Azure NSG 及 VM 防火牆允許公網 TCP 80、443；SSH 22 只開放給管理 IP
* VM 可下載 Ubuntu 套件並向 HTTPS 憑證服務連線
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

## 第二步：下載同一版本的兩個檔案

在 [Releases](https://github.com/chenxideng/ai-consumption-control-tower-deploy/releases)
選擇已發布版本，下載：

* `ai-consumption-control-tower-v1.1.0-linux-x64.run`
* `SHA256SUMS`

上面以 `v1.1.0` 為例。只使用實際 Release 中列出的檔案，
不同版本請使用對應版本的檔名及校驗檔，不要混用。
沒有 `.run` 和校驗檔的版本不可部署。

把兩個檔案放進 VM 同一個私人目錄，透過 SSH 進入該目錄。
不需要把 GitHub Token、源碼或任何其他公司的檔案放到 VM。

## 第三步：校驗並執行一個安裝命令

以下在 VM 的 SSH 終端執行，不是在 Windows PowerShell 執行：

```bash
sha256sum --check SHA256SUMS &&
  sudo bash ai-consumption-control-tower-v1.1.0-linux-x64.run
```

使用實際下載的 `.run` 檔名。只有 SHA-256 顯示 `OK` 才會執行安裝；
校驗失敗請重新下載，不要繞過。

安裝時依提示：

1. 貼上 Azure VM 的完整 DNS name，不加 `https://`、端口或路徑。
2. 為 `superadmin` 設定至少 12 個 UTF-8 位元組的初始密碼，並確認一次。
3. 等待安裝完成，開啟輸出的 HTTPS 網址。

Caddy 會自動申請和續期瀏覽器信任的網站 SSL 證書。
不用購買或上傳證書，也不用安裝根證書或略過瀏覽器安全警告。
請持續保留 TCP 80、443 可達，以便網站存取及證書驗證。

安裝器會設定必要套件、獨立服務帳號、受保護的 SQLite、
開機自動啟動和只監聽 loopback 的後端。啟動或驗證失敗時會回滾，
不會以測試檔案代替真實安裝包。

## 首次登入

1. 開啟安裝器輸出的網址，登入頁是 `/ai-finops/login/`。
2. 用 `superadmin` 及剛才設定的密碼登入。
3. 按要求更改密碼，再使用儀表板。
4. 在 Admin → Users & roles 建立本機使用者、角色和模組權限。

通用包沒有預設共用帳密，不需要公司啟用碼即可使用 Synthetic Demo。
Demo 數字不是客戶 Azure 用量；Azure Live、Fabric 及告警資料庫
必須另外配置及授權。應用身分憑證與網站 SSL 是不同事項，
前者不是安裝 Demo 的先決條件。

## 已獲批准的公司專屬 ZIP

公司專屬帳密、啟用碼及 `company-bootstrap.json` 不會放在這個公開倉庫。
請先向提供者申請，由管理員批准後，使用私密連結下載公司 ZIP。
下載和首次啟用有效期由審批通過起計七天。

公司版應使用該私密 ZIP 裡的 `.run` 及安裝引導，在全新 VM 執行：

```bash
sudo bash ai-consumption-control-tower-v1.1.0-linux-x64.run \
  --bootstrap company-bootstrap.json
```

使用 ZIP 內實際版本的檔名。登入資料和啟用碼只取自該公司 ZIP，
首次更改密碼後，到 Admin → Installation 啟用並確認回傳開關。
回傳須明示同意，可以關閉；通用安裝不會自動登記公司身分。
不要在同一安裝上混用不同公司的引導檔。

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
| 公司申請仍待審批 | 等待提供者批准，在原私密連結查詢；公開通用包不包含公司憑證 |
| Live 沒有資料 | 配置客戶自己的 Entra、Azure／Fabric 權限，不以 Demo 數字代替 |

Azure 操作參考：[設定 VM 內建 DNS 名稱](https://learn.microsoft.com/en-us/azure/virtual-machines/create-fqdn)。
HTTPS 原理：[Caddy 自動 HTTPS](https://caddyserver.com/docs/automatic-https)。
