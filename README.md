# Kubernetes-Security 3章

- Why did you started to English teacher?
- What is you hobby?
  - My hobby is play PC game. For example I like CoD. It is shotting game.
  - And I love cook! My best of dish is NIKUJAGA.
  - Inside are potatoes and meat and soy sauce.
  - If you get a chance, you should try it. You can eat it in Restaurant.
- What kind of drama or movie do you like?
  - I like suits that american drama.
    - It is lawyer story. Already done up to 8 seasons.



- I have none yet
- I dont have any questions



## Threat Modeling

この章では、

脅威モデルの簡単な紹介から始まり、Kubernetesエコシステム内でのコンポーネントの相互作用について説明します。さらにデフォルトのKubernetsの設定で脅威を見ていきます。その後、Kubernetes エコシステムでアプリケーションを脅威モデル化する際に、追加の脅威アクターと攻撃面をどのように導入するかについてお話します。

この章のゴールは、デフォルトのKubernetesの設定では、十分にデプロイされたアプリケーションを攻撃者から保護できないことがわかるようになります。

Kubernetesは日々進化しています。そのためこの章では、環境によって脅威の深刻度が異なるため、緩和策を設けていません。

----

この章の強調箇所として、Kubernetes エコシステムとは、Kubernetes クラスタ内の Kubernetes コンポーネントとワークロードを含んでいます。そのため、開発者とDevOpsエンジニアはデプロイするリスクを考えなくてはなりません。さらに既知の脅威に対するリスク軽減計画を策定していなければなりません。

この章では、次のトピックをお話しします。

- 脅威モデルの紹介
- コンポーネントの相互作用
- Kubernetes環境における脅威アクター
- Kubernetesのコンポーネント/オブジェクト 脅威モデル
- Kubernetesでの脅威モデリングアプリケーション

----

## 脅威モデルの紹介

脅威のモデル化とは、ソフトウェア開発ライフサイクル（SDLC）の設計段階でシステム全体を分析し、システムに対するリスクを特定するプロセスです。

脅威モデル化は、開発サイクルの早い段階でセキュリティ要件を考え、最初からリスクの深刻度を下げるために使用されます。

脅威のモデル化には、脅威を特定し、それぞれの脅威の影響を理解し、最終的にすべての脅威の緩和戦略を策定することが含まれます。脅威のモデル化は、エコシステム内のリスクの可能性と影響、および存在する場合にはそれに対応する軽減戦略を単純なマトリックスとして表現することを目的としています。

脅威モデル化について理解したのちに次のことについて定義することができます。

1. アセット：守るべきエコシステムの資産
2. セキュリティコントロール：識別されたリスクからシステムのアセットを保護することを目的としています。これらは、アセットへのリスクを回避するためのセーフガードまたは対策です。
3. 脅威アクター：脅威アクターとは、リスクを悪用するスクリプト・キディ、国家のハッカー、ハクティビストを含む実体や組織のことです。（**ハクティビスト**とは、サイバー犯罪に関する用語で、社会的・政治的な主張を目的としたハッキング活動（ハクティビズム）を行う者のことである。）
4. 攻撃面：脅威アクターが相互作用しているシステムの部分。それは、システム内の脅威アクターのエントリーポイントが含まれます。
5. 脅威：アセットへのリスク
6. 緩和：緩和は、アセットに対する脅威の可能性と影響を減らす方法を定義します。

業界では通常、脅威のモデル化について以下のいずれかのアプローチを採用しています。

- STRIDE Microsoftにて１９９９年に公開されました。これは、Spoofing(なりすまし),Tampering(改ざん),　Repudiation(否認), Information Disclosure(情報漏洩), Denial of Service(サービスの拒否), Escalation of Privileges(権限昇格)の頭文字をとってます。STRIDEは、システムの脅威モデルの答えとして、”システムに対して何が問題を起こすのか？”について重点を置いているモデルである。
- PASTA 攻撃シミュレーションのプロセスと脅威分析は、脅威モデル化のリスクを中心としたアプローチです。PASTAは攻撃者中心のアプローチを採用しており、ビジネスチームや技術チームがアセット中心での緩和戦略として利用されています。
- VAST Visual(視覚的に), Agile(柔軟に),Simple(簡潔に)  脅威モデル化は、アプリケーションおよびインフラストラクチャ開発全体の脅威モデリングをSDLCおよびアジャイルソフトウェア開発と統合することを目的としています。
  ITは、開発者、アーキテクト、セキュリティ研究者、ビジネスエグゼクティブなど、すべてのステークホルダーに実用的なアウトプットを提供する可視化スキームを提供します。

