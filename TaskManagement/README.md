# TaskManagement Service

基於 Queue 和 Callback 架構的任務管理服務，採用 BDD 驅動開發方法。

## 專案結構

```
TaskManagement/
├── TaskManagement.sln                          # 解決方案檔案
├── src/
│   └── TaskManagement.Api/                     # 主要 API 專案
│       ├── Controllers/                        # API 控制器
│       ├── Program.cs                          # 應用程式進入點
│       ├── appsettings.json                    # 設定檔
│       └── TaskManagement.Api.csproj          # 專案檔
├── tests/
│   ├── TaskManagement.AcceptanceTests/         # BDD 驗收測試 (SpecFlow)
│   │   └── Features/                           # Gherkin 場景檔案
│   ├── TaskManagement.IntegrationTests/        # 整合測試
│   └── TaskManagement.UnitTests/               # 單元測試
└── README.md                                   # 專案說明
```

## 開發環境需求

- .NET 9.0 SDK
- Visual Studio 2022 或 Visual Studio Code
- Git

## 快速開始

### 1. 複製專案
```bash
git clone <repository-url>
cd TaskManagement
```

### 2. 還原套件
```bash
dotnet restore
```

### 3. 建置專案
```bash
dotnet build
```

### 4. 執行 API
```bash
dotnet run --project src/TaskManagement.Api
```

### 5. 執行測試
```bash
# 執行所有測試
dotnet test

# 執行特定測試專案
dotnet test tests/TaskManagement.UnitTests
dotnet test tests/TaskManagement.IntegrationTests
dotnet test tests/TaskManagement.AcceptanceTests
```

## API 端點

應用程式啟動後，可以透過以下端點存取：

- **Swagger UI**: `https://localhost:7xxx/swagger`
- **健康檢查**: `https://localhost:7xxx/health`
- **OpenAPI 規格**: `https://localhost:7xxx/swagger/v1/swagger.json`

## 任務管理 API 端點 (預計實作)

- `POST /api/v1/tasks` - 建立任務
- `POST /api/v1/tasks/{id}/dequeue` - 取出任務
- `POST /api/v1/tasks/{id}/execute` - 執行任務
- `GET /api/v1/tasks/{id}` - 查詢任務
- `DELETE /api/v1/tasks/{id}` - 取消任務
- `GET /api/v1/tasks` - 任務列表查詢

## BDD 開發流程

1. **撰寫 Gherkin 場景** (tests/TaskManagement.AcceptanceTests/Features/)
2. **執行測試** (Red 狀態)
3. **實作功能** 讓測試通過
4. **重構和優化**

## 設定說明

### appsettings.json
- `TaskManagement:MaxConcurrentTasks`: 最大併發任務數 (預設: 100)
- `TaskManagement:CallbackTimeoutSeconds`: Callback 超時時間 (預設: 30 秒)
- `TaskManagement:ArchiveCompletedTasks`: 是否封存完成的任務 (預設: true)

## 開發指南

### 新增 API 端點
1. 在 `src/TaskManagement.Api/Controllers/` 建立控制器
2. 在 `tests/TaskManagement.AcceptanceTests/Features/` 撰寫 BDD 場景
3. 執行測試確保 Red 狀態
4. 實作功能讓測試通過

### 新增測試
- **單元測試**: `tests/TaskManagement.UnitTests/`
- **整合測試**: `tests/TaskManagement.IntegrationTests/`
- **BDD 測試**: `tests/TaskManagement.AcceptanceTests/Features/`

## 部署

專案支援 Docker 容器化部署 (後續版本實作)。

## 貢獻指南

1. Fork 專案
2. 建立功能分支
3. 撰寫測試
4. 實作功能
5. 提交 Pull Request

## 授權

[指定授權條款]