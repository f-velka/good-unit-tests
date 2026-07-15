---
name: good-unit-tests
description: Write or review unit tests according to Vladimir Khorikov's "Unit Testing Principles, Practices, and Patterns" (「単体テストの考え方・使い方」). Use whenever writing new unit tests, reviewing existing test code, refactoring a test suite, deciding what to mock/stub, deciding what deserves a unit test vs an integration test, or when a test feels fragile/flaky/hard to maintain.
---

# Good Unit Tests

`Unit Testing Principles, Practices, and Patterns` (Vladimir Khorikov) の主張に基づき、
価値のある単体テストを書く・レビューするためのスキル。テストを書く/直す作業に入る前に
このファイルを読み、必要に応じて `references/` の各ファイルを参照すること。

## 良いテストの定義：4本の柱

テストの価値は次の4本の柱で決まる。**4つすべてを同時に最大化することはできない**。
特に「リファクタリングへの耐性」は妥協してはならない基礎的な性質として扱う
（これが低いテストは、他の3本がどれだけ高くても価値が低い）。

1. **退行に対する保護** — バグを検出できるか（実行されるプロダクションコード量、ドメインの複雑さに比例）
2. **リファクタリングへの耐性** — 実装を変えても振る舞いが変わらなければテストが壊れないか（偽陽性を生まないか）
3. **迅速なフィードバック** — 実行が速いか
4. **保守のしやすさ** — 読みやすいか、保守にコストがかからないか

詳細・トレードオフの考え方は `references/four-pillars.md` を参照。

## テストを書く/レビューする手順

### 1. 対象コードを4種類に分類する（Humble Object）

テスト戦略を決める前に、対象コードが以下のどれかを判定する。

- **ドメインモデル/アルゴリズム**（複雑度・ドメイン重要性: 高、協力者数: 少）→ 単体テストで手厚くカバーする主戦場
- **取るに足らないコード**（複雑度: 低、協力者数: 少）→ テスト不要（getter/setter、引数なしコンストラクタ等）
- **コントローラ**（複雑度: 低、協力者数: 多）→ 単体テストではなく統合テストで薄くカバー
- **過度に複雑なコード**（複雑度: 高、協力者数: 多）→ そのままテストしない。Humble Objectパターンで
  ロジックをドメインモデル（狭いが深い）とコントローラ（広いが浅い）に分割してから、それぞれを上記の方針でテストする

詳細は `references/code-classification.md`。

### 2. 何を検証するかは「観察可能な振る舞い」だけにする

- テストは公開API（クライアントが目的を達成するために使う公開の操作・状態）だけを対象にする
- private メソッド/フィールドを直接テストしない。テストのためだけにカプセル化を破らない
- 実装の詳細（内部アルゴリズム、呼び出し順序、内部状態）に依存するアサーションを書かない
- 迷ったら「このテストは、対象コードをまったく違う実装に書き換えても（振る舞いが同じなら）通り続けるか？」を自問する

### 3. テストダブル（モック/スタブ）の使用方針

- **モック化してよいのは「管理下にない依存」（out-of-process かつ自分たちが管理していない外部システム、例: 決済API、メール送信）だけ**
- 自社のDBのような「管理下にある依存」はモックにせず、実物（または実物に近いテスト用インスタンス）を使う
- モックは「外部への出力コミュニケーション」の検証にのみ使う。入力の模倣（内部状態の準備）にはスタブを使い、スタブの呼び出しをアサートしない
- 呼び出し回数・順序などをモックで過剰に検証しない（overspecification を避ける）
- 単体テストの検証手法は 出力値ベース > 状態ベース > コミュニケーションベース の優先順で選ぶ（この順に保守性・リファクタリング耐性が高い）
- 学派は古典学派（Detroit school）を採用する: テスト対象は「1つのクラス」ではなく「1つの振る舞い」。共有依存（DBなど、テスト間で状態が漏れる依存）のみを置き換える

詳細は `references/mocking-policy.md`。

### 4. AAA パターンを守る

- 1テストにつき Arrange / Act / Assert は各1ブロック（Actは1回だけ）
- 同じフェーズが複数回現れる場合は、複数の振る舞いを1テストに詰め込んでいる兆候 → テストを分割する
- テスト本体に `if`/`for`/`while` などの分岐・ループを書かない（テストのサイクロマティック複雑度は0に保つ）
- Arrange が複雑になりすぎる場合はテスト用ファクトリメソッド／テストデータビルダーを使って整理する（`references/checklist.md` 参照）

### 5. テスト名は「振る舞いの説明」にする

- `MethodName_Scenario_ExpectedResult` のような機械的な命名規則に縛られない
- そのドメインを知らない非開発者が読んでも意味が分かる、自然文に近い説明にする
  （例: `delivery_with_a_past_date_is_invalid`）

### 6. 仕上げチェック

テストを書き終えたら `references/checklist.md` のアンチパターンリストと突き合わせて確認する。

## クイックリファレンス

- `references/four-pillars.md` — 4本の柱とトレードオフの詳細
- `references/code-classification.md` — 4種類のコード分類とHumble Objectパターン
- `references/mocking-policy.md` — モック/スタブ方針、古典学派/ロンドン学派、3つの検証手法
- `references/checklist.md` — アンチパターンのチェックリスト（テスト完成後に照合する）
