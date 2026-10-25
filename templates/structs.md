# 抽象クラス

役割ごとの関数を、役割ごとの抽象クラスとして定めたもの。

抽象クラスは関数の入力と出力だけを定め、各関数の内部の判断過程は現場に委ねる。

記述は次の四種からなる。

1. クラス定義: 関数の入力と出力に現れるクラス
2. 役割の関数: 人が判断を行う関数
3. 機械処理の関数: 人の判断を含まない関数
4. 違反規則: 関数定義との照合で検出できる違反

## 1. クラス定義

グループは進行の順に並べ、各グループの中では参照されるクラスを先に置く。

### 発注と納品

    class Fabless  // 設計側の組織

    class Foundry  // 製造側の組織

    class MoldID  // モールドの一意な識別子

    class TestCase  // この値を入れたらこうなる
        テストケースID
        シナリオ
        事前条件
        テストデータ
        期待結果

    class Mold  // 結果が出る前に決めておく
        MoldID
        リビジョン
        発行日
        期限
        Fabless
        Foundry
        ステータス: 作成中 | 発送済 | 製造中 | 納品済 | 差し戻し中 | 受入完了
        取り下げ: なし | 取り下げ日時と理由
        [TestCase]
        参照: [MoldID | 参照文書]

    class Artifact  // ファウンドリのソースコード

    class TestCaseResponse  // TestCase 一件への応答
        テストケースID
        判定: 満たしている | 満たしていない | 適用外
        根拠: 該当箇所 | 適用外の根拠

    class RequirementProof  // Mold の各 TestCase への応答チェックリスト
        MoldID
        リビジョン
        [TestCaseResponse]

    class LogicProof  // 成果物が仕様通り動くことを、テストを実行した結果で示す
        テストコード
        実行環境
        実行結果

    class Delivery
        Artifact
        RequirementProof
        LogicProof

### ディシジョン

モールドに関する判断は Decision を継承した一件として記録する。判断の別はクラスで表す。

    class Decision  // モールドに関する一件の判断
        MoldID
        リビジョン
        判断者
        判断日時

    class DispatchDecision extends Decision  // 発送

    class AcknowledgeDecision extends Decision  // 請書

    class RejectDecision extends Decision  // 不備返送
        [不備のある項目と理由]

    class ShipDecision extends Decision  // 納品
        Delivery

    class AcceptDecision extends Decision  // 受入完了

    class SendBackDecision extends Decision  // 差し戻し
        [満たしていないテストケースID]
        理由

    class WithdrawDecision extends Decision  // 取り下げ
        理由

    class DecisionLog  // Decision の記録

クラスごとに、判断する側と更新後のステータスが決まる。

| クラス | 判断者 | 更新後のステータス |
|---|---|---|
| DispatchDecision | ファブレス | 発送済 |
| AcknowledgeDecision | ファウンドリ | 製造中 |
| RejectDecision | ファウンドリ | 作成中 |
| ShipDecision | ファウンドリ | 納品済 |
| AcceptDecision | ファブレス | 受入完了 |
| SendBackDecision | ファブレス | 差し戻し中 |
| WithdrawDecision | ファブレス | 変えない |

### 停止要因の申告

    class Blocker  // 製造担当者が自分の作業を進められなくなる原因

    class AndonCall  // アンドンによる申告
        コールID
        申告者ID
        Blocker
        申告日時

    class AndonLog  // 申告と応答の対。応答がまだ入っていない申告は未引き受けとする
        AndonCall
        AndonAcceptance | 未引き受け

### 質問と回答

    class LocalContext  // 質問者が持つ文脈

    class Question  // 質問内容（宛先を含まない）

    class DecisionGround  // 回答の根拠

    class Answer  // オーソリティの回答
        回答内容
        DecisionGround

    class CacheEntry
        CacheEntryID
        Answer
        Authority
        createdAt
        invalidatedAt

    class InvalidReport  // CacheEntry の回答が、質問者の LocalContext に対して無効だという報告
        CacheEntryID
        報告者ID
        報告日時

    class ReQuestion  // 再質問。無効の報告を含めて質問し直す
        Question
        InvalidReport

    class RoutingLog  // 質問ごとのルーティングの経路と回答の記録

### 測定

    class MeasurementSource  // 測定元。カレンダー、チャット、メール、Git、チケット、CI の記録と、本方法論のログ

    class MeasurementTarget  // 測定対象。何を、どの記録から、どの期間で算出するか

    class Measurement
        MeasurementTarget
        値
        期間
        算出日時

    class AlertLevel  // 通知の段階。段階の数と名前は測定対象ごとに決める

    class Threshold
        MeasurementTarget
        閾値
        AlertLevel
        設定者

    class MeasurementLog  // 測定ごとに一件
        Measurement
        [Threshold]
        AlertLevel | None

    class Alert  // 閾値に達した事実
        Measurement
        Threshold
        発出日時

    class AlertRecord
        Alert
        AlertResponse | 未対応

### 通知と応答

