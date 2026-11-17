---
title: AI Consumption Control Tower 客戶部署
description: 客戶下載通用編譯安裝包及安裝指南，不包含應用程式源碼、公司帳密或啟用碼。
---

## 下載及安裝

這是公開的客戶交付倉庫，不是應用程式源碼倉庫。
下載不需要 GitHub 帳號。

1. 到[提供者首頁](https://charlesdeng.eastasia.cloudapp.azure.com/ai-finops/)，
   選擇英文／繁體／簡體，填寫公司申請後立即下載專屬 ZIP，不需等待審批。
2. 依 ZIP 內對應語言的手冊或[安裝指南](deploy.md)，在 Ubuntu VM 執行一個命令。
3. 使用 Azure VM 內建的 DNS 名稱，HTTPS 證書由安裝程式自動設定及續期。

不需要購買網域、DNS 代管或網站 SSL 證書。
VM 不需要應用程式源碼、Git、Node.js 或 npm。
VM、公網 IP、流量及自行開通的 Azure／Fabric 服務仍會計費。

只有通過建置及安裝驗證的版本才會出現在 Releases。
如果尚未列出 `.run` 和校驗檔，表示還沒有可交付版本。
GitHub 自動提供的「Source code」壓縮檔只包含本倉庫文件，不是安裝包。

## 通用版與公司專屬版

通用編譯安裝包及校驗值可在
[Releases](https://github.com/chenxideng/ai-consumption-control-tower-deploy/releases)
公開下載，供已登記安裝更新使用，不包含公司帳密或啟用碼。
沒有公司引導及有效連網授權時，受保護功能會鎖定，不能以通用包跳過啟用。

公司專屬 ZIP 提交申請後即可透過私密連結下載，包含安裝程式、
對應語言手冊、安裝引導、臨時帳密及 32 字元啟用碼；
它們不會放在這個公開倉庫或公開 Release。
新授權預設啟用，提供者可隨時停用或重新啟用，下載與使用權分開。
手冊、帳密說明及初始網站語言跟隨下載時的頁面語言。

客戶網站每 60 秒驗證一次簽署授權，有效租期最長 90 秒。
授權被停用或失聯而無法續期，會鎖定 Dashboard 及受保護 API，
顯示重新申請及補充證明的提示；登入、改密碼及安裝恢復仍可使用。
這項必要驗證不受選用使用回傳開關控制，也不會關閉 Azure 工作負載或停止計費。
舊版客戶包須升級，才可依賴遠端管控。

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
