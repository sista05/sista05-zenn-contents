
## 時限でジョブを流してくれます

今回は、salesforceのデータを取ってくるケースがあるのでそうしました。


なんと、dbtとsalesforceがコラボってます。
https://hub.getdbt.com/fivetran/salesforce_source/latest/



パッケージのインストール方法はここにございますよ。
https://dev.classmethod.jp/articles/dbt-package-dbt-utils/




dbt環境作成には玉井さんの記事がいいですよ
https://dev.classmethod.jp/articles/dbt-continuous-integration/

dbt docsを簡略化、日本語化する方法
https://dev.classmethod.jp/articles/dbt-documentation/

1時間毎にデータを流すようにしました。




deployment.ymlが要りますよ。


プロジェクトのあるdbtのディレクトリを指定するよ。


git@じゃなきゃダメでしたよ。


## 構造がわかりやすいです


jinja
jinja、ansibleでもよく使ってました。







## データもドキュメント管理します





`dbt document`コマンドを打つと


:::message
メッセージをここに
:::

:::details タイトル
表示したい内容
:::


:::message
**Point !!**
プロジェクトにはdbt_project.ymlと**deployment.yml**ファイルが必須。
:::

## テストもバッチリです


<!--
## 落としどころ
時間がなければ、ここは後日にする、


dbtはgit管理することが必須なので、コードの正確性については、

git管理するときは、GithubActionsなどと連携されて運用した方が良さそうですね。
また、それであれば個人用の検証環境にも適用できそうだ(?)
今の所、BQのactionsはあるが他のはないので作りたい。
これは、後日もし作る機会があれば載せたいと考えています。
-->

:::details dbtとFivetran、そしてdbt transformationsをご存じない方向けの説明
先に述べたfivetranが提供しているsalesforce_source packageでは、
非エンジニアも参照することがあるので、salesforce_source packageの日本語forkを作成中
- available
- bb

dbtの例と、dbt Cloudの例を挙げておく
これら周辺についてはQiitaにもまとめているのでぜひぜひご覧ください！


- 現時点ではジョブスケジューラの
  ような機能を提供
- 細かく設定するには、実装側で
  いろいろ工夫が必要
- 実装は進められているので、
  それまで代わりの機能は
  こちらでまかなって待とう

:::