発生した事実を配信し、応答が入るまで閉じない。アンドン・ディシジョン・アラートの三系統がこの形をとる。

    class Notification  // 配信の記録
        配信先
        配信日時

    class AndonNotification extends Notification
        AndonCall

    class DecisionNotification extends Notification
        Decision

    class AlertNotification extends Notification
        Alert

    class Response  // 通知を閉じる記録
        応答者
        応答日時

    class AndonAcceptance extends Response  // 引き受け。この記録が入った時点でアンドンコールはクローズする
        コールID

    class AlertResponse extends Response
        Alert の参照

    class DecisionReceipt extends Response  // ディシジョンの受領
        Decision の参照

## 2. 役割の関数

各関数の下に、違反となる条件を書く。

組織（Fabless / Foundry）はクラス定義に置き、人が判断する関数は組織の中の人（FablessMember / FoundryMember）に持たせる。

### ファブレスの中の人

    abstract class FablessMember
        担当者ID
        名前
        Fabless
        draftMold(): Mold
        sendMold(Mold): DispatchDecision
        verifyDelivery(Delivery, Mold): AcceptDecision | SendBackDecision
        discardMold(Mold): WithdrawDecision

- 入力にファウンドリの事情を含まない
- draftMold で作った Mold は sendMold で発送する
- 差し戻しの根拠は Mold のテストケースに限る
- 進行中とは、AcknowledgeDecision が出た後を指す
- RejectDecision を受けたら、不備を直した Mold をリビジョンを上げて出し直す
- 進行中の Mold の要求内容を書き換える関数は持たない。追加要求は draftMold で新規の Mold として発行する
- discardMold は、進行中の Mold がファブレスの事情で不要になった場合に呼ぶ。要求の追加や変更のためには呼ばない
- discardMold を呼んだ後、差し戻しはできない

### ファウンドリの中の人

    abstract class FoundryMember
        担当者ID
        名前
        Foundry
        validateMold(Mold): AcknowledgeDecision | RejectDecision
        implement(Mold): Artifact
        proveRequirement(Artifact, Mold): RequirementProof
        proveLogic(Artifact, Mold): LogicProof
        ship(Artifact, RequirementProof, LogicProof): ShipDecision
        pullAndon(Blocker): AndonCall

- 入力にファブレスの内部事情や上長の意向などの外部要因・人間由来要因を含まない
- 製造に入る前に validateMold でモールドを検証する。必須項目が埋まっているか、各テストケースに期待結果があるかを確かめる
- 検証に通れば AcknowledgeDecision を出す。請書を出すまで implement を呼ばない
- 不備があれば RejectDecision をファブレスに返す
- 入力は Mold に限る。Mold にない要求を実装作業に含めない
- RequirementProof と LogicProof の片方でも欠ければ ship は呼べない
- ship は Delivery を作り、それを持つ ShipDecision を出す
- pullAndon はファウンドリの誰でも呼べる。申告に理由の正当化を要求しない
- pullAndon はアンドンを引くだけであり、宛先を指定しない
- WithdrawDecision を受けたら、停止時点の状態をそのままパレットにまとめる

### 受領者

Decision の通知を受けて応答する者。ファブレスの中の人とファウンドリの中の人の双方がこの役割を持つ。

    abstract class Receiver
        receive(DecisionNotification): DecisionReceipt

- 受領は、受け取ったことの記録に限る。受領を拒む関数は持たない
- 異議は新たな Decision として出す

### 質問者

製造中に、モールド由来の想定していない停止要因（エッジケースなど）に当たった場合の FoundryMember からの質問。FoundryMember を継承する。

    abstract class Questioner extends FoundryMember
        ask(LocalContext): Question
        evaluateCache(CacheEntry, LocalContext): Answer | ReQuestion

- Question に宛先を含めない。ルーターへ送信する
- 回答の有効性は質問者だけが検証できる。CacheEntry の回答が自分の LocalContext に対して有効でなければ、InvalidReport を含めた ReQuestion を出す

### ルーター

    abstract class Router
        route(Question | ReQuestion): CacheEntry | Authority | NULL
        search(Question): Authority | NULL

- route は Question を RoutingLog の過去の質問と照合する。該当がなければ NULL を返す
- route が NULL を返したら、search で答えられるオーソリティを探す。見つかれば転送し、見つからなければ RoutingLog に宛先不明として記録する
- search で見つけたオーソリティは RoutingLog に記録する。以後の照合はこの記録を使う
- 転送先の answer が NULL を返したら、search で別のオーソリティを探す。NULL を返したオーソリティは RoutingLog に記録する
- ReQuestion を受けたら、InvalidReport が指す CacheEntry に invalidatedAt を記録してソフトデリートし、質問をオーソリティへ転送する
- ルーターは回答の内容に触れない。オーソリティからの質問は受け付けない

### オーソリティ

    abstract class Authority
        担当者ID
        名前
        answer(Question): Answer | NULL

- 答えられない場合は NULL を返す。別のオーソリティを名指ししない
- LocalContext の全体を要求しない

### アンドン応答者

アンドンの通知を見て引き受ける FoundryMember。FoundryMember を継承する。

    abstract class AndonResponder extends FoundryMember
        acceptAndon(AndonCall): AndonAcceptance

