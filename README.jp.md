# GTM ガイドライン      

このガイドラインは、[Google Tag Manager（GTM）](https://www.google.com/analytics/tag-manager/)ユーザーが実際に GTM を使用してトラッキング設定を実装する前に、認識、理解する必要事項を記載しています。    
GTM は 誰でも使用できるシンプルな WYSIWYG トラッキングエディタとして提供されていると理解されていますが、GTM はユーザーにウェブサイト全体と、そのデータ、機能への無制限のアクセスを提供する強力なツールです。 大きな力があれば大きな責任が生まれます。このガイドラインは、複雑なトラッキング設定の設定指針を持つのに役立つはずです。  

## JavaScript

Google Tag Managerの JavaScript は、意図しない予期しない方法で Web サイトの JavaScript と干渉する可能性があります。 GTM の実装がサイトの円滑な運営に与える影響が少ないことを確認するために、次の基本的なガイドラインに従ってください。           

### 匿名関数により、カスタム JavaScript を分離する      

GTM の JS はグローバル空間を汚染してはいけません。 コードを匿名関数でラップすることで、必ず変数スコープを制限してください。          

    // 良い例
    (function() {
        var foo = 'bar'
    })()

    // 悪い例
    var foo = 'bar'

ページビューの処理期間で変数情報を保持する必要がある場合、または複数のタグから同じ変数にアクセスする必要がある場合は、dataLayer 変数を使用します。     

    // 1st tag
    (function() {
        dataLayer.push({
            foo: 'bar'
        })
    })()

    // 2nd tag
    console.log({{dl.foo}})

### カスタム JavaScript変数 に関数を設定し、コードを再利用する

単純な文字列または整数の代わりに、 関数自体を GTM 変数の結果として関数を返すことができます。      

    // Custom JavaScript variable: js.double
    function () {
        var doubleSize = function(number) {
            return number * 2
        }
        return doubleSize
    }

無名外部関数は他の関数、この例では `doubleSize` を返します。      
その後、他のJavaScript ベースの GTM タグと変数の中で関数として{{js.double}}を使用できます。     

    {{js.double}}(2)
    // Output: 4
	
### ウェブサイトのブラウザサポートガイドラインに従う      

あなたのウェブサイト/あなたの会社が公式にサポートしているブラウザを把握してください。 残念ながら、まだ普及していないブラウザではサポートされていない JavaScript 機能がたくさんあります。 フォールバックやプログラムによるブラウザサポートの事前チェックを行わずにこれらを実装すると、トラッキング設定や Web サイトの機能自体が損なわれる可能性があります。   

### 組み込みの機能 で十分な場合は、カスタムJavaScriptを使用しない    
カスタム JavaScript は、Google Tag Manager の組み込みの機能よりもエラーが発生しやすいものです。 可能であれば、JavaScript の代わりに組み込みのタグテンプレートを使用してください。

* トラッキングコードの実装を要求された場合は、GTM ネイティブのタグテンプレートがあるかどうかを確認する。   
* if…else 構文の代わりにルックアップテーブル、または正規表現の表を使う。       

### すべての機能を文書化する      

JavaScript 関数を書くのであれば、[JSDoc3構文を使って](http://usejsdoc.org/about-getting-started.html)それらを文書化してください。    

	/**
	 * Increases the supplied number by one
	 * @param  {number} number The number to increase
	 * @return {number}        The number increased by 1
	 */
	function increaseByOne(number) {
		return number + 1;
	}

### JS エラーレポート を使用する    

すべてのデバイス上のすべてのユーザーに、すべてのコードを問題なく動作させることは困難です。 考えられるすべての設定でGTMの実装をテストすることはできません。  
テストが困難な状況下でのみ発生するバグを検出するには、Sentry、New Relic 等の JavaScript のエラー報告ツールを使用します。    

JavaScriptエラーが発生するたびに検出され、問題の根本原因を評価できるデータベースに保存されます。
JavaScriptエラー検出に自前のエラー報告ツールを使用しないでください、それらはあなたのビジネスとは関係ありません。         

### 本番環境でconsole.logを使用しない

本番環境では `console.log`を使わないでください。 不必要にあなたのコードとユーザーのブラウザコンソールの両方を雑然とさせます。    
プレビューモードでの開発中は、デバッグ目的で自由に使用してください。         

## セキュリティ    

### サブリソース完全性 Subresource Integrity (SRI) を使用する   

サードパーティのスクリプトを実装する場合は、気付かないうちにリソースが変更されないようにサブリソース整合性ハッシュを使用してください。     
詳細については https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity を参照してください。   

### コンテンツセキュリティポリシー（CSP）を要求する     

コンテンツセキュリティポリシーでは、ブラウザがスクリプト、フォント、スタイルシートなどをロードできるホストを定義できます。 詳細については https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP を参照してください。    
CSP はあなたのウェブサイト全体とフロントエンドの開発に大きな技術的影響を与えるので、すべての開発者が CSPの設定に関与する必要があります。    
CSP はあなたが承認しなかった第三者の不正なスクリプトが突然実行されるのを防ぐことができます。        

### 他の開発者に相談する         

自分が何をしているのかよくわからない場合は、担当の1人以上のプロの JavaScript 開発者、または Web サイトで JavaScript をコーディングしている人に相談してください。 


## dataLayer 

### ツール固有の dataLayer の命名と構造を使用しない    

dataLayer 変数の名前は常にできるだけわかりやすいものにしてください。
たとえ便利に見えるかもしれませんが、ツール固有のキーを使用しないでください。 ウェブサイトからツールロジックを分離することは、Google Tag Manager を使用することの最も重要な目標の1つです。      

    // 良い例
    dataLayer.push({
        loginStatus: true
    })

    // 悪い例
    dataLayer.push({
        dimension17: 1
    })

必要に応じて、カスタム JavaScript 変数を使用して個々のツールのデータを前処理します。    

### 個人を特定できる情報（PII）を dataLayer に設定しない     

第三者が個人またはデバイスを識別するために使用できる情報は、dataLayer で使用しないでください。
それは電子メールと IP アドレスを含みます。

あなただけが個人を特定できるのであれば、あなたの所有するユーザー ID を設定するのは問題ありません。しかし、それだとしてもあなたは最初にプライバシーの専門家に相談するべきです。      

### Google Tag Manager から dataLayer に値を設定しない    

ほとんどの場合、GTM の外部と内部の両方からイベントが発生する可能性があるため混乱する可能性があるため、Google Tag Manager 内部のカスタム HTML / JS コードで `dataLayer.push`を使用することはお勧めできません。    

タグ間の依存関係を実装しようとしている場合、又は、競争条件（タグをある順序で処理したいが、その順序を保証することができない状況）を避けたい場合には、可能な限り組み込みの[タグシーケンス](https://www.simoahava.com/analytics/understanding-tag-sequencing-in-google-tag-manager/)機能を使用してください。     

## アクセス管理    

コンテナの公開アクセス権限を持つユーザをできるだけ少なくします。 公開権限を必要としない人は読み取りアクセス権を設定すべきです。四半期ごとにユーザをより制限されたアクセスレベルに移動できるか、完全に削除できるかを確認します。       

## ワークフロー

### ワークスペース

緊急の事態に備えて、「デフォルトワークスペース」は常に空のままにしてください。 突然バグ修正が必要になっても、以下を行う必要はありません。   
a) 空のワークスペースが残っていないという理由だけで、すでに行われている作業を犠牲にする。      
b) バグ修正、動作確認、公開のため他の変更が行われたワークスペースを使用する。     

１つの作業ごとに 1 つのワークスペースを使用します。1 つのワークスペースでさまざまな問題を解決しないでください。     

### すべての設定に定数を使う      

複数のタグが同じ設定値（トラッキングアカウント ID など）を使用したらすぐに、タイプが "Constant" の GTM 変数を作成し、それをタグから参照します。    

### 必要な組み込み変数だけを有効にします     

積極的に使用しているものだけをアクティブにしてください。 非アクティブな組み込み変数の値を計算する必要がないため、リソースを節約できます。     

### バージョン名と Note(注意書き)    

* すべてのバージョンに名前を付けます。バージョンの変更は、名前だけで直感的に明らかになります。技術的な詳細ではなく、変更による「ビジネスへの影響」に注目して名前を記載してください。 JIRA などのチケットまたはタスクシステムを使用している場合は、バージョン名にチケットID を含めます。     

* 各バージョンのバージョンノートを書きます。 ノートには以下の内容を記載すべきです。    
    * 誰が変更を要求したのか?       
    * リリース前のトラッキング動作は正常か? 今の動作は正常か?      
    * この GTM の実装が依存している Web サイトの変更はあったか？ もしそうなら、どんな変更か？        


### フォルダ   

フォルダーを使用して、目的別に要素をグループ化します。

| Type          | Description                                                                 |
| :------------ | :-------------------------------------------------------------------------- |
| Ad Tech       | 広告技術、AdformやCriteoなどのサードパーティトラフィックパートナー |
| Web Analytics | Google AnalyticsやWebtrekkなどのウェブ解析ソフトウェア          |
| Utilities     | さまざまな目的に使用できるヘルパー関数と小さなスクリプト |

## ネーミング

### タグ

### トリガー      

`Type` `Description`

| Type       | Description                                                                                                                                   |
| :--------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| Conversion | 即座にビジネスにプラスの影響を与えるユーザーアクション                                                                                         |
| Pageview   | ページビュー 実際のフルページロードなのか、それともSPAの新しい画面なのか区別しない |
| Event      | 上記の種類のいずれにも一致しないカスタムGTMイベント                                                                        |
| State      | ウェブサイトの特定の状態を説明し、通常はトリガーとしてではなく例外として使用される                                           |

### 変数

`type.[tool.]name`

`type`は利用可能なさまざまな種類のGoogle Tag Manager変数を指します。 変数の型は `{{myVariable}}`のような参照を見てもすぐには見えないので、データがどこから来たのかを識別するのは困難です。    

| Variable type             | Type value  |
| :------------------------ | :---------- |
| HTTP Referrer             |  ref        |
| URL                       | url         |
| 1st Party Cookie          | cookie      |
| Custom JavaScript         | js          |
| Data Layer Variable       | dl          |
| JavaScript Variable       | global      |
| Auto-Event Variable       | auto        |
| DOM Element               | dom         |
| Element Visibility        | vis         |
| Constant                  | const       |
| Custom Event              | custom      |
| Environment Name          | env         |
| Google Analytics Settings | settings    |
| Lookup Table              | lookup      |
| Random Number             | rnd         |
| RegEx Table               | regexlookup |
| Container ID              | gtm         |
| Container Version Number  | gtm         |
| Debug Mode                | debug       |

`tool`は、1つのツールにのみ関連している（そして関連することになる）変数のオプションの部分です。 例えば、定数としての特定のサードパーティツールの1つのタグのためにデータを特定のフォーマット提供する JavaScript のデータ変換スクリプトに付与します。    
この場合の「tool」は、「adform」、「hotjar」、「kissmetrics」のような製品、または共通の略語「eec」を持つ Google Analytics Enhanced eコマースのような普遍的な概念のいずれかです。     

`name`はこの変数を説明する名前です。 短くしても `type`や` tool`と組み合わせると、それが何を表しているのかすぐに理解できるはずです。    

#### 変数名の例    

* Adform 発行者 ID： `const.adform.id`    
* ヤフーだけに関連する特定のフォーマットで商品データを提供する JavaScript のデータ変換スクリプト: `js.yahoo.productData`
