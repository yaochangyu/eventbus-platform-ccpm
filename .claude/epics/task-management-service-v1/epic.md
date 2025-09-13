---
name: task-management-service-v1
status: backlog
created: 2025-09-13T13:14:02Z
progress: 0%
prd: .claude/prds/task-management-service-v1.md
github: [Will be updated when synced to GitHub]
---

# Epic: Task Management Service V1

## Overview

實作基於 Queue 和 Callback 架構的任務管理服務，採用 ASP.NET Core 9 的極簡MVP設計。服務提供完整的任務生命週期管理（建立→取出→執行→追蹤），並支援漸進式架構演進。核心設計理念：簡單、可測試、可擴展。

## Architecture Decisions

### 核心技術選擇
- **Web Framework**: ASP.NET Core 9 - 最新穩定版本，原生支援高效能API
- **儲存策略**: Memory First → Database Later - 降低複雜度，加速MVP交付
- **Queue 實作**: ConcurrentQueue → RabbitMQ/Kafka - 階段式升級策略
- **HTTP 客戶端**: HttpClient with Polly - 內建重試和容錯機制
- **測試框架**: xUnit + SpecFlow - BDD驅動開發確保品質

### 設計模式
- **Repository Pattern**: 抽象化資料存取，支援未來切換
- **Strategy Pattern**: Queue和Storage的可插拔實作
- **Result Pattern**: 統一錯誤處理，避免異常洩漏
- **CQRS Lite**: 讀寫分離的簡化版，提升效能

### 關鍵架構決定理由
1. **Memory優先**: 快速驗證商業邏輯，避免基礎設施複雜度
2. **單體設計**: 簡化部署和監控，符合團隊規模
3. **RESTful API**: 標準化介面，易於整合和測試
4. **無認證**: 內部服務，降低安全複雜度

## Technical Approach

### Backend Services

**核心API層 (Controllers)**
```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class TasksController : ControllerBase
{
    // POST /api/v1/tasks - 建立任務
    // POST /api/v1/tasks/{id}/dequeue - 取出任務
    // POST /api/v1/tasks/{id}/execute - 執行任務
    // GET /api/v1/tasks/{id} - 查詢任務
    // DELETE /api/v1/tasks/{id} - 取消任務
    // GET /api/v1/tasks - 任務列表查詢
}
```

**領域模型 (Domain Models)**
```csharp
// 精簡的任務實體設計
public class TaskItem
{
    public Guid Id { get; init; }
    public string Name { get; set; }
    public string CallbackUrl { get; set; }
    public TaskStatus Status { get; set; }
    public object Payload { get; set; }
    public DateTime CreatedAt { get; init; }
    public DateTime UpdatedAt { get; set; }
}

// 狀態枚舉
public enum TaskStatus { Queued, Pending, Processing, Completed, Failed, Cancelled }
```

**服務層 (Business Logic)**
- `ITaskService`: 任務CRUD和狀態管理
- `IQueueService`: Queue抽象，支援多種實作
- `ICallbackService`: HTTP回調處理
- `IArchiveService`: 完成任務的封存邏輯

**儲存層 (Data Access)**
- `ITaskRepository`: 統一的資料存取介面
- `InMemoryTaskRepository`: Phase 1 記憶體實作
- `SqlServerTaskRepository`: Phase 2 SQL Server實作

### Infrastructure

**依賴注入配置**
```csharp
// Program.cs - 服務註冊
builder.Services.AddScoped<ITaskService, TaskService>();
builder.Services.AddScoped<ITaskRepository, InMemoryTaskRepository>();
builder.Services.AddSingleton<IQueueService, InMemoryQueueService>();
builder.Services.AddHttpClient<ICallbackService, CallbackService>();
```

**健康檢查和監控**
- `/health` 端點
- Structured Logging (Serilog)
- 基本 Metrics 收集

**配置管理**
```json
{
  "TaskManagement": {
    "MaxConcurrentTasks": 100,
    "CallbackTimeoutSeconds": 30,
    "ArchiveCompletedTasks": true
  }
}
```

## Implementation Strategy

### 開發方法論
1. **BDD First**: 每個功能先寫測試場景
2. **Red-Green-Refactor**: 標準TDD循環
3. **垂直切片**: 每次實作完整的功能端到端
4. **增量交付**: 每個Sprint可獨立部署

### Phase 1: MVP Core (2週)
**目標**: 功能完整的Memory版本

**風險緩解策略**
- 抽象層設計: 為Phase 2升級做準備
- 完整測試覆蓋: 確保重構時安全
- 效能基準測試: 驗證100併發需求

### Phase 2: Production Ready (2週)
**目標**: 企業級持久化和擴展性

**升級策略**
- 無停機切換: 透過配置切換儲存層
- 資料遷移工具: Memory到SQL的平滑轉移
- 回滾機制: 支援緊急降級

## Task Breakdown Preview

基於架構分析，精簡為 **8個核心任務**：