脅威のモデル化には他にもアプローチがありますが、業界内で最も利用されているのは前者の3つです。

脅威モデルのスコープが十分に定義されていない場合、脅威モデル化は無限に長い作業になる可能性があります。
エコシステム内の脅威の特定を始める前に、各コンポーネントのアーキテクチャと動作、およびコンポーネント間の相互作用を明確に理解しておくことが重要です。

まず、Kubernetesのエコシステム内の脅威を調査する前に、Kubernetes内の異なるコンポーネント間の相互作用を見ていきます。

## コンポーネントの相互作用

Kubernetsのコンポーネントは、クラスタ内で動作するマイクロサービスが期待通りに機能するように協調して動作します。もし、デーモンセットとしてマイクロサービスをデプロイしたとすると、Kubernetes コンポーネントは、すべてのノードでマイクロサービスを実行しているポッドが 1 つあることを確認します。では、その裏側はどうなっているのでしょうか？　下記の図をご覧ください。

これらのコンポーネントが何をするのかを簡単にまとめてみました。

- Kube-apiserver: Kubernetes API サーバはコントロールプレーンコンポーネントでオブジェクトのデータの検証と設定を行います。
- Etched: etcdは、設定、状態、メタデータなどのデータを保存するために使用される高い可用性のあるキーバリューストアです。
- Kube-scheduler: kube-schedulerはKubernetesのデフォルトのスケジューラです。新しく作成されたポッドを監視し、ポッドをノードに割り当てます。
- Kuber-controller-manager: Kuber-controller-managerの状態の更新を監視し、それに応じてクラスタに変更を加えるコアコントローラの組み合わせ。
- Cloud-controller-manager: Cloud-controller-managerは、基盤となるクラウドプロバイダーと連携するためのコントローラを実行します。
- Kubelet: kubeletはノードをAPIサーバに登録し、Podspecsを使って作成されたポッドを監視して、ポッドとコンテナが健全であることを確認します。

kube-apiserver のみが etcd と通信していることに注目してください。Kube-scheduler, Kuber-controller-managerやCloud-controller-managerなどの他のコンポーネントは、それぞれの責任を果たすためにマスターノードで実行されているkube-apiserverと相互作用します。

----

これらのコンポーネントがどのように相互に連携するかを試すために、DaemonSetの作成を例としてみてみましょう。

デーモンセットを作成するには、以下の手順で行います。

1. ユーザはkube-apiserverにリクエストを送信し、HTTPS経由でDaemonSetワークロードを作成します。
2. 認証、認可、オブジェクト検証の後、kube-apiserverは、etcdデータベースにDaemonSetのワークロードオブジェクト情報を作成します。etcdでは、送信中のデータも静止中のデータもデフォルトでは暗号化されていません。
3. DaemonSetコントローラは、新しいDaemonSetオブジェクトが作成されるのを監視します。そしてその後、kube-apiserverにPod作成リクエストを送ります。DaemonSetは基本的にマイクロサービスがすべてのノードのポッドの中で実行されることを意味していることに注意してください。
4. Kube-apiserver は２のアクションを繰り返し、etcdデータベースにDaemonSetのワークロードオブジェクト情報を作成します。
5. Kube-scheduler は新しいPodが作成されるのを監視します。その後ノード選択基準に基づいて、どのノードでポッドを実行するかを決定します。その後、kube-schedulerは、どのnodeのpodを動かすのかをkube-apiserverにリクエストを送信します。
6. kube-apiserverは、kube-schedulerからリスエストを受け取ってからetcdのpodのノードの割り当て情報を更新します。
7. ワーカーノード上で実行されているkubeletは、このノードに割り当てられた新しいポッドを監視します。その後、Container Runtime Interface(CRI)コンポーネントにリクエストを送信し,コンテナを実行します。またその後、kubeletは、kube-apiserverにpodのステータスを送信します。
8. Kube-apiserverはkubeletのターゲットノードからpodのステータス情報を受け取ります。その後、etcdのpodのステータス情報を更新します。
9. 一度DaemonSetから作成されたpodは、他のKubernetesコンポーネントと通信することができ、また、マイクロサービスは、立ち上がって実行します。

全ての通信がコンポーネント間でデフォルでセキュアなわけではない。それは、それらのコンポーネントの設定に依存します。詳細は第６章でやります。

-----

## Kubernetes環境における脅威アクター

脅威アクターとは、アセットを保護すべきシステム内で実行されているエンティティまたはコードのことです。守る観点から、まず初めに理解する必要があるのは、あなたの潜在的な敵は誰なのか、でないとあなたの防衛戦略はあまりにも曖昧になります。Kubernetes環境における脅威アクターは、大きく分けて3つに分類されます。

