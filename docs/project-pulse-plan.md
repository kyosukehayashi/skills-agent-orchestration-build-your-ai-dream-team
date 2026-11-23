# Project Pulse ダッシュボード実装計画

## 概要

Mona のチームメンバーが、進行中のプロジェクト、担当者、ステータス、最近の活動、優先度またはリスク、および短い概要をひと目で把握できる、軽量な静的ダッシュボードを作る。`.github/project-pulse-brief.md` と `.github/steps/3-step.md` に記載された要件に従い、HTML/CSS/JSON の画面と VS Code の実行設定を用意する。

## 調査結果と前提

- `app/` は存在するが、現時点でアプリのファイルはない。`.vscode/launch.json` もまだない。
- `docs/agent-team.md` には Orchestrator が Planner、Designer、Coder を調整する既存の役割分担が記載されている。
- Planner は調査と計画を担当し、コードを書かない。Designer は視覚デザイン、情報設計、アクセシビリティ、レスポンシブ対応を担う。Coder は割り当てられたアプリの実装を担い、指定された場合は実行用の設定も作成する。
- Project Pulse のデータはトップレベルの `projects` 配列とし、各項目に `name`、`owner`、`status`、`recentActivity`、`priority` を含める。brief は短い contributor 向け概要も求めているため、各項目に `summary` を加えることを提案する。これは追加フィールドであり、必須フィールドの置き換えではない。
- リポジトリには `scripts/validate-exercise.sh` があるが、これは主に演習リポジトリや手順を検査するもので、完成したダッシュボードの表示・動作を検証するアプリテストではない。アプリ用パッケージマニフェストや専用テストスクリプトは確認できない。
- `.github/steps/3-step.md` は起動設定に `python3 -m http.server 5500`、`app/` からの配信、`http://localhost:%s/index.html` を開く `serverReadyAction` を指定している。Coder の定義は `cwd` を `${workspaceFolder}/app` とし、`index.html` を開くよう求めている。

## ファイル所有権

| ファイル | 作成・編集の担当 | 範囲と責任 |
|---|---|---|
| `app/index.html` | **Coder** | 「Project Pulse」のタイトル、意味のある見出し構造、`styles.css` と `project-data.json` の参照、プロジェクトカードの描画を実装する。各カードで名前、担当者、ステータス、最近の活動、優先度、および概要を読めるようにする。 |
| `app/styles.css` | **Designer** | カードを中心にした完成度のある画面、情報階層、余白、文字、ステータスと優先度の視覚表現、レスポンシブ対応、可読性・アクセシビリティを実装する。`.dashboard` と `.project-card` を必ず含め、カードに `border-radius` と `box-shadow` を用いる。 |
| `app/project-data.json` | **Coder** | トップレベル `projects` 配列と、brief 指定の必須フィールドを持つ複数のサンプルプロジェクトを用意する。brief の短い概要要件に応える `summary` の追加を推奨する。 |
| `.vscode/launch.json` | **Coder** | strict JSON（コメントなし）で `Run Project Pulse Dashboard` を定義する。`cwd` は `${workspaceFolder}/app`、サーバーは `python3 -m http.server 5500`、起動後の URL は `http://localhost:%s/index.html` とし、フォルダー一覧でなく `index.html` を開く。 |

Designer は `app/styles.css` だけを編集し、Coder は残りの3ファイルだけを編集する。Orchestrator はファイルを直接実装せず、受け渡し条件と統合確認を担う。この分担は Designer のスタイリング期待と、Coder が指定された場合に launch 設定も作成するという定義に沿い、同じファイルへの同時編集を避ける。

## 実装手順と依存関係（Dependencies）

1. **前提と画面契約を確定する（Orchestrator）**
   - brief と本計画を基準とし、`projects` のフィールド名、HTML と CSS の接続に使うクラス、カードで表示する情報を Designer と Coder に共有する。
   - 少なくとも `.dashboard` と `.project-card` を共通の CSS フックとして指定する。追加のクラスや視覚上の選択は Designer が明示し、Coder は HTML に反映する。
2. **スタイルと静的アプリを実装する（Designer / Coder）**
   - Designer は確定したフックを前提に `app/styles.css` を作成する。
   - Coder は `app/project-data.json` と `app/index.html` を作成し、データからカードを表示する。画面と JSON は同じフィールド契約に従う。ローカル HTTP サーバーで開く想定とし、データ読み込みの失敗や空データ時に利用者が状況を把握できる表示を検討する。
   - Coder は指定済みのサーバー起動条件で `.vscode/launch.json` も作成する。
