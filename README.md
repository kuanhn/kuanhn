### Hi there, I'm Quan Hoang (Kots) 👋

DevOps Engineer and Senior Manager focused on building reliable platforms, scaling cloud infrastructure, and leading high-performing teams.

### Current Roles

- Cloud Infrastructure Engineer at [DAConsortium][daconsortium]
- Senior Manager at [DAC Data Technology Vietnam JSC][dtvn]

### Core Stack

<img align="left" alt="Terminal" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/terminal/terminal.png" />
<img align="left" alt="Vim" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/vim/vim.png" />
<img align="left" alt="Git" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/git/git.png" />
<img align="left" alt="GitHub" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/github/github.png" />
<img align="left" alt="Terraform" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/terraform/terraform.png" />
<img align="left" alt="Google Cloud Platform" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/google-cloud/google-cloud.png" />
<img align="left" alt="Amazon Web Services" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/aws/aws.png" />
<img align="left" alt="Kubernetes" width="26px" src="https://raw.githubusercontent.com/github/explore/main/topics/kubernetes/kubernetes.png" />

<br />

### Connect with me

[<img align="left" alt="kuanhn | Facebook" width="22px" src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/facebook.svg" />][facebook]
[<img align="left" alt="kuanhn | Twitter" width="22px" src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/twitter.svg" />][twitter]
[<img align="left" alt="kuanhn | LinkedIn" width="22px" src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/linkedin.svg" />][linkedin]

<br />

[facebook]: https://www.facebook.com/iamkots/
[twitter]: https://twitter.com/quannhathoang
[linkedin]: https://www.linkedin.com/in/quanhoangnhat/
[daconsortium]: https://github.com/DAConsortium
[dtvn]: https://github.com/DAC-Data-Technology-Vietnam

---

### SERPツール利用状況ログ収集の推奨構成（社内向けメモ）

ご提示の構成（Pythonアプリ → Cloud Run(Flask API) → Firestore）は、スモールスタートとして妥当です。  
認証なし・IP制限前提の場合は、以下を最低限セットで実施するのが安全です。

- Cloud Run
  - Ingress を `internal and Cloud Load Balancing` に設定
  - 外部公開は HTTPS Load Balancer 経由に限定し、Cloud Armor で送信元IPを許可リスト化
  - サービスアカウントは最小権限（Firestore書き込み権限のみ）
- API（Flask）
  - `/healthz` と `/logs` を分離
  - ログデータは JSON 受信、必須項目（tool_version / host_id / event_type / timestamp）をバリデーション
  - 個人情報・機微情報を送信しないようクライアント側でマスキング
- Firestore
  - `serp_usage_logs` コレクションに日付パーティション相当のキー（`event_date=YYYY-MM-DD`）を保持
  - TTLポリシーで保存期間を自動削除（例: 180日）
  - 将来の BigQuery 連携を見据えて、1イベント1ドキュメントのフラット構造にする

将来的に分析を強化する場合は、次の構成がより運用しやすいです。

- Pythonアプリ → Cloud Run API → Pub/Sub → BigQuery（+ Looker Studio）
  - Firestoreよりも集計系クエリに強く、CSVエクスポートが不要
  - イベント量が増えてもスケールしやすい
  - 監査や再処理（DLQ）を設計しやすい
