# プロンプト：取引ポスティング処理 `CBTRN02C`（バッチジョブ `POSTTRAN`）の詳細設計書をリバースエンジニアリングで生成する

本ファイルは、LLM に渡すことで CardDemo の取引ポスティング処理プログラム
`CBTRN02C`（バッチジョブ `POSTTRAN`）の**詳細設計書（Markdown 形式）**を
リバースエンジニアリングにより自動生成させるための「プロンプト文面」です。
以下の「＝＝＝ プロンプトここから ＝＝＝」から
「＝＝＝ プロンプトここまで ＝＝＝」までをコピーして LLM に渡してください。

---

＝＝＝ プロンプトここから ＝＝＝

あなたはメインフレーム（COBOL/JCL/VSAM）システムのリバースエンジニアリングに
精通したシステムアナリストです。GitHub リポジトリ
`xxxxxtech/aws-mainframe-modernization-carddemo`（ブランチ `main`）の CardDemo
アプリケーションを対象に、取引ポスティング処理を担うバッチ COBOL プログラム
`CBTRN02C`（バッチジョブ `POSTTRAN`）の**詳細設計書**を、ソースコードを解析して
**リバースエンジニアリング**で作成してください。

## 0. 出力に関する基本方針
- 出力は**日本語**で記述すること。不自然な直訳を避け、日本人の技術者が読んで
  違和感のない自然で簡潔な日本語にすること。
- 出力は **Markdown 形式の「詳細設計書」1 本**とすること。
- 記述はすべて**ソースコードの実際の行を根拠**とし、**事実（コードに書かれていること）**と
  **推測（設計意図の解釈やモダナイゼーションの提案）**を明確に区別すること。
  推測部分には「（推測）」と明記すること。
- 各仕様の根拠には、可能な限り**ファイルパスと行番号／段落名**を併記すること。

## 1. 解析対象ファイル
以下のファイルを読み解いて設計書に反映すること。

- 主対象プログラム：`app/cbl/CBTRN02C.cbl`（約 731 行）
- 起動 JCL：`app/jcl/POSTTRAN.jcl`（`EXEC PGM=CBTRN02C`、DD 定義）
- スケジューラ定義：
  - `app/scheduler/CardDemo.ca7`（CA-7 定義。ジョブ `POSTTRAN` の登録・トリガ・
    実行スケジュールの記載あり）
  - `app/scheduler/CardDemo.controlm`（Control-M 定義。関連する取引系ジョブ
    `TRANBKP`／`COMBTRAN` 等が定義されている。`POSTTRAN` 自体の定義有無も確認し、
    事実として記述すること）
- 関連コピーブック（`app/cpy/` 配下。レコードレイアウトの根拠）：
  - `CVTRA06Y.cpy` … 日次取引レコード `DALYTRAN-RECORD`（RECLN=350）
  - `CVTRA05Y.cpy` … 取引マスタレコード `TRAN-RECORD`（RECLN=350）
  - `CVACT03Y.cpy` … カード相互参照レコード `CARD-XREF-RECORD`（RECLN=50）
  - `CVACT01Y.cpy` … 口座レコード `ACCOUNT-RECORD`（RECLN=300）
  - `CVTRA01Y.cpy` … 取引カテゴリ残高レコード `TRAN-CAT-BAL-RECORD`（RECLN=50）
- 参考：`README.md` の Batch Components 表（`POSTTRAN | CBTRN02C | Transaction processing job` の記載）

## 2. 詳細設計書に含めるべきセクション
以下のセクションを、この順序で必ず含めること。

### (1) ドキュメント情報
- 対象プログラム：`CBTRN02C`
- 種別：バッチ COBOL プログラム
- ジョブ名：`POSTTRAN`
- 作成方式：リバースエンジニアリング（ソースコード解析）
- 対象リポジトリ／ブランチ、解析対象ファイル一覧

### (2) 機能概要
- 日次取引ファイルのレコードを検証し、口座残高・取引カテゴリ残高へ反映し、
  取引マスタへ登録する処理であること。
- 検証で不合格となった取引はリジェクトファイルへ出力すること。
- 処理件数・リジェクト件数の集計と、リジェクトが 1 件以上ある場合の
  リターンコード 4 設定（`app/cbl/CBTRN02C.cbl` 229-231 行付近）にも言及すること。

### (3) 起動条件・JCL・パラメータ・実行スケジュール
- `app/jcl/POSTTRAN.jcl` の内容（ステップ `STEP15`、`EXEC PGM=CBTRN02C`、`STEPLIB`）を説明する。
- DD 名とデータセットの対応表を作成する（例：`DALYTRAN`→`AWS.M2.CARDDEMO.DALYTRAN.PS`、
  `TRANFILE`→`...TRANSACT.VSAM.KSDS`、`XREFFILE`→`...CARDXREF.VSAM.KSDS`、
  `ACCTFILE`→`...ACCTDATA.VSAM.KSDS`、`TCATBALF`→`...TCATBALF.VSAM.KSDS`、
  `DALYREJS`→`...DALYREJS(+1)`（GDG、`RECFM=F,LRECL=430`））。