1. エンドユーザ：エンティティはアプリケーションに接続できる。アクターのエントリーポイントは、通常、ロードバランサーまたはイングレスです。時にpod,コンテナ、またはNodePortsは直接インターネットにされされているかもしれない、エンドユーザーのためのエントリーポイントを増や必要があるかもしれない
2. 内部犯：エンティティは、Kubernetesクラスター内部へのアクセスを制限されている。クラスター内の悪意のあるコンテナまたは、生成されたPodは、内部攻撃者の例です。
3. 特権攻撃者:エンティティはKubernetesクラスター内部に管理者権限でのアクセスができます。インフラストラクチャー管理者、危殆化した kube インスタンス、悪意のあるノードなどが特権攻撃者の例です。

リスクを悪用するスクリプト・キディ、国家のハッカー、ハクティビストなどのアクターは上記の３つに分類されます。

次のダイアグラムは、Kubernetesエコシステム内の別のアクターについてです。



エンドユーザーは一般的に、イングレスコントローラ、ロードバランサ、ポッドによって公開されたHTTP/HTTPSルートと通信します。エンドユーザーは最も権限の低い人です。他の手として内部犯は、クラスター内のリソースにアクセスするのに制限がされています。特権攻撃者は、特権を持っており、クラスタを変更する権限を持っています。これらの三つの攻撃者のカテゴリーは、脅威の厳しさを決定するのに役に立ちます。エンドユーザが関与する脅威は、特権攻撃者が関与する脅威よりも深刻度が高くなります。
けれども、それらは、ダイアグラムでは独立したロールのように見えるのは、攻撃者は特権昇格攻撃者を利用して、エンドユーザから内部攻撃者に変更することができるためです。

----

## Kubernetesクラスター脅威

Kube のコンポーネントと脅威となるアクターについての新しい理解により、私たちはKube クラスターの脅威モデル化の旅へと進みます。次の記載のテーブルは、メジャーなkubeコンポーネント、ノードそしてpodについてカバーしてます。これらのコンポーネントはすべてアセットであり、脅威から保護されるべきであることに注意してください。
コンポーネントのいずれかが危険にさらされると、特権のエスカレーションのような攻撃の次のステップにつながる可能性があります。また、kube-apiserverとetcdはKubernetesクラスタの頭脳であり心臓部であることにも注意してください。もしどちらかが危険にさらされたら、ゲームオーバーになってしまいます。

下記の表はKubeのデフォルトの設定に含まれる脅威についてまとめました。また、それらの脅威からアセットを管理者や開発者がどのように守ることができるのかについて記載しています。

| Asset          | Threat                                                       | Security control                                 | Mitigation strategy                                          |
| :------------- | :----------------------------------------------------------- | :----------------------------------------------- | ------------------------------------------------------------ |
| kube-apiserver | デフォルトのAuditポリシはありません。<br />これにより、攻撃後フォレンジック分析ができません。 | audit ポリシ                                     | auditポリシを有効にする。<br />メタデータレベルはエコシステム全体で推奨されています。 |
|                | --- anonymous---<br />auth はデフォルトで true に設定されており、基本的にはどのような接続でも kube-apiserver に接続できるようになっています。 | 認証認可                                         | --anonymous-auth が false に設定されていないことを確認してください。 |
|                | 自己署名証明書を使用しているため、認証が弱い。               | クライアントCAを有効<br />--client-ca-file       | kube-apiserver への接続を監視して異常をチェックします。      |
| etcd           | デフォルトではデータは暗号化されていません。                 | データを暗号化<br />--encryption-provider-config | デフォルトでは、設定パラメータ --encryption-provider-config を kube-apiserver に渡します。 |
|                | 認証はデフォルトでは有効になっていません。                   | 認証認可                                         | 認証が有効になっていることを確認し、TLS を使用してクラスタ内の悪意のあるコンポーネントやオブジェクトからのアクセスを回避します。 |
|                | Kubernetesエコシステム内の複数のコンポーネントからアクセス可能。 | mTLS                                             | mTLS は kube-apiserver 以外の etcd への接続をすべて拒否します。また、対応するサーバとクライアントの証明書と鍵のペアを生成します。<br />これらの証明書は、起動時にAPIサーバなどで使用されます。 |
| kube-scheduler | Kubernetesエコシステム内の任意のコンポーネントからアクセス可能。Kubernetes クラスタを自由に使用できます。 | N/A                                              | kube-apiserver 以外の kube-scheduler への接続を悪意のあるものとしてフラグを立てます。 |



