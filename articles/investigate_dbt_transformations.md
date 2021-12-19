---
title: "Fivetranのdbt transformationsを徹底検証"
emoji: "🔀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Fivetran", "dbt"]
published: true
---

# 🔍この記事の目的

FivetranのData Transformation機能である、**dbt transformations**について調べました。
Fivetranやdbtをご存知ない方向けには、こちら↓の記事がオススメです。

https://qiita.com/sista05/items/cbacfefaa667eb3fe509

企業がdbtを採用するケースが増えてきているようです。dbtの汎用性の高さを考えた時に、すでにデータパイプラインは構築済みでdbtを応用して利用していきたいという企業であれば、そのまま問題なくパイプラインを活かすことができます。ですが、これからデータマネジメントにdbtを用いていきたいといった場合にはdbtのユースケースをしっかり考えて導入を検討したいところです。

dbtのユースケースを伺ってみると、そこまでエンジニア寄りで無い方も使うといったお話も多いようです。dbt Cloudを使えばIDEや実行環境、CIまで提供していて便利なのですがプラン料金がかかります。また、データパイプライン構築済みの環境ではdbt Cloudの導入メリットもそこまで高くなく、ユースケースで使い分けるものと感じました。そこで、Fivetranのdbt transformationsはどのような使用感になるのか検証してみました。

:::message alert
**現在dbt transformationsはβ版であり、この記事は 今後の機能拡張と不足を補う手段について考察したものになります**
:::

# ⚙️dbt transformationsの設定

設定項目を眺めた時に、dbtやdbt Cloudとの違いが浮き彫りになってきます。
設定の詳細はトグル内をご覧いただくとして、違いに焦点を当てていきたいと思います。

:::details 詳細設定

git連携とデフォルトスキーマ項目。デフォルトスキーマは一度設定すると変えられない。

![](/images/setting-1.png =500x)

![](/images/warning-1.png)

クレデンシャル設定。接続DBにより項目内容がまちまちになる。

![](/images/setting-2.png)

アドバンスドオプション。
取得先のブランチ名とリポジトリのトップにdbtプロジェクトが無い場合の設定など。

![](/images/setting-3.png)
:::

### dbt ・ dbt Cloudとの違い

dbtを使用する際にはprofile.ymlというファイルが必要になります。これは接続先DBの設定情報などをまとめたファイルですが、dbt Cloudとdbt transformationsでは設定情報をサービスが管理します。そして**dbt transformationsではこのprofileに相当する設定は一つしかありません。** 設定項目のスキーマも一度設定したら変更することはできず、ターゲットもprodに固定されています(これらの点については後述します)。

また、dbt transformationsではgitリポジトリから実行コードを取得して、dbt transformationsの独自環境内でJobが実行されます。そして、**dbt transformationsではdeployment.ymlに記載したJobという形でしかdbtを実行することができません。**

**dbtサービス三者比較**
|  | dbt  | dbt Cloud | dbt transformations |
| :----: | :----: | :----: | :----:|
| **設定** | profile.yml(複数設定可) | UI(複数設定可) | UI(単数) |
| **実行環境** | ローカル | IDE(Cloud) | Fivetran内環境|
| **dbt実行** | 任意のCLI | 任意のCLI/Job実行 | Job実行 |

# 🧐dbt transformationsの使用感

## UIイメージ
上記でも触れましたが、dbt transformationsではdbt Cloudのように任意のdbtコマンドの発行はできず、deployment.ymlに記載したジョブだけが登録される形になります。つまりジョブスケジューラ機能を提供していると考えていいと思います。

![](/images/job-1.png)
*登録されたJobとその実行結果はUIで確認できる*

:::details 登録されたジョブの実行結果例
実行結果例

![](/images/job-2.png)

ジョブが失敗した様子

![](/images/version_error.png)

:::