- スケジューラ（CA-7／Control-M）上の位置づけ・トリガ関係・実行タイミングを、
  スケジューラ定義ファイルから読み取れる事実に基づいて記述する。

### (4) 入出力ファイル一覧
論理名・DD 名・データセット名・コピーブック・アクセス方式（順次／索引 RANDOM）・
レコードキー・入出力区分・用途を表形式でまとめること。
`FILE-CONTROL`（`app/cbl/CBTRN02C.cbl` 28-61 行）を根拠にすること。
- `DALYTRAN-FILE`（順次・入力）
- `TRANSACT-FILE`（索引・RANDOM・キー `FD-TRANS-ID`・出力）
- `XREF-FILE`（索引・RANDOM・キー `FD-XREF-CARD-NUM`・参照）
- `ACCOUNT-FILE`（索引・RANDOM・キー `FD-ACCT-ID`・更新）
- `TCATBAL-FILE`（索引・RANDOM・キー `FD-TRAN-CAT-KEY`・更新／新規作成）
- `DALYREJS-FILE`（順次・リジェクト出力）

### (5) レコードレイアウト
上記コピーブックごとに、項目名・PIC・桁数・説明を表形式で記述すること。
特に以下は必須：
- `DALYTRAN-RECORD`（`CVTRA06Y.cpy`）：取引 ID・種別・カテゴリ・金額 `PIC S9(09)V99`・
  カード番号・タイムスタンプ等。
- `ACCOUNT-RECORD`（`CVACT01Y.cpy`）：`ACCT-CURR-BAL`／`ACCT-CREDIT-LIMIT`／
  `ACCT-CURR-CYC-CREDIT`／`ACCT-CURR-CYC-DEBIT`／`ACCT-EXPIRAION-DATE`
  （原文の綴りのまま）等。
- `CARD-XREF-RECORD`（`CVACT03Y.cpy`）：`XREF-CARD-NUM`／`XREF-CUST-ID`／`XREF-ACCT-ID`。
- `TRAN-CAT-BAL-RECORD`（`CVTRA01Y.cpy`）：キー（口座 ID＋種別＋カテゴリ）＋残高 `TRAN-CAT-BAL`。
- 金額項目は COMP-3（パック 10 進）ではなく表示形式だが、桁と符号・小数 2 桁を明記すること。

### (6) 処理フロー
- 主処理（`PROCEDURE DIVISION` 193-234 行）の全体フローを説明する：
  各ファイル OPEN → 日次取引を EOF まで READ → 1 件ごとに検証（`1500-VALIDATE-TRAN`）→
  合格なら `2000-POST-TRANSACTION`、不合格ならリジェクト件数加算＋`2500-WRITE-REJECT-REC` →
  全ファイル CLOSE → 集計表示 → リジェクトありなら `RETURN-CODE=4`。
- **mermaid のフローチャート**（`flowchart TD`）で主処理ループと検証・ポスティングの分岐を図示すること。

### (7) 段落（PARAGRAPH）別ロジック仕様
主要段落ごとに入力・処理・出力・呼び出し関係・エラー時挙動を記述すること。
特に以下は内部ロジックを詳細に記述すること。
- `1500-VALIDATE-TRAN`（370-378 行）：`1500-A-LOOKUP-XREF` を呼び、失敗が無ければ
  `1500-B-LOOKUP-ACCT` を呼ぶ。`* ADD MORE VALIDATIONS HERE`（377 行）という
  拡張余地コメントの存在にも触れること。
- `1500-A-LOOKUP-XREF`（380-392 行）：`DALYTRAN-CARD-NUM` で XREF ファイルを READ、
  `INVALID KEY` 時に理由コード 100「INVALID CARD NUMBER FOUND」を設定。
- `1500-B-LOOKUP-ACCT`（393-422 行）：`XREF-ACCT-ID` で口座を READ。`INVALID KEY` 時に
  理由コード 101「ACCOUNT RECORD NOT FOUND」。残高計算（後述）と与信限度・有効期限の
  検証を行う。
- `2000-POST-TRANSACTION`（424-444 行）：日次取引項目を取引マスタ項目へ MOVE し、
  DB2 形式タイムスタンプ生成、`2700-UPDATE-TCATBAL`／`2800-UPDATE-ACCOUNT-REC`／
  `2900-WRITE-TRANSACTION-FILE` を呼ぶ。
- `2700-UPDATE-TCATBAL`／`2700-A-CREATE-TCATBAL-REC`／`2700-B-UPDATE-TCATBAL-REC`
  （467-542 行）：カテゴリ残高の READ、存在しなければ新規 WRITE、存在すれば加算 REWRITE。
