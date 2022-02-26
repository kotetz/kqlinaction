# KQL in Action

## はじめに

### 使い方
KQL を初めて使う方が KQL の書き方を身に着けるための演習用の問題集と回答、解説です。  
実際の KQL を使う場面で頻繁に出現するコマンド（演算子、関数）を様々な角度からいろいろな使い方で実行することで、手でコマンドを覚えてもらうような利用方法を想定しています。

### 用語
KQL では他のプログラム言語と同じように、各言語要素に対して名前がつけられています。この名前を覚えることは必須ではありませんが、学習効率を高めるためには意識しておいたほうが良い要素です。単に全てを「コマンド」とひとくくりにするのではなく、現在自分はどの要素が使っているのかを意識しながら KQL を書くと仕組みへの理解が速やかに深まります。
- 演算子、オペレーター： 意味は同じです。
- テーブル演算子： 最も多く使われるオペレーターです。テーブルの入力を受け付け、テーブルを出力します。処理の最初やパイプラインの次に現れる要素はテーブル演算子です。
- スカラー演算子： 主にテーブル オペレーターの中で使われるオペレーターです。数値演算子、論理演算子、文字列演算子などがあり、テーブルの各行の特定の列の値などデータ処理を行います。後述の関数と区別がつきにくいのですが、 between と not-between はスカラー演算子です。
- 関数、ファンクション： 意味は同じで特定の処理結果を返します。入力値に対して処理を実施するものや、入力値を取らずに値を返すものがあります。スカラー演算子と似ているので注意してください。

```javascript
<Table>
| <Table Operator> <function>()
| <Table Operator> <value> <Operator> <value>
```


### リンク

- [Log Analytics DEMO workspace](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade)  
サンプル データが用意されているデモ ワークスペースです。このページの KQL はこのデータに対して実行することを想定しています。

