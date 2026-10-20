# AWS AI Chat

AWS Bedrockを使って質問に回答する、React製のチャットアプリです。AWS Amplify Gen 2でメール認証とバックエンドを構成し、チャットの質問をBedrockへ送信します。

## 主な機能

- メールアドレスを使ったサインアップとサインイン
- Bedrockのモデルを選んで質問を送信
- チャット画面、メッセージ表示、サンプル会話一覧

## アーキテクチャ

1. Reactの画面はAmplify Authenticatorで認証を行います。
2. `src/api/bedrock.ts` がAmplify Dataの `BedrockChat` クエリを呼び出します。
3. AppSyncが認証済みユーザーのリクエストを `bedrockChat` Lambdaへ渡します。
4. LambdaがAWS SDKのBedrock RuntimeクライアントとConverse APIでモデルを呼び出します。

バックエンドの認証、データスキーマ、Lambda、BedrockのIAM権限は `amplify/` 以下で定義しています。

## 現在の実装状況

会話履歴の一覧と表示には `src/sampleData.ts` のサンプルデータを使用しています。画面から作成した会話もブラウザー内のメモリーにだけ保持されるため、ページを再読み込みすると消えます。バックエンドには `Conversation` と `Message` のデータモデルがありますが、現在の画面はそれらを読み書きしていません。

## 技術スタック

- React 19、TypeScript、Vite 7
- AWS Amplify Gen 2（Auth、Data、Functions）
- AWS AppSync、AWS Lambda、Amazon Bedrock Runtime
- Tailwind CSS 4

## ローカル開発

### 必要なもの

- Node.js 22.12以降
- AWSアカウントと、Amplifyバックエンドをデプロイできる認証情報
- 対象リージョンで利用可能なBedrockモデルへのアクセス

### 起動

依存パッケージをインストールします。

```bash
npm ci
```

AWS認証情報を設定した状態で、バックエンド用のサンドボックスを起動します。初回はAWS上に開発用バックエンドが作成され、フロントエンドが参照する `amplify_outputs.json` が生成されます。このファイルはGit管理対象外です。

```bash
npx ampx sandbox
```

サンドボックスの起動後、別のターミナルでフロントエンドを起動します。

```bash
npm run dev
```

AmplifyのサンドボックスはAWS上にリソースを作成します。AWS認証情報の設定方法は[Amplify Gen 2のアカウント設定](https://docs.amplify.aws/react/start/account-setup/)を、サンドボックスの詳細は[クラウドサンドボックスのガイド](https://docs.amplify.aws/react/deploy-and-host/sandbox-environments/setup/)を参照してください。

## 開発コマンド

| コマンド          | 内容                                         |
| ----------------- | -------------------------------------------- |
| `npm run dev`     | Viteの開発サーバーを起動                     |
| `npm run build`   | TypeScriptのビルド確認と本番用ファイルの生成 |
| `npm run preview` | 本番ビルドをローカルでプレビュー             |
| `npm run lint`    | ESLintを実行                                 |

## デプロイ

`amplify.yml` はAWS Amplify Hostingのビルド設定です。`npx ampx pipeline-deploy` で対象ブランチのバックエンドをデプロイし、その後 `npm run build` で生成した `dist/` をフロントエンドとして公開します。Amplify Hostingにリポジトリとブランチを接続し、必要なAWS設定を行ってください。

## ディレクトリ構成

```text
src/
  api/                  Amplify DataクライアントとBedrock呼び出し
  components/           チャット画面のレイアウトとUI
  pages/                新規チャット、会話画面
  sampleData.ts         表示用のサンプル会話
  types/                チャットの型
amplify/
  auth/                 メール認証の設定
  data/                 Conversation、Message、BedrockChatのスキーマ
  function/bedrockChat/ Bedrock Converse APIを呼ぶLambda
  backend.ts            AmplifyバックエンドとIAM権限
amplify.yml             Amplify Hostingのビルド設定
```