- AndonAcceptance は引き受けた本人が出す。申告者や第三者が代わりに書かない
- 一つの AndonCall に対する AndonAcceptance は最初の一件で確定する。以降の引き受けは無効とする
- 引き受け後の対応内容や完了時刻は本方法論の記録対象ではない

## 3. 機械処理の関数

人の判断を含まない関数。実行主体は人でなくてよい。

### 通知の配信

    abstract class Broadcaster
        broadcast(通知対象): Notification

    abstract class AndonBroadcaster extends Broadcaster
        broadcast(AndonCall): AndonNotification

    abstract class DecisionBroadcaster extends Broadcaster
        broadcast(Decision): DecisionNotification

    abstract class AlertBroadcaster extends Broadcaster
        broadcast(Alert): AlertNotification

- 対象が記録された時点で配信する。配信に人の判断を含まない
- AndonNotification の配信先はファウンドリ全体とする
- DecisionNotification の配信先は、その Decision を受ける側とする
- 応答が入るまで通知は閉じない

### モールドのステータス更新

    abstract class MoldTracker
        updateStatus(Mold, Decision): Mold

- Decision のクラスに応じて Mold のステータスを更新する
- 更新の対象はステータスと取り下げに限る。要求内容には触れない
- WithdrawDecision が記録された時点で、取り下げ欄に日時と理由を書く

### 測定とアラート

測定は、ログと既存ツールの記録から値を機械的に算出し、事前に決めた閾値と比べて、達したら通知する一連の処理として行う。

    abstract class Monitor
        measure(MeasurementTarget, [MeasurementSource]): Measurement
        checkThreshold(Measurement, [Threshold]): Alert | None
        recordAlert(Alert, AlertResponse | 未対応): AlertRecord

- 一つの測定対象に、AlertLevel の異なる Threshold を複数置いてよい
- 測定ごとに、その時点で適用した Threshold と到達した AlertLevel を MeasurementLog に残す。Threshold の変更はこのログから追える
- checkThreshold は、到達した Threshold のうち AlertLevel が最も高いものについて Alert を出す
- checkThreshold の結果を人が取捨しない
- 対応されなかった Alert も AlertRecord として残す

測定対象の例:

| 測定対象 | 測定元 | 閾値と AlertLevel の例 |
|---|---|---|
| 同期的なやり取りの時間比率 | カレンダー、チャット、メール | 勤務時間に対して 5% で注意、15% で警告 |
| アンドンコールの復帰時間 | AndonLog と Git、チケットの記録 | 申告から作業再開までの時間 |
| 宛先不明の件数 | RoutingLog | 期間あたりに search が NULL を返した件数 |
| 差し戻しの件数 | DecisionLog | モールドあたりの SendBackDecision の件数 |
| 応答のない通知 | Notification と Response | 配信から応答までの時間 |

## 4. 違反規則

関数定義と記録の照合で機械的に検出しうる違反。クラス定義によって起こりえない違反（プルーフを欠く納品など）は載せない。

| 違反 | 検出に使う記録 |
|---|---|
| 進行中の Mold の要求内容の書き換え | Mold のリビジョン履歴 |
| 取り下げ後の差し戻し | DecisionLog の WithdrawDecision と SendBackDecision の判断日時の突き合わせ |
| 応答のない通知の放置 | Notification と Response の突き合わせ |
| ReQuestion によらない CacheEntry の削除 | CacheEntry の invalidatedAt と RoutingLog の ReQuestion の突き合わせ |
| 質問者による宛先指定 | Question の内容 |
| オーソリティによる LocalContext 全体の要求 | RoutingLog の回答内容 |
| pullAndon における宛先指定 | AndonCall の内容 |
| 同一 AndonCall に対する複数の AndonAcceptance | AndonLog の AndonCall と AndonAcceptance の突き合わせ |
| 申告と引き受けの記録の不整合 | AndonLog のコールID の突き合わせ |
| 記録を伴わない呼び出し | 各ログの欠番 |

## 追加の記述形式

役割ごとにクラスや関数を追加する場合、以下の形式で書く。

    class クラス名  // 定義

    abstract class クラス名
        関数名(入力): 出力
    制約:
    - 入力に含めないもの
    - 出力に必ず伴うもの

継承関係を持つ場合は次のように書く。

    class 子クラス名 extends 親クラス名

    abstract class 子クラス名 extends 親クラス名
        関数名(入力): 出力

## 記述例

    abstract class FablessMember
        draftMold(): Mold
        sendMold(Mold): DispatchDecision
        verifyDelivery(Delivery, Mold): AcceptDecision | SendBackDecision

    abstract class FoundryMember
        ship(Artifact, RequirementProof, LogicProof): ShipDecision
        pullAndon(Blocker): AndonCall

    abstract class Questioner extends FoundryMember
        ask(LocalContext): Question

    abstract class Router
        route(Question | ReQuestion): CacheEntry | Authority | NULL
        search(Question): Authority | NULL

    abstract class Authority
        answer(Question): Answer | NULL

    abstract class AndonResponder extends FoundryMember
        acceptAndon(AndonCall): AndonAcceptance
