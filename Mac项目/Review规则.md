#  Mac Code Review 规则

代码检查（macOS + SwiftUI 特化版）。

## 检查项
| **检查项** | **内容** |
| --- | --- |
| 架构层级 | 是否严格遵守 L0-L3 分层，View 不包含业务逻辑 |
| 状态管理 | 是否正确使用 `@State`, `@Binding`, `@ObservedObject`，避免状态冗余 |
| 数据绑定 | ViewModel 与 View 之间是否通过数据驱动，禁止手动操作 UI |
| DI (注入) | ViewModel 依赖的 Service 是否通过构造器注入 |
| 异步规范 | 异步操作是否使用 Result + Completion，并在主线程更新状态 |
| Mac 特性 | 菜单栏、侧边栏、快捷键是否按规范实现 |
| 可测试性 | ViewModel 是否易于进行单元测试（无 UI 耦合） |

## 输出评分
| **指标** | **分数** |
| --- | --- |
| 架构质量 (Architecture) | /10 |
| 状态驱动 (State Management) | /10 |
| 可维护性 (Maintainability) | /10 |
| Mac 交互规范 (Human Interface) | /10 |