| Asset              | Threat                                                       | Security control                                             | Mitigation strategy                                          |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| Controller manager | コンポーネントの分離が正しく設定されていないと、<br />特権のエスカレーション攻撃につながる可能性があります。 | N/A                                                          | N/A                                                          |
|                    | コントローラマネージャは環境変数、コマンドライン引数、Kubernetesのシークレットなどの秘密情報を扱いますが、<br />各コンポーネントはそれらのシークレットに対する保護を最小限に抑えています。 | Kubernetesのシークレットのローテーション                     | Kubernetesのシークレットは、シークレットを扱うための標準的な方法を提供します。<br />シークレットは起動時に暗号化されるように設定する必要があります。HashiCorpのようなシークレット保管庫をシークレットの代わりに使うこともできます。 |
| kubelet            | kubeletは、暗号化されていないディスクにブートストラップ証明書を書き込みます。 | N/A                                                          | 証明書はクラスタが稼働した後に削除してください。また、ディスク暗号化も有効にしてください。 |
|                    | kubelet エンドポイントは、ノードやコンテナを侵害するために利用できます。これらのエンドポイントは認証されていません。 | 認証認可                                                     | 認証を確実にするためにkubeletでCAバンドルを提供します。      |
| Container runtime  | コンテナランタイムのためのフィルタがないと、エコシステム内で悪意のあるイメージを取得する原因になります。 | ImagePolicyWebhookアドミッションコントローラとClairなどのイメージスキャンツール | ImagePolicyWebhookアドミッションコントローラを使用して、承認されたソースからのイメージがエコシステム内で実行されるようにします。<br />Clairなどのツールでイメージをスキャンするのも有効です。 |
| Nodes              | 危殆化したノードはカーネルモジュールをポッドに注入して危殆化させることができます。 | /etc/modprobe.d/<br />kubernetes-blacklist.conf              | コンテナの危殆化を防ぐために、カーネルモジュールのブラックリストを提供することができます。 |
|                    | ノード上で脆弱性のあるバイナリやサービスは、ポッドやクラスターの侵害につながる可能性があります。 | 最小化されたOS                                               | Alpineのように、コンテナランタイムをサポートするためのバイナリやライブラリを最小限に抑えたOSを使用してください。 |
|                    | kubeconfigや秘密鍵を含むKubernetesデータへのアクセスできる。 | N/A                                                          | ホスト侵入検知システム(HIDS)とファイル整合性監視(FIM)は、kubeconfigや秘密鍵がアクセスされた場合に通知するのに役立ちます。 |



| Asset | Threat                                                       | Security control           | Mitigation strategy                                          |
| :---- | :----------------------------------------------------------- | :------------------------- | ------------------------------------------------------------ |
| Pods  | ポッドはルートユーザとして実行し、<br />ホストを危険に晒される可能性があります。 | ポッドセキュリティポリシー | 通常のワークフローでは、まれにポッドにルート権限が必要になることがあります。<br />ポッドのセキュリティポリシーは、ポッドが特権ユーザとして実行されないようにすることができます。 |
|       | ポッド内のネットワーク分離は、ネットワークポリシーを使用して実装することができます。<br />ネットワークポリシーは、エラー通知なしで失敗します。 | N/A                        | ネットワークトラフィックに対して、ポリシーが正しく適用されているかどうか検証することをお勧めします。 |
|       | デフォルトでは、すべてのポッドはデフォルトのサービスアカウントに関連付けられています。<br/>RBAC が有効になっていない場合、危殆化したポッドはクラスタ内の API 呼び出しを行うことができます。 | N/A                        | kubectl パッチ serviceaccount default -p "automountServiceAccountToken:false" は、<br />デフォルトのサービスアカウントトークンの自動マウントを無効にします。 |



脅威は他にもあり、それは後の章で説明します。前の表が、何を保護する必要があるのか、また、Kubernetes クラスタをどのように保護するのかについて、声を大にして考えるきっかけになることを期待しています。

## Kubernetesでの脅威モデリングアプリケーション

では、Kubernetes上にデプロイされたアプリケーションで脅威のモデル化がどのように異なるのかを説明しましょう。Kubernetesでのデプロイは、脅威モデルにさらなる複雑さが増します。Kubernetesは、デプロイされたアプリケーションへの脅威を調査する前に加えて考慮事項、アセット、脅威のアクター、そして新しいセキュリティコントロールについて考える必要がある。

例として、下図が三構成のWebアプリケーションなる。





同じアプリケーションでも、Kubernetes環境では少し違うように見えます。