ジョブ実行に当たっての注意点ですが、例えばdbt_package.ymlで任意のパッケージをインストールしたい場合、パッケージインストールはdbtのバージョン指定がされているケースが多く、dbtバージョンを変えるには以下の記述が必要になります(明示的な記載がなく軽くハマりました)。Fivetranでは[dbtのサポートバージョン](https://fivetran.com/docs/changelog)が指定されており、サポート対象外のバージョンを利用する場合は`--no-version-check`で一応のerror回避はできそうです。

:::message
**Point !!**
dbtのバージョンを指定する場合は、deployment.ymlにdbtVersionを記載する。
<br>
```yml:development.yml
dbtVersion: 0.21.0

Job:
~以下ジョブ設定に続く~
```

:::

また、一般的なジョブスケジューラを想定した時に、大体最近のUIでは処理のパイプラインを可視化しているのが普通ですが、現状でパイプラインを描画する機能はございません。dbtでは`dbt docs generate`コマンドでテーブル間の依存関係を描画できますが、この点についても後述したいと思います。

<!--とても大きなパイプラインでしたらairflowと組んだり、fivetranのパイプラインと最低でも、CIとdocumentは必要に感じました。-->

![dag.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/244490/62fccf7c-257a-5633-c8c7-041c5d672c07.png)
*dbtではテーブル依存関係のDAGを作成できる*

# 🛠Feature Requestsと追加までの暫定対応

ここまでdbt transformationsの基本機能を見てきました。
Fivetranには[機能追加要望ページ](https://support.fivetran.com/hc/en-us/community/topics/360001909373-Feature-Requests)があります。そちらに挙がっている主な要望やFivetranの公式回答などを踏まえながら、現時点で未実装の機能の代替案も探っていきたいと思います。

## 新しいバージョンのdbtを使わせて欲しい

Fivetranはサポートしているdbtバージョンをアナウンス[^1]しています。
Fivetranとしてはdbtの実行環境を提供している立場もあり、新しいdbtがリリースされてもリグレッションテストが通ったバージョンのdbtしか使わせられないというのは理解できます。

[^1]:https://fivetran.com/docs/transformations/dbt/setup-guide

## Fivetran操作をトリガーとしてdbt実行したい

Fivetranのデータ取得をトリガーにJobをキックする使い方は普通に考えられる機能で、Fivetranは最優先の対応[^2]と公式回答しています。CIについてもリクエストがあり、dbt Cloudが選択肢にないクライアントもいるのでFivetran+dbtの統合で同じようなものを実装してほしいという要望も挙がっているので期待できるかもしれません。

[^2]:https://support.fivetran.com/hc/en-us/community/posts/360051728414-Transformation-dbt-triggered-on-git-push

## 複数の設定項目を持ちたい

一つしか設定を持てない、ターゲットのprod固定、スキーマ固定、などなどの設定縛りに対して多くの改善要望が寄せられているようです。これらに関するFivetranからの明確な回答は少なく対応を決めかねているようですね[^3]。dbt Cloudではしっかり設定を分けられるようにされているので同じようなことは可能なはずです。追って対応を待ちましょう。

代替案としては、Jobに`dbt run --target dev`のように記載することで凌いだり

```sql:query.sql
{%- if target.name == "dev" -%} POC
{%- elif target.name == "qa" -%} QA
# For testing in Fivetran reverting to POC
{%- elif target.name == "prod" -%} PRD
{%- else -%} invalid_database
{%- endif -%}
```

プロジェクト毎に利用するスキーマを変更する。という設定もできなくはなさそうです。

```yaml:dbt_project.yml
models:
  poc_project:
    base:
      +schema: poc_schema
```

[^3]:こちらのようなBugFixのような対応は見つけられました。(https://support.fivetran.com/hc/en-us/community/posts/360051261534-Transformations-Allow-DBT-Role-to-be-configurable)

<!-- https://support.fivetran.com/hc/en-us/community/posts/1500000521381-Transformations-Allow-customization-to-the-generated-profiles-yml-for-Dbt -->

## DBデータ資料をホスティングして欲しい

dbtでは`dbt docs generate`コマンドでDBデータが詰まった静的Webページを生成できます。DBテーブルやカラムの説明、また データの容量やどのテーブルと依存関係があるかのDAG図なども生成されるので、dbt transformationsから閲覧したい要望は自然でしょう。

![日本語](/images/japanese-docs.png =500x)
*salesforce_source packageの日本語版を作成中*

代替案として、PRがMergeされたらGithub ActionsなどのCIでドキュメントを生成するようにして、それをそのままバケットにpushしてしまうような仕組みを作るのはそこまで難しくないと感じました。社内の人だけ閲覧できるようにするならバケットにIP制限をかけたり、よりセキュリティを強くするなら例えばCognito認証をかけたりしてもいいと思います。

<!-- 時間があったら図を載せよ -->

# 📑調査まとめ

![](/images/matome.jpg)
*板書は頭を下げ倒してお願いしました*

考えてみると、dbt Cloudがそれ自体でdbtの機能を包括しているということは、裏を返せばdbt機能をUI上で実現させるとそれだけで一つのサービスとして成立してしまう。ということでもあるので、dbt Cloudと同じような機能をdbt transformationsにも実装して欲しいという要望は要求過多にも感じますし、Fivetranの主機能を措いてdbt機能を充実させるには単純に開発リソースも足りなさそうです。

それでも、Fivetranは積極的にdbtのパッケージをリリースしていることもありdbtとの結びつきが強いので、おそらく継続的に機能拡張は続けられるものと考えられますし、現状で足りていない機能についてもそれを補う手段もございます。引き続き動向は注視し続けていきたいです。