- [ ] **T1: 專案基礎架構** - ASP.NET Core 9專案、DI容器、基礎配置
- [ ] **T2: 領域模型設計** - TaskItem實體、狀態枚舉、Result模式
- [ ] **T3: Memory儲存實作** - InMemoryRepository、Queue服務、抽象介面
- [ ] **T4: API控制器開發** - 6個核心端點、模型驗證、錯誤處理
- [ ] **T5: Callback機制** - HttpClient整合、重試邏輯、狀態更新
- [ ] **T6: BDD測試套件** - SpecFlow設定、完整測試場景、測試資料
- [ ] **T7: 效能驗證** - 併發測試、效能基準、記憶體使用分析
- [ ] **T8: 部署準備** - Docker化、健康檢查、監控整合

## Dependencies

### 外部相依性
- **Callback API服務**: 目標系統必須提供可呼叫的HTTP端點
- **開發環境**: .NET 9 SDK, Docker Desktop
- **測試環境**: 穩定的網路連線用於Callback測試

### 內部相依性
- **架構審查**: 確保設計符合企業標準
- **BDD培訓**: 團隊熟悉SpecFlow和Gherkin語法
- **DevOps支援**: CI/CD流程建立

### 技術相依性
```xml
<!-- 核心套件清單 -->
<PackageReference Include="Microsoft.AspNetCore.App" Version="9.0" />
<PackageReference Include="Polly" Version="8.2.0" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="SpecFlow.xUnit" Version="3.9.74" />
```

## Success Criteria (Technical)

### 功能驗收標準
- ✅ **API完整性**: 6個端點全部正常運作，符合OpenAPI規格
- ✅ **狀態一致性**: 任務狀態轉換100%正確，無資料競爭
- ✅ **Callback可靠性**: HTTP呼叫成功率>95%，失敗處理正確

### 效能基準
- ✅ **回應速度**: API回應時間<200ms (95th percentile)
- ✅ **併發處理**: 同時處理100個任務無效能劣化
- ✅ **記憶體使用**: 穩態記憶體<512MB，無記憶體洩漏

### 品質閾值
- ✅ **測試覆蓋率**: 程式碼覆蓋率>80%
- ✅ **BDD場景**: 所有Gherkin場景通過
- ✅ **靜態分析**: SonarQube品質門檻通過

### 可維護性指標
- ✅ **架構清晰度**: 依賴關係圖清楚，職責分離明確
- ✅ **文件完整性**: API文件、部署指南、故障排除手冊
- ✅ **監控就緒**: 健康檢查、日誌結構化、基礎指標收集

## Estimated Effort

### 時程規劃
**總工期**: 4週 (Phase 1: 2週，Phase 2: 2週)

**Phase 1 詳細估算**:
- Week 1: T1-T3 (架構+模型+儲存) - 3天
- Week 1: T4-T5 (API+Callback) - 2天
- Week 2: T6-T8 (測試+效能+部署) - 5天

**關鍵路徑項目**:
1. **T3 Memory儲存**: 架構基礎，影響後續所有開發
2. **T5 Callback機制**: 最複雜邏輯，需要充分測試時間
3. **T6 BDD測試**: 品質保證，決定交付信心

### 資源需求
- **開發人員**: 1-2名資深.NET開發者
- **測試協作**: 0.5名測試工程師支援BDD場景設計
- **DevOps支援**: 0.2名DevOps工程師處理Docker化

### 風險緩衝
- **技術風險緩衝**: 20% (Callback整合的不確定性)
- **整合測試緩衝**: 15% (外部API依賴測試)
- **品質保證緩衝**: 10% (BDD測試調優時間)

**總緩衝時間**: 1週額外時間用於處理預期外的技術挑戰

## Tasks Created (BDD順序修正版)

### Phase 1: 測試驅動設置 (Test-First Setup)
- [ ] 001.md - 專案架構與測試基礎建置 (parallel: false) - 4-6 hours
- [ ] 002.md - BDD測試框架設置與配置 (parallel: false, depends on: 001) - 8-10 hours
- [ ] 003.md - 領域模型設計與實作 (parallel: true, depends on: 001) - 8-12 hours
- [ ] 004.md - Gherkin場景撰寫與測試案例設計 (parallel: false, depends on: 002, 003) - 12-16 hours

### Phase 2: Red-Green-Refactor 循環
- [ ] 005.md - Memory儲存層實作 (Red→Green) (parallel: false, depends on: 003, 004) - 10-12 hours
- [ ] 006.md - API控制器開發 (Red→Green) (parallel: false, depends on: 005) - 12-16 hours
- [ ] 007.md - Callback機制實作 (Red→Green) (parallel: true, depends on: 006) - 14-18 hours

### Phase 3: 品質驗證與部署
- [ ] 008.md - 效能驗證測試實作 (parallel: true, depends on: 006, 007) - 12-16 hours
- [ ] 009.md - 部署環境準備與DevOps整合 (parallel: false, depends on: 001-008) - 14-18 hours

## BDD 流程亮點 ✨
- **測試先行**: T002、T004 在實作前建立完整測試框架和場景
- **Red→Green循環**: T005-T007 專注於讓測試從紅燈變綠燈
- **每個實作任務都有明確的BDD驗證標準**

Total tasks: 9
Parallel tasks: 3 (003, 007, 008)
Sequential tasks: 6 (001, 002, 004, 005, 006, 009)
Estimated total effort: 94-136 hours

---

**Epic狀態**: 已重新分解為9個BDD驅動任務 ✅
**預計開工**: Sprint規劃完成後
**里程碑**: Phase 1 MVP (2週後), Phase 2 Production (4週後)