3. **統合・検証する（Orchestrator、必要に応じて担当者が修正）**
   - HTML のクラスが Designer の CSS に存在し、JSON の項目が HTML の表示に対応することを確認する。
   - 静的チェックの後、指定 launch 設定から起動して画面を確認する。見つかった問題の修正は該当ファイルの所有者に割り当てる。

**依存関係:** 手順1の共通契約は Designer と Coder の画面実装に先行する。Designer の CSS と Coder の HTML/JSON は、その契約が共有された後なら並行できる。HTML と JSON はスキーマの一致が必要なので、Coder が同一担当として協調して実装する。launch 設定は固定された起動条件を使うため、画面ファイルの実装に依存しないが、最終的な統合確認は全ファイルの作成後に行う。

## 並列作業の判断（Parallel work）

**条件付きで並列実行できる。** Orchestrator が共通のクラス・データ契約を先に渡した後、Designer は `app/styles.css`、Coder は `app/index.html` と `app/project-data.json` を担当し、Coder は同じ割り当て内で `.vscode/launch.json` も作成できる。ファイル所有権が重ならず、brief がデータキーと主要フックを既に定めているためである。

**並行してはならない作業:** 共有契約を決める前に Designer と Coder が個別に画面クラスやデータ形状を決めること、互いの担当ファイルを編集すること、実装完了前に統合済みとみなすこと。追加 CSS フックの変更が必要になった場合は、まず Designer と Coder 間で合意し、その後に各自の所有ファイルへ反映する。最終確認と launch 設定からの実行は、4ファイルが揃ってから行う。

## 検証

1. **演習ステップ2の期待:** `docs/project-pulse-plan.md` に Project Pulse、Designer/Coder の責任、4つのファイルパス、依存関係、並列作業の判断、validation を記述する。`.github/workflows/2-step.yml` はこれらの語句・パスおよび計画ファイルの存在をチェックする。
2. **JSON 構文:** Python 3 が利用可能な環境では `python3 -m json.tool app/project-data.json` と `python3 -m json.tool .vscode/launch.json` を実行する。`.github/workflows/3-step.yml` もこの2ファイルを JSON として検査する。
3. **内容と接続:** `.github/workflows/3-step.yml` が確認する、タイトル、CSS/JSON 参照、カード用フック、ステータス・活動・優先度の表示、`.dashboard` / `.project-card` / `border-radius` / `box-shadow`、トップレベル `projects` と各必須データフィールド、launch 名と `index.html` を目視でも確認する。HTML に `project-data.json` を参照するだけでなく、データ項目が実際のカードとして表示されることを確認する。
4. **実行確認:** VS Code の Run and Debug から `Run Project Pulse Dashboard` を実行し、サーバーが `app/` を配信し、ブラウザーが `index.html` を開くことを確認する。複数カード、情報の読みやすさ、狭い画面でのレイアウトも確認する。直接 `file://` で開くことを正規の実行経路とはせず、指定された HTTP 起動設定を使う。
5. `scripts/validate-exercise.sh` は演習リポジトリ全体のチェックとして必要に応じて実行できる。ただし、アプリの実行確認や視覚的な品質の代わりにはならない。また、このスクリプトの実行には Bash、Ruby、Python 3 などが必要であり、現環境にすべて揃っているとは確認できていない。

## リスクと未解決事項

- ステータスや優先度の具体的な値・語彙は brief で定義されていない。サンプルデータで一貫した値を選び、必要なら Orchestrator が合意を取る。特定の色だけに意味を依存させず、バッジに文字ラベルも示す。
- brief の概要要件を満たす `summary` は、列挙された必須フィールドには含まれない。必須フィールドを保ったまま追加する方針だが、厳密に最小スキーマに限定するかは確認余地がある。
- 「recentActivity」の日時形式や内容、サンプルプロジェクト数は指定されていない。実装時に読みやすい例を選び、外部サービスや実データ連携は追加しない。
- `python3` と VS Code のブラウザー起動機能が実行環境で利用できるかは未確認。指定コマンドが使えない場合は環境差として報告し、ユーザー承認なしに別ツールや依存関係を追加しない。
- `scripts/validate-exercise.sh` はプロジェクト機能テストではなく、CI の Step 3 も HTML/CSS の主要チェックを語句で検査するため、実際にページを起動して表示とデータ描画を確認する必要がある。
