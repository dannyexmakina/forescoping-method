# チュートリアル: ファブレス編(AI をファウンドリとする)

ファブレスがモールドを作り、AI をファウンドリとして成果物を出させる一周を通す。

## 前提

- 参照: `manuals/fabless.md`, `manuals/foundry.md`, `templates/mold.md`, `templates/structs.md`, `glossary.md`
- ファウンドリ役として、任意の生成 AI を使う

## 発注する機能

システム全体の一機能として、ユーザー登録機能の入力受付部を発注する。テストケースを書き下す。

## 手順 1: モールドを作る

`templates/mold.md` を複製し、`manuals/fabless.md` の 1-1 から 1-3 に沿って埋める。

### 識別

- モールドID: M-001
- リビジョン: 1
- 発行日: 2026-01-15
- 期限: 2026-01-22
- ファブレス: 設計担当A
- ファウンドリ: 生成AI

### テストケース

期待結果の `error` に続く配列は、テストデータの記載順（email, password, display_name, birthdate）に並べる。

| ID | シナリオ | 事前条件 | テストデータ | 期待結果 |
|---|---|---|---|---|
| TC-001 | 全項目が正しい値 | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2000-01-01"}` | `ok` |
| TC-002 | 記号を含むメールアドレス | なし | `{email:"user+tag@example.co.jp", password:"Zxcvbn98", display_name:"a", birthdate:"1990-12-31"}` | `ok` |
| TC-003 | password 8文字（下限） | なし | `{email:"taro@example.com", password:"Abcdefg1", display_name:"山田太郎", birthdate:"2000-01-01"}` | `ok` |
| TC-004 | password 64文字（上限） | なし | `{email:"taro@example.com", password:"Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1", display_name:"山田太郎", birthdate:"2000-01-01"}` | `ok` |
| TC-005 | display_name 1文字（下限） | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"a", birthdate:"2000-01-01"}` | `ok` |
| TC-006 | display_name 32文字（上限） | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"abcdefghijabcdefghijabcdefghijab", birthdate:"2000-01-01"}` | `ok` |
| TC-007 | birthdate が実行日の前日 | システム日付 2026-01-15 | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2026-01-14"}` | `ok` |
| TC-008 | email が形式違反 | なし | `{email:"not-an-email", password:"Abcdef12", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["email"]` |
| TC-009 | password 7文字（下限未満） | なし | `{email:"taro@example.com", password:"Abcdef1", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["password"]` |
| TC-010 | password 65文字（上限超過） | なし | `{email:"taro@example.com", password:"Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1Abcdefg1x", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["password"]` |
| TC-011 | password が英大文字を含まない | なし | `{email:"taro@example.com", password:"abcdef12", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["password"]` |
| TC-012 | password が数字を含まない | なし | `{email:"taro@example.com", password:"Abcdefgh", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["password"]` |
| TC-013 | display_name 33文字（上限超過） | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"abcdefghijabcdefghijabcdefghijabc", birthdate:"2000-01-01"}` | `error, ["display_name"]` |
| TC-014 | birthdate が実行日当日 | システム日付 2026-01-15 | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2026-01-15"}` | `error, ["birthdate"]` |
| TC-015 | birthdate が未来日 | システム日付 2026-01-15 | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2026-01-16"}` | `error, ["birthdate"]` |
| TC-016 | birthdate が実在しない日付 | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2000-02-30"}` | `error, ["birthdate"]` |
| TC-017 | birthdate が形式違反 | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:"2000/01/01"}` | `error, ["birthdate"]` |
| TC-018 | 不正な値が2件 | なし | `{email:"not-an-email", password:"abc", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["email","password"]` |
| TC-019 | 全項目が不正な値 | なし | `{email:"not-an-email", password:"abc", display_name:"abcdefghijabcdefghijabcdefghijabc", birthdate:"2000/01/01"}` | `error, ["email","password","display_name","birthdate"]` |
| TC-020 | email が未入力 | なし | `{email:"", password:"Abcdef12", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["email"]` |
| TC-021 | display_name が未入力 | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"", birthdate:"2000-01-01"}` | `error, ["display_name"]` |
| TC-022 | 未入力が2件 | なし | `{email:"", password:"", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["email","password"]` |
| TC-023 | 全項目が未入力 | なし | `{email:"", password:"", display_name:"", birthdate:""}` | `error, ["email","password","display_name","birthdate"]` |
| TC-024 | 未入力1件と不正な値1件が混在 | なし | `{email:"", password:"abc", display_name:"山田太郎", birthdate:"2000-01-01"}` | `error, ["email"]` |
| TC-025 | 未入力1件と不正な値2件が混在 | なし | `{email:"", password:"abc", display_name:"abcdefghijabcdefghijabcdefghijabc", birthdate:"2000-01-01"}` | `error, ["email"]` |
| TC-026 | 未入力2件と不正な値1件が混在 | なし | `{email:"", password:"", display_name:"abcdefghijabcdefghijabcdefghijabc", birthdate:"2000-01-01"}` | `error, ["email","password"]` |
| TC-027 | 未入力1件、他3件は正しい値 | なし | `{email:"taro@example.com", password:"Abcdef12", display_name:"山田太郎", birthdate:""}` | `error, ["birthdate"]` |

TC-024 から TC-026 が、未入力と不正な値が同時にあるときの扱いを示す。未入力があれば、不正な値の項目は結果に含めない。

### 参照

- なし

## 手順 2: AI に発送する

AI に次を渡す。

- モールド本文
- `manuals/foundry.md`

指示は「ファウンドリ手順書に従って、ソースコード・Requirement Proof・Logic Proof を返してください」の一行。

渡した時点で DispatchDecision を記録する。

## 手順 3: 請書を確かめる

AI が手順書に従えば、まずモールドの検証結果が返る。

- 不備がなければ、AcknowledgeDecision に相当する応答が返る。モールドのリビジョンと一致しているかを見る
- 不備が返った場合は、指摘された欄を直し、リビジョンを上げて出し直す

AI が検証を飛ばしてソースコードを返してきた場合、手順書に従っていない。検証からやり直させる。

## 手順 4: パレットを確認する

次の三点が揃っているかを見る。欠けていれば再生成させる。

- ソースコード
- Requirement Proof
- Logic Proof

揃っていれば ShipDecision を受け取ったものとして扱い、受領を記録する。

## 手順 5: プルーフを検証する

`manuals/fabless.md` の 4-1 から 4-2 に沿って検証する。

- Requirement Proof のモールドIDとリビジョンの一致
- TC-001 から TC-027 のすべてに応答があるか
- 「満たしている」の該当箇所が実際にそう読めるか
- Logic Proof のテストコードを自環境で実行し、全件が pass するか

満たしていないテストケースがあれば SendBackDecision を出す。満たしていないテストケースIDと理由を書く。

## 手順 6: 失敗の切り分け

差し戻しが発生した場合、次を判断する。

- テストケースの記述が曖昧、または網羅が足りなかったのか
- モールドは十分だったが、AI の出力が不正解だったのか

前者はモールドを直しリビジョンを上げて出し直す。後者はモールドを変えず AI に再生成させる。

切り分けの手がかりは Requirement Proof にある。AI が「満たしている」と書いた該当箇所を読み、実際には満たしていない場合、AI がモールドを読み違えている。読み違えの原因がモールドの記述にあれば前者、なければ後者とする。

## この一周で確かめること

- モールドを書くのにかかった時間と、AI が返すまでの時間
- 差し戻しの回数
- 差し戻しの原因が、モールド側とAI側のどちらに多かったか

差し戻しがモールド側に偏る場合、スコープの切り方かテストケースの書き方に手を入れる。AI側に偏る場合、モールドはそのままで、ファウンドリを替えるか再生成の回数を見込む。
