# MonaのProject Pulseダッシュボード開発チーム

このチームは、ローカル環境で起動したGitHub Copilot CLIを窓口にして、Mona向けProject Pulseダッシュボードの調査、計画、UI設計、実装を協調して進めます。エージェントの役割と利用可能なツールは、以下のカスタムエージェント定義に従います。

| エージェント | モデル | 責任範囲 | 利用可能なツール | 定義ファイル |
|---|---|---|---|---|
| **Orchestrator** | Claude Opus 5.5 (copilot) | 作業をタスクに分解し、専門エージェントへ委任します。担当ファイルと依存関係を調整し、成果を統合・確認して結果を報告します。自ら実装はしません。 | `read`（読み取り）、`agent`（エージェントの委任）、`memory`（記憶） | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 5.5 (copilot) | リポジトリや関連ドキュメントを調査して実装計画を作ります。手順、担当ファイル、依存関係、並列化できる作業、エッジケース、検証項目、未解決の質問を示します。コードは書きません。 | `read`（読み取り）、`search`（検索）、`web`（ウェブ参照）、`memory`（記憶）、`todo`（タスク管理） | `.github/agents/planner.agent.md` |
| **Designer** | Gemini 3.7 Flash (copilot) | UI/UX、アクセシビリティ、情報設計、操作フロー、視覚デザイン、レスポンシブ対応を担当します。Project Pulseでは、カード、ステータス、優先度、余白などを整え、ダッシュボードらしい見た目を設計します。 | `read`（読み取り）、`edit`（編集）、`search`（検索）、`web`（ウェブ参照）、`memory`（記憶）、`todo`（タスク管理） | `.github/agents/designer.agent.md` |
| **Coder** | GPT-5.5 (copilot) | 割り当てられた範囲でコードを書き、バグ修正やアプリの実装を行います。Project Pulseでは、依頼された場合に起動設定などの実行補助ファイルも作成し、変更を検証します。 | `read`（読み取り）、`edit`（編集）、`search`（検索）、`execute`（コマンド実行）、`web`（ウェブ参照）、`memory`（記憶）、`todo`（タスク管理） | `.github/agents/coder.agent.md` |

## 協力する順序

1. Orchestratorが依頼を受け、Plannerにリポジトリ調査と実装計画を依頼します。
2. Plannerが計画と担当範囲を提示した後、Orchestratorは依存関係に沿って作業を段階分けします。
3. DesignerがProject Pulseの画面構成や視覚方針を定め、その設計をCoderへ渡します。ファイル範囲が重ならず、先行成果に依存しない作業に限り、DesignerとCoderの作業を並行できます。
4. Coderが割り当てられた範囲で実装し、適切な検証を行います。重複する範囲や先行成果が必要な作業は、順番に進めます。
5. Orchestratorが成果を統合して整合性を確認し、結果や残る課題を報告します。

すべてのエージェントは割り当てられたファイル範囲を守り、Gitへのステージ、コミット、プッシュは行いません。Git操作は利用者がローカルのGitHub Copilot CLIから行います。
