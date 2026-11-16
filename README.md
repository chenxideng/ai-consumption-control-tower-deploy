---
title: AI Consumption Control Tower 客戶部署
description: 客戶下載通用編譯安裝包及安裝指南，不包含應用程式源碼、公司帳密或啟用碼。
---

## 下載及安裝

這是公開的客戶交付倉庫，不是應用程式源碼倉庫。
下載不需要 GitHub 帳號。

1. 到 [Releases](https://github.com/chenxideng/ai-consumption-control-tower-deploy/releases)
   下載同一版本的 `.run` 安裝包及 `SHA256SUMS`。
2. 按[安裝指南](deploy.md)操作，在 Ubuntu VM 執行一個安裝命令。
3. 使用 Azure VM 內建的 DNS 名稱，HTTPS 證書由安裝程式自動設定及續期。

不需要購買網域、DNS 代管或網站 SSL 證書。
VM 不需要應用程式源碼、Git、Node.js 或 npm。
VM、公網 IP、流量及自行開通的 Azure／Fabric 服務仍會計費。

只有通過建置及安裝驗證的版本才會出現在 Releases。
如果尚未列出 `.run` 和校驗檔，表示還沒有可交付版本。
GitHub 自動提供的「Source code」壓縮檔只包含本倉庫文件，不是安裝包。

## 通用版與公司專屬版

通用安裝包可公開下載。安裝時自行設定 `superadmin` 密碼，
首次登入需要更改密碼。沒有共用的預設密碼，也不包含公司啟用碼、
回傳身分或客戶資料。先安裝即可使用 Synthetic Demo。

公司專屬 ZIP 需要向提供者提交申請，再由管理員審批。
獲准後才可透過專屬私密連結下載安裝引導、臨時帳密和啟用碼；
它們不會放在這個公開倉庫或公開 Release。
通用包的公開下載不等於公司申請獲批，也不會自動開通公司回傳。

Azure Live、Fabric 資料及告警儲存需另外配置並使用客戶自己的授權。
安裝不會自動取得 Azure 權限、購買 Fabric 或啟用工作負載熔斷。

## 交付與保護邊界

Release 只提供編譯安裝包及 SHA-256 校驗檔。
安裝包包含執行所需的編譯產物、安裝工具和第三方授權聲明，
不包含原始應用專案、後端 JavaScript 源檔、Git 歷史、source maps、
環境設定、私鑰、Token 或任何公司的帳密及啟用碼。

編譯不是不可逆加密或 DRM。VM 管理員仍可能逆向程式，
瀏覽器執行的編譯後前端 JavaScript 也必然可見。
請勿在 Issues、檔案、截圖或 Release 上傳客戶資料或任何登入資訊。