- [Kusto Query Language](https://docs.microsoft.com/ja-jp/azure/data-explorer/kusto/query/)  
Azure Data Explorer の KQL のリファレンスです。Log Analytics ではこのなかの一部が実装されています。



<details><summary>Template</summary>

```javascript
KQL Here
```
解説 :  

</details>

## 基本の KQL

### 目的の情報がどこにあるかを探す
主に使用するオペレーター
- [search](https://docs.microsoft.com/azure/data-explorer/kusto/query/searchoperator?pivots=azuredataexplorer)
- [distinct](https://docs.microsoft.com/azure/data-explorer/kusto/query/distinctoperator)


<details><summary>contoso を含むレコードの検索を行ってください</summary>

```javascript
search "contoso"
```
解説：  
search オペレーターはテーブルを指定しない場合、ワークスペース上の全てのテーブルに対して検索を行います。一般的な KQL ではテーブルを指定してデータを取得しますが、目的のデータがどのテーブルに含まれているかわからない場合や、そもそもデータがワークスペースに存在するかどうかわからない場合などはsearch オペレーターを使用することができます。  
大量のデータを検索する場合パフォーマンス上不利になるため、定期的に実行されるクエリなどでの多用は避けてください。
また、Kusto の実行環境によっては使えません。例えば Log Analytics では使うことができますが、Azure Resource Graph では使うことができません。
</details>

<details><summary>contoso と retail を両方含むレコードの検索を行ってください</summary>

```javascript
// Prefered
search "contoso" and "retail"

// NOT Prefered
// search "contoso" | search "retail"
// search "contoso" | where Name contains "retail"

```
解説 :  
search オペレーターは多くの結果を生成する可能性があり、パフォーマンスに影響を与えます。複数の条件を指定する必要がある場合には、１つの search オペレーターの条件に含めるようにしてください。複数の search オペレーターをパイプしたり、search オペレーターの結果を where でフィルタするような処理は、中間処理のために前の search オペレーターによって大きな結果セットが生成されます。これに対して複数の条件を指定した１つの search オペレーターはそれよりも小さな結果セットを生成します。

</details>


<details><summary>大文字から始まる Contoso を含むレコードを表示してください</summary>

```javascript
search kind=case_sensitive "Contoso"
```
解説 :  
search オペレーターは既定で大文字小文字を区別せずに検索を行いますが、kind 引数で明示的に動作を指定することができます。case_sensitive は大文字小文字を区別する指定です。

</details>

<details><summary>Perf テーブルから contosohotels を含むレコードを表示してください</summary>

```javascript
search in (Perf) "contosohotels"

// able to search piped Table
Perf
| search "contosohotels"
```
解説 :  
search  オペレーターはテーブルを指定して検索を行うことができます。in 引数の値で検索対象とするテーブルを指定します。search オペレーターはパイプから受け取ったテーブルに対して検索を行うこともできます。

</details>

<details><summary>SecurityEvent テーブルから contosohotels を含むレコードを表示してください</summary>

```javascript
search in (SecurityEvent) "contosohotels"

// able to search piped Table
SecurityEvent
| search "contosohotels"
```
解説 :  
この例は１つ前とほとんど同じです。同じ文字列を持つレコードが複数のテーブルに存在する可能性があることを理解するために用意しています。

</details>

<details><summary>Alert テーブルと SecurityAlert テーブルから  contosohotels を含むレコードを表示してください</summary>

```javascript
search in (Alert,SecurityAlert) "contosohotels"

// Other expression
// Need to combine multiple tables
Alert
| union SecurityAlert
| search "contosohotels"
```
解説 :  
in 引数では複数のテーブルを指定することができます。この例では関連しそうな情報をもつテーブルをまたいで検索を行います。追加の例は KQL では複数のテーブルをパイプラインから渡すことができないため、予め複数のテーブルを union で接続しています。

</details>


<details><summary> <strong>[重要]ワークスペースにあるテーブルの一覧を表示してください</strong> </summary>

```javascript
search *
| distinct $table
```
解説 :  
KQL では通常テーブルを指定して検索を行うため、ワークスペースにどのようなテーブルがあるかを知る必要があります。
search オペレーターの検索結果には所属するテーブルを表す $table カラムが含まれるため、distinct オペレーターで一意の値を抽出することで、ワークスペースに含まれている全てのテーブルを一覧することができます。  

存在するテーブルは UI から確認するのは非常に時間がかかるため、簡単にテーブル名だけを確認する方法としてこの例を挙げています。

</details>
  
### 必要なレコードを表示する
主に使用するオペレーター
- [take / limit](https://docs.microsoft.com/azure/data-explorer/kusto/query/takeoperator)
- top
- sort
- where


### データを加工する
- extend
- project
- project-away


### データを集計する
主に使用するオペレーター
- count
- summarize

### データをフォーマットする
- parse
- parse_xml()
- parse_json()
- parse_csv()





<details><summary>SecurityEvent テーブルの、EventSourceNameが Microsoft-Windows-AppLocker であるデータに含まれている TargetLogonId の一覧を 各コンピューターごとに表示してください</summary>

```javascript
SecurityEvent
| where EventSourceName == "Microsoft-Windows-AppLocker"
| extend RawEvent = parse_xml(EventData)
| project Computer,tostring(tagetId=RawEvent.UserData.RuleAndFileData.TargetLogonId)
| distinct Computer,tagetId 
| summarize make_list(tagetId) by Computer
```
解説 :  
SecurityEvent は LogAnalytics エージェントが収集するコンピューターのイベント ログです。イベントログによっては元の XML の情報が EventData カラムに含まれており、parse_xml で解析し、中のデータを扱うことができます。
分析されたデータは型が不明な dynamic 型になるため、データを利用するためには明示的に型を指定する必要があります。この例では tostring 関数を使用して文字列に変換しています。   
集計関数 make_list は指定したカラムからリストを作成します。今回の例の TargetLogonId は重複するものが多く、イベント自体の数も多いため事前に distinct で重複の除去を行っています。  
parse_xml は負荷の高い処理なので、処理前のデータは where などでフィルタし、処理データを可能な限り少なくすることが推奨です。   


</details>


### データを可視化する
主に使用するオペレーター
- render


### 複数のテーブルを検索する
主に使用するオペレーター
- union
- [join](https://docs.microsoft.com/azure/data-explorer/kusto/query/joinoperator?pivots=azuremonitor)


<details><summary>AzureDiagnostics と AzureMetrics テーブルから各リソースからどのような Category で診断ログが送られているか、どのような MetricName を使用できるかを１つの結果セットで表示してください。</summary>

```javascript
AzureDiagnostics
| distinct Resource,Category
|union (
    AzureMetrics
    | distinct Resource,MetricName
)
```
解説 :  
union オペレーターは 2 つのテーブルを縦に結合します。2 つのテーブルが同じカラムを含む場合そのカラムは連結されます。既定では双方のテーブルに含まれる全てのカラムが結果セットに含まれ、元々列が存在しなかったテーブルのセルには null が設定されます。この動作は kind 引数で変更することができ、kind=inner を設定すると結果セットは共通する列のみで作成されます。

</details>



<details><summary>AzureMetrics で各コンピューターが過去 24 時間に記録した Percentage CPU, Data Disk Read Bytes/sec と Data Disk Write Bytes/sec の最大値を 1 つの結果セットで表示してください。</summary>

```javascript
AzureMetrics
| where MetricName == "Percentage CPU" 
| summarize  CPUMax = max(Maximum) by  bin(TimeGenerated,1h),Resource 
| join (
    AzureMetrics
    | where MetricName == "Data Disk Read Bytes/sec" 
    | summarize  DiskReadMax = max(Maximum) by  bin(TimeGenerated,1h),Resource 
) on Resource,TimeGenerated
| join (
    AzureMetrics
    | where MetricName == "Data Disk Write Bytes/sec" 
    | summarize  DiskWriteMax = max(Maximum) by  bin(TimeGenerated,1h),Resource 
) on Resource,TimeGenerated
| project TimeGenerated,Resource,CPUMax,DiskReadMax,DiskWriteMax
```
解説 :  
join オペレーターはテーブルを横に連結し、

</details>





### あるテーブルのデータを他の検索に利用する

### 文字列の加工
--


### レコードの中身を解析する
主に使用するオペレーター
- parse
- parse-where

### IP アドレスの処理


### Azure リソースをクエリする


<!--

Metrix  | iops | summarize max() by computer | render

Metrix  | memory | summarize max() by computer | render

Metrix | cpu | summarize avg*( by cmoputer)

evenglog | logonfailure | summarize count() by user

activytylog | operation | summarize count() by operation,datetime


時間経過によるディスクIOの変化
時間経過によるネットワークIOの変化


配列で指定したコンピューターに関する情報
配列で指定したユーザーがログインしたコンピューター


AVD ホストが時間当たり何台必要かを表示する

リソースグループ / 管理ユーザーごとにセキュアスコアの平均値を表示


特定のテーブルからコンピュータのリストを作成し、そのコンピューターのログを表示する



---sec




特定のグループメンバーの
・ログイン
・リソースの変更
・権限の使用→過去１か月にないオペレーションを行った場合にはアラート


マルウェアの検出イベント（サンプル）

ユーザーのログインアノマリ
ユーザーのファイルしようアノマリ


起動したExeのうち、他のコンピューターにはないユニークなもの
ユーザープロファイルから起動されているプログラム


リソースごと

DNS で名前解決を行わないネットワーク通信


SQL のクエリ一覧

IIS のアクセスログ

AppGW と IIS ログの紐づけ


AppGW / AzureFirewll / DDoS protection に特徴的なログの


ポスチャが悪化した場合に検出（先週に比べて１０％低下するとアラート、など）
セキュリティ更新プログラム以外でポスチャが悪化した場合にはアラート


 -->

 