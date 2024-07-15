# claspについての勉強メモ

## claspとは
GASをgitみたいに管理できる。

claspを使用すると、自分のコンピュータ上でコードを書き、完了したらApps Scriptにアップロードできます。また、既存のApps Scriptプロジェクトをダウンロードしてローカルで編集することも可能です。コードがローカルにある場合、お気に入りの開発ツール（例：git）を使用してApps Scriptプロジェクトを作業できます。

## How to Install
clasp は Node.js >= v6.0.0 が必要なので。必要に応じてインストールやアップデートを行う。

<https://nodejs.org/en/download/package-manager>

```sh
sudo npm install n -g
sudo n latest
```

nodeをインストールしたら、claspを入れる。

```sh
npm i @google/clasp -g
```

clasp をインストールしたら、googleアカウントでログインする。
Google App Script プロジェクトを作成するアカウントでログインすること。

```sh
clasp login
```

## How to Use
terminalから スクリプトファイルを作成するときは、リポジトリ作成 → `git init` → `git remote add origin https://github.com/username/your-repo-name.git`するような感覚で進める。<br>※git init と git remote add は同時に行われる感じかも

このとき、MyDriveに単体でAppScriptファイルが作成される。

```sh
# リポジトリを作成する
mkdir example_file;
cd example_file;
# MyDriveにAppScriptファイルを作成する
# -- title "hoge" がタイトルになる
clasp create --title "hoge"  --type standalone;
```

<https://script.google.com/home/usersettings>からGoogle Apps ScriptのAPIを有効化しておく必要がある。有効にしていないとき、`clasp create`したときに怒られる。



一方で、`git clone`のような操作もできる。googleスプレッドシートに紐づいたapp scriptのようにしたい場合は特に有効である。<br>
`https://script.google.com/home/projects/script_id/settings`から`script_id`をコピーして、terminalで`clasp clone`する。

```sh
# リポジトリを作成する
mkdir example_file;
cd example_file;
clasp clone script_od
```

## Appendix
### コーディング時に、コード補完されるようになるパッケージ

```sh
npm install @types/google-apps-script
```

### 注意点：GASプロジェクトごとにpackage.jsonが必要
ひとつのリポジトリ（`example_file`）で複数のGASプロジェクトを管理したい場合、GASプロジェクトごとに`package.json`を作成する必要がある

...らしいのだが、そもそも下記のような状態にする方法がわからない

```txt
./
|--GASプロジェクト1
|  |--.clasp.json
|  |--appscript.json
|  |--code.ts
|  |--package.json ←空のjsonを作る
|
|--GASプロジェクト2
|  |--.clasp.json
|  |--appscript.json
|  |--code.ts
|  |--package.json ←空のjsonを作る
|
|--node_modules
|--.gitignore
|--package-lock.json
|--package.json ←ちゃんと記述する
```


[参考：GASをGit管理するならGithubアシスタントとclaspどっちを使うか？](https://zenn.dev/rescuenow/articles/936a1f4fb4d889#gas%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%94%E3%81%A8%E3%81%ABpackage.json%E3%81%8C%E5%BF%85%E8%A6%81)


### 注意点：APIキーなどの機密情報をGitに載せることの問題点と対策について
APIキー（e.g. slackのwebhook url）などの機密情報をgitに載せずに環境変数に保存すること。
APIキーをコード内に直接記述せずに安全に管理できます。また、プロジェクトの設定を変更する際も、コードを修正せずにAPIキーを更新できます。

以下の手順で設定できます：

1. 画面左上の「設定」アイコン（歯車マーク）をクリックします。
2. サイドパネルで「プロジェクト設定」を選択します。
3. 「スクリプト プロパティ」セクションを探します。
4. 「行を追加」ボタンをクリックして、新しいプロパティを追加します。`
5. 「プロパティ」列にキー名（例：API_KEY）を、「値」列に対応する値を入力します。
6. 「保存」をクリックして変更を適用します。

これらの手順で、スクリプトプロパティを設定できる。コード内でプロパティを取得する方法は以下。

```js
const API_KEY = PropertiesService.getScriptProperties().getProperty('API_KEY');
```

### なぜ良くないか
- a. セキュリティリスク
  - リポジトリが公開されると、誰でも機密情報にアクセスできてしまう
  - 悪用されるリスクが高まる（不正アクセス、サービスの悪用など）
- b. 機密性の喪失
  - 一度Gitにコミットすると、履歴に残り続ける
  - 削除しても過去のコミットから復元可能
- c. コンプライアンス違反
  - 多くの組織やサービスで、APIキーの公開は利用規約違反となる
  - 法的問題や、サービスの利用停止につながる可能性がある
- d. 環境依存の問題
  - 開発、テスト、本番環境で異なる設定が必要な場合に柔軟性が失われる

### 対策
- 環境変数の使用
  - APIキーを環境変数として設定し、コード内で参照する
- 設定ファイルの分離
  - 機密情報を別のファイル（例：`config.json`）に保存し、.gitignoreに追加する
  - テンプレートファイル（例：`config.example.json`）をリポジトリに含め、実際の値は開発者が個別に設定する
- .gitignoreの適切な設定
  - 機密情報を含むファイルを`.gitignore`に追加し、バージョン管理から除外する
- Git履歴のクリーンアップ
  - 誤ってコミットした場合、`git filter-branch`や`BFG Repo-Cleaner`を使用して履歴から完全に削除する

## 参考（ここで勉強した）
- <https://github.com/google/clasp>
- [clasp - The Apps Script CLI](https://codelabs.developers.google.com/codelabs/clasp?authuser=0#0)
- [Google Apps ScriptをGitHubで管理するには](https://zenn.dev/game8_blog/articles/db189ba79d1750)
- [GASをGit管理するならGithubアシスタントとclaspどっちを使うか？](https://zenn.dev/rescuenow/articles/936a1f4fb4d889)