- `2800-UPDATE-ACCOUNT-REC`（545-560 行）：口座残高・当月クレジット／デビットの更新と REWRITE。
- `2500-WRITE-REJECT-REC`（446-465 行）：リジェクトレコード＋バリデーショントレーラの出力。
- `2900-WRITE-TRANSACTION-FILE`（562-579 行）：取引マスタへの WRITE。
- `Z-GET-DB2-FORMAT-TIMESTAMP`（DB2 形式タイムスタンプ生成、690-705 行付近）。
- `9999-ABEND-PROGRAM`（707-711 行）：`CEE3ABD` による ABEND（ABCODE=999）。
- `9910-DISPLAY-IO-STATUS`（714-727 行）：ファイルステータス表示ロジック。
- 各 OPEN/CLOSE 段落（`0000`〜`0500`、`9000`〜`9500`）は共通のエラー処理パターン
  （異常時に IO ステータス表示＋ABEND）としてまとめて記述してよい。

### (8) バリデーション仕様・エラー／リジェクト理由コード一覧
コードから抽出した理由コードを表形式でまとめること（コード・メッセージ・
設定箇所・意味・発生条件）。少なくとも以下を含めること。
- 100：`INVALID CARD NUMBER FOUND`（XREF 未登録、385-387 行）
- 101：`ACCOUNT RECORD NOT FOUND`（口座未登録、397-399 行）
- 102：`OVERLIMIT TRANSACTION`（与信超過、407-413 行）
- 103：`TRANSACTION RECEIVED AFTER ACCT EXPIRATION`（口座有効期限切れ、414-420 行）
- 109：`ACCOUNT RECORD NOT FOUND`（口座 REWRITE 時の INVALID KEY、555-558 行）
理由コードは `WS-VALIDATION-FAIL-REASON`（`PIC 9(04)`、181 行）に格納され、
0 以外のときリジェクトとなる（211-216 行）点も明記すること。

### (9) 残高計算・更新ロジック（計算式レベル）
コードの `COMPUTE`／`ADD` を計算式として明記すること。
- 与信判定用の残高見込み（403-405 行）：
  `WS-TEMP-BAL = ACCT-CURR-CYC-CREDIT − ACCT-CURR-CYC-DEBIT + DALYTRAN-AMT`
- 与信超過判定（407 行）：`ACCT-CREDIT-LIMIT >= WS-TEMP-BAL` を満たさなければ理由 102。
- 有効期限判定（414 行）：`ACCT-EXPIRAION-DATE >= DALYTRAN-ORIG-TS(1:10)`（先頭 10 桁の日付比較）を
  満たさなければ理由 103。
- 口座残高更新（547-552 行）：`ACCT-CURR-BAL = ACCT-CURR-BAL + DALYTRAN-AMT`。
  金額が 0 以上なら `ACCT-CURR-CYC-CREDIT` に加算、負なら `ACCT-CURR-CYC-DEBIT` に加算。
- カテゴリ残高更新（508・527 行）：`TRAN-CAT-BAL = TRAN-CAT-BAL + DALYTRAN-AMT`。

### (10) 例外処理・異常系
- ファイル入出力エラー時の共通パターン（`APPL-RESULT` 判定 →
  `9910-DISPLAY-IO-STATUS` → `9999-ABEND-PROGRAM`）。
- `CEE3ABD`（ABCODE=999）による強制終了（707-711 行）。
- TCATBAL の `INVALID KEY`／ステータス `23`（レコード未存在）を新規作成に振り替える
  分岐（474-499 行）。
- EOF 判定（ステータス `10` → `APPL-EOF` → `END-OF-FILE='Y'`、345-369 行）。

### (11) 制約事項・モダナイゼーション観点（推測を含む）
以下を「（推測）」を明示したうえで論じること。
- VSAM（KSDS）→ RDB（DB2/PostgreSQL 等）移行時の考慮（索引キー、レコード長、
  パック／表示数値の扱い、`ACCT-EXPIRAION-DATE` の日付文字列比較のロジック）。
- バッチ処理の Web/API 化・イベント駆動化の観点（1 件ずつの READ→UPDATE ループ、
  トランザクション境界、リトライ・冪等性）。
- ハードコードされた理由コード・メッセージの外部化、拡張余地コメント
  （`* ADD MORE VALIDATIONS HERE`）への対応。
- 文字コード（EBCDIC）・GDG（`DALYREJS(+1)`）依存への配慮。

## 3. 記述上の注意
- 表・箇条書き・コードブロック・mermaid を適切に用い、読みやすく構成すること。
- COBOL の項目名・段落名・理由コードは原文どおり（綴り・大文字含む）記載すること
  （例：`ACCT-EXPIRAION-DATE` は原文の綴りのまま）。
- 事実と推測を必ず区別し、推測には根拠と併せて「（推測）」を付すこと。

## 4. 出力ファイルの推奨保存先
生成した詳細設計書は、次のパスに保存することを推奨する：

```
docs/詳細設計書_CBTRN02C.md
```

＝＝＝ プロンプトここまで ＝＝＝
