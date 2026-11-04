# EDP-Web：SAML 配置与通信字段对照（中日对照）
# EDP-Web：SAML 設定項目と通信項目の対応表（中日対照）

日期／作成日：2026-11-04

## 1. 结论与适用范围／結論と対象範囲

**中文**

配置文件中的 key 与 SAML 报文中的字段名可能不同。框架读取配置后，用于生成报文、决定发送地址或验证认证结果；并非把所有配置 key-value 原样发送给 GSSO。

以下以 **EDP-Web（SP）主动发起登录 → GSSO（IdP）认证 → 认证结果返回 EDP-Web** 的常见 SAML 2.0 浏览器登录流程为例。请求通常使用 HTTP-Redirect，响应使用 HTTP-POST。主要列出正常登录所涉及的内容，不是完整的 SAML XML Schema 字段清单。

**日本語**

設定ファイルのキーと、SAML メッセージ内の項目名は異なる場合があります。フレームワークは設定値を読み取り、メッセージの生成、送信先の決定、認証結果の検証に使用します。すべての設定キーと値を、そのまま GSSO に送信するわけではありません。

以下では、**EDP-Web（SP）からログインを開始 → GSSO（IdP）で認証 → EDP-Web に認証結果を返す**という一般的な SAML 2.0 のブラウザログインを対象とします。通常、リクエストには HTTP-Redirect、レスポンスには HTTP-POST を使用します。正常ログインに関係する主な項目を整理したものであり、SAML XML Schema の全項目一覧ではありません。

### 区分的含义／区分の意味

| 区分 | 中文 | 日本語 |
| --- | --- | --- |
| 必需／必須 | 本文所述流程必须具备的信息或能力；不代表必须在 YAML 中逐项填写，也不代表每条 XML 都包含该字段。 | 本文のフローに必要な情報または機能です。YAML にすべてを個別記載する必要がある、またはすべての XML にその項目が含まれる、という意味ではありません。 |
| 条件必需／条件付き必須 | 在指定场景、GSSO 接入规则或 EDP 业务要求成立时必须具备。 | 対象の利用条件、GSSO の接続要件、または EDP の業務要件に該当する場合に必要です。 |
| 可选／任意 | 通用流程不一定需要。若项目明确要求，则成为本次接入的必需项。 | 一般的なフローでは必須ではありません。プロジェクトで要求された場合は、今回の接続では必須になります。 |

**中文：** 这是一份配置与通信原理的对照说明，不表示 EDP-Web 的实际配置值、证书或 GSSO 个别要求已经确认。  
**日本語：** 本書は設定と通信の対応関係を説明する資料です。EDP-Web の実際の設定値、証明書、GSSO 固有の要件が確定済みであることを示すものではありません。

## 2. 根据配置决定的内容／設定に基づいて決まる内容

**中文**

以下配置 key 以 Spring Boot 的 SAML 自动配置方式举例，省略共同前缀：

**日本語**

以下の設定キーは、Spring Boot の SAML 自動設定を使用する場合の例です。共通のプレフィックスは省略しています。

```text
spring.security.saml2.relyingparty.registration.edp-idp.
```

**中文：** `edp-idp` 是 EDP 内部给该组连接配置取的名称（Registration ID），不是 GSSO 的 Entity ID。不同版本、Java 配置或自定义配置方式，配置 key 可能不同；SAML 标准字段名不会因此改变。  
**日本語：** `edp-idp` は、EDP 内部でこの接続設定に付ける名前（Registration ID）であり、GSSO の Entity ID ではありません。バージョン、Java による設定、独自の設定方式によって設定キーは異なる場合がありますが、それによって SAML の標準項目名が変わるわけではありません。

| 区分 | 项目／項目 | 配置 key／設定キー | SAML 对应位置／SAML の対応箇所 | 中文说明 | 日本語説明 |
| --- | --- | --- | --- | --- | --- |
| 必需／必須 | SP Entity ID | `entity-id` | 请求／リクエスト：`Issuer`；断言／アサーション：`Audience` | 标识 EDP-Web。请求中声明“我是谁”；返回断言中的 Audience 用于表示该结果面向哪个 SP。 | EDP-Web を識別します。リクエストでは送信元を示し、返却されるアサーションの Audience では対象 SP を示します。 |
| 必需／必須 | ACS URL | `acs.location` | `AssertionConsumerServiceURL`、`Recipient`，通常还包括／通常は `Destination` も該当 | EDP 接收认证结果的地址。请求中可以显式携带；返回中的 Recipient 指向此地址。 | EDP が認証結果を受け取る URL です。リクエストに明示する場合があり、返却時の Recipient はこの URL を指します。 |
| 必需／必須 | IdP Entity ID | `assertingparty.entity-id` | 返回报文／断言的 `Issuer`／レスポンス・アサーションの `Issuer` | 标识 GSSO。EDP 用它核对结果是否来自预期的 IdP。 | GSSO を識別します。EDP は認証結果が想定した IdP から発行されたものかを確認します。 |
| 必需／必須 | GSSO 登录接收地址／ログイン要求の送信先 | `assertingparty.singlesignon.url` | 请求的发送地址；通常也用于 `Destination`／送信先 URL。通常は `Destination` にも使用 | 决定认证请求发到哪里。地址必须可用，但不能据此认定所有请求都必须显式包含 Destination 属性。 | 認証リクエストの送信先を決めます。送信先は必要ですが、すべてのリクエストで Destination 属性の明示が必須という意味ではありません。 |
| 可选／按约定；任意／合意内容による | NameID 格式／NameID の形式 | `name-id-format` | `NameIDPolicy@Format` | 请求希望返回哪一种格式的用户标识；不是配置某个用户的实际 ID。 | どの形式のユーザー識別子を返してほしいかを要求します。特定ユーザーの実際の ID を設定する項目ではありません。 |

**中文**

ACS 地址必须确定，但请求中不一定必须显式携带 `AssertionConsumerServiceURL`，也可以由 GSSO 使用事先登记的地址。IdP 的 Entity ID、登录地址、证书等也可能从 Metadata 自动读取，因此不一定要逐项手写配置。

**日本語**

ACS の URL は決める必要がありますが、リクエストで `AssertionConsumerServiceURL` を必ず明示するとは限りません。GSSO に事前登録した URL を使用する場合もあります。また、IdP の Entity ID、ログイン URL、証明書などは Metadata から読み込めるため、すべてを手動で個別設定するとは限りません。

### 同一个配置值对应多个报文字段／一つの設定値が複数の通信項目に対応する例

以下为说明用示例值和 XML 片段，不是可直接部署的完整配置。  
以下は説明用の値と XML の抜粋であり、そのまま適用できる完全な設定ではありません。

```text
EDP 配置／EDP の設定：
entity-id = urn:example:edp-web

EDP 发出的请求／EDP が送信するリクエスト：
<Issuer>urn:example:edp-web</Issuer>

GSSO 返回的断言／GSSO が返すアサーション：
<Audience>urn:example:edp-web</Audience>
```

**中文：** GSSO 读取的是 SAML 的 `Issuer`，不会读取 EDP 配置文件中的 `entity-id` 这个 key。  
**日本語：** GSSO が読み取るのは SAML の `Issuer` であり、EDP の設定ファイルにある `entity-id` というキーではありません。

参考／参照：[S1](#references)、[S5](#references)、[S8](#references)。

## 3. 框架运行时生成、复制或填写的内容／実行時にフレームワークが生成・転記・設定する内容

**中文：** 以下内容通常由 EDP 或 GSSO 的 SAML 实现处理，不需要开发者在配置文件中逐项填写。  
**日本語：** 以下の項目は通常、EDP または GSSO の SAML 実装が処理するため、開発者が設定ファイルに一つずつ記載する必要はありません。

| 区分 | 内容／内容 | SAML 字段／SAML 項目 | 中文说明 | 日本語説明 |
| --- | --- | --- | --- | --- |
| 必需／必須 | 协议版本／プロトコルのバージョン | `Version` | 框架填写固定值 `2.0`。 | フレームワークが固定値 `2.0` を設定します。 |
| 必需／必須 | 唯一编号／一意の ID | `ID` | EDP 生成请求 ID；GSSO 分别生成响应 ID 和断言 ID。 | EDP はリクエスト ID を生成し、GSSO はレスポンス ID とアサーション ID をそれぞれ生成します。 |
| 必需／必須 | 生成时间／発行時刻 | `IssueInstant` | 各方在生成请求、响应或断言时填写。 | 各側がリクエスト、レスポンス、アサーションを発行する際に設定します。 |
| 本文 SP 发起流程必需／本文の SP 起点フローでは必須 | 与原请求的关联／元リクエストとの対応 | `InResponseTo` | GSSO 填入原请求 ID，EDP 用它关联并核对结果。不是重新生成的随机编号。 | GSSO が元のリクエスト ID を設定し、EDP が結果との対応を検証します。新しいランダムな ID ではありません。 |
| 必需／必須 | 处理结果／処理結果 | `Status/StatusCode` | GSSO 根据实际处理结果填写，例如成功或失败。 | GSSO が成功・失敗など、実際の処理結果を設定します。 |
| 本文常见流程必需／本文の一般的なフローでは必須 | 主体确认方式／サブジェクト確認方式 | `SubjectConfirmation@Method` | 使用标准的 `bearer` 方式。 | 標準の `bearer` 方式を使用します。 |
| 本文常见流程必需／本文の一般的なフローでは必須 | 主体确认的有效期限／サブジェクト確認の有効期限 | `SubjectConfirmationData@NotOnOrAfter` | GSSO 根据有效期规则生成；EDP 检查是否已经过期。 | GSSO が有効期間の規則に基づいて生成し、EDP が期限切れでないことを確認します。 |
| 本文常见流程必需／本文の一般的なフローでは必須 | 认证时间及方式／認証時刻と認証方式 | `AuthnStatement@AuthnInstant`、`AuthnContext` | GSSO 根据实际认证情况填写。 | GSSO が実際の認証内容に基づいて設定します。 |
| 认证结果必须可验签／認証結果の署名検証は必須 | 数字签名／電子署名 | `ds:Signature` 等 | GSSO 在运行时计算签名，EDP 验证；签名值不是人工填写的固定值。 | GSSO が実行時に署名を計算し、EDP が検証します。署名値は手動設定する固定値ではありません。 |
| 可选，通常会有／任意、通常は設定される | 断言整体有效时间／アサーション全体の有効期間 | `Conditions@NotBefore`、`Conditions@NotOnOrAfter` | GSSO 按策略生成；与上面的 SubjectConfirmationData 有效期限属于不同位置。 | GSSO がポリシーに基づいて生成します。上記の SubjectConfirmationData の有効期限とは別の項目です。 |
| 可选／任意 | IdP 会话信息／IdP のセッション情報 | `SessionIndex`、`SessionNotOnOrAfter` | GSSO 生成；单点退出等场景可能需要。 | GSSO が生成します。シングルログアウトなどで必要になる場合があります。 |

**中文**

- `InResponseTo` 的要求以上述 SP 发起登录为前提。若采用未经请求的 IdP 发起登录，处理规则不同。
- 成功认证结果需要签名保护；具体签响应、断言还是两者，要按接入要求确认。
- `Conditions@NotBefore` 与 `SubjectConfirmationData` 不可混用。标准 bearer Web SSO 的 `SubjectConfirmationData` 不包含 `NotBefore`。
- 表中 `@` 表示 XML 属性，例如 `NameIDPolicy@Format` 表示 `NameIDPolicy` 元素上的 `Format` 属性。

**日本語**

- `InResponseTo` の扱いは、上記の SP 起点ログインを前提としています。リクエストを伴わない IdP 起点ログインでは、扱いが異なります。
- 正常な認証結果には署名による保護が必要です。レスポンス、アサーション、または両方のどこに署名するかは、接続要件に従って確認します。
- `Conditions@NotBefore` と `SubjectConfirmationData` は混同しないでください。標準の bearer Web SSO では、`SubjectConfirmationData` に `NotBefore` は含めません。
- 表中の `@` は XML 属性を示します。例えば `NameIDPolicy@Format` は、`NameIDPolicy` 要素の `Format` 属性です。

参考／参照：[S1](#references)、[S2](#references)、[S3](#references)、[S5](#references)。

## 4. 随登录用户变化的内容／ログインユーザーによって変わる内容

**中文**

这些内容主要由 GSSO 在认证成功后的断言中返回。通常不需要把每个用户的实际 ID 写进 EDP 的配置文件，再发送给 GSSO。

**日本語**

これらの情報は主に、認証成功後のアサーションに含めて GSSO から返されます。通常、各ユーザーの実際の ID を EDP の設定ファイルに記載して GSSO に送信する必要はありません。

| 区分 | 内容／内容 | SAML 字段／SAML 項目 | 中文说明 | 日本語説明 |
| --- | --- | --- | --- | --- |
| 常见 Spring 默认接入需要／一般的な Spring の標準構成で必要 | 登录用户标识／ログインユーザー識別子 | `Subject/NameID` | Spring 默认使用 NameID 作为认证用户名称；EDP 再映射到本地用户。这里不是说 SAML 的所有实现都必须用 NameID。 | Spring の標準構成では NameID を認証ユーザー名として使用し、EDP 側でアプリ内のユーザーに対応付けます。すべての SAML 実装で NameID が必須という意味ではありません。 |
| 按业务需要，可能必需／業務要件によって必須 | 员工编号、公司代码等／社員番号、会社コードなど | `AttributeStatement/Attribute`、`AttributeValue` | 如果 NameID 不能直接确定 EDP 用户，就可能需要额外的用户识别属性。 | NameID だけで EDP のユーザーを特定できない場合は、追加の識別用属性が必要になることがあります。 |
| 可选／任意 | 姓名、邮箱、部门等／氏名、メールアドレス、部署など | 同上／同上 | 用于画面显示或业务处理。 | 画面表示や業務処理に使用します。 |
| 可选／任意 | 用户组、角色等／グループ、ロールなど | 同上／同上 | 如果由 GSSO 提供，则映射成 EDP 权限；也可以由 EDP 查询自己的权限数据。 | GSSO から受け取る場合は EDP の権限に対応付けます。EDP 自身の権限データを参照する構成も可能です。 |
| 可选／自定义；任意／独自属性 | 额外业务属性／追加の業務属性 | 同上／同上 | 双方约定属性名称、值格式和用途。 | 属性名、値の形式、用途を双方で決めます。 |

### 属性示例／属性の例

```xml
<Attribute Name="employeeId">
    <AttributeValue>123456</AttributeValue>
</Attribute>
```

**中文**

`employeeId` 只是示例名称，不是 Spring 或 SAML 规定所有系统都使用的 key。实际名称可能是另一种字符串，也可能是完整 URI，必须与 GSSO 的设置对应。Spring 可以读取属性，但“该用户对应 EDP 哪个账号、具有什么权限”仍需要 EDP 的业务逻辑处理。

**日本語**

`employeeId` は例であり、Spring や SAML が全システム共通で定めているキーではありません。実際の属性名は別の文字列や完全な URI の場合もあり、GSSO 側の設定と一致させる必要があります。Spring は属性を読み取れますが、「どの EDP アカウントに対応するか」「どの権限を付与するか」は、EDP 側の業務ロジックで処理する必要があります。

参考／参照：[S1](#references)、[S5](#references)、[S7](#references)。

## 5. 本地处理与安全控制配置／ローカル処理とセキュリティ制御の設定

**中文：** 以下配置不是普通的业务传输字段。文件路径通常只在本地使用，安全开关用于控制框架行为。  
**日本語：** 以下は通常の業務データとして送信する項目ではありません。ファイルパスは基本的にローカルで使用し、セキュリティ設定はフレームワークの動作を制御します。

配置 key 仍省略第 2 节中的共同前缀。  
設定キーは、引き続き第 2 節の共通プレフィックスを省略しています。

| 区分 | 内容／内容 | 配置 key／設定キー | 中文说明 | 日本語説明 |
| --- | --- | --- | --- | --- |
| 可信验签信息必需／信頼できる署名検証情報は必須 | GSSO 签名公钥证书／GSSO の署名検証用証明書 | `assertingparty.verification.credentials[0].certificate-location`，或从 Metadata 获取／または Metadata から取得 | EDP 用于验证 GSSO 签名。证书文件的本地路径不会发送给 GSSO。 | EDP が GSSO の署名を検証するために使用します。証明書のローカルファイルパスは GSSO に送信しません。 |
| 可选的配置获取方式／任意の設定取得方法 | GSSO Metadata | `assertingparty.metadata-uri` | 自动读取 IdP ID、登录地址、证书等。该配置 key 不进入每次登录报文。 | IdP の ID、ログイン URL、証明書などを読み込みます。この設定キーが毎回のログインメッセージに入るわけではありません。 |
| 条件必需／条件付き必須 | 登录请求签名／ログインリクエストへの署名 | `assertingparty.singlesignon.sign-request` | GSSO 要求请求签名时必须启用。它控制是否签名，不是实际签名值。 | GSSO がリクエスト署名を要求する場合は有効にします。署名するかどうかを制御する設定であり、署名値そのものではありません。 |
| 请求签名时必需／リクエスト署名時に必須 | SP 签名私钥及证书／SP の署名用秘密鍵と証明書 | `signing.credentials[0].private-key-location`；`signing.credentials[0].certificate-location` | 私钥用于生成签名；公钥证书通过登记或 Metadata 提供给 GSSO 用于验证。 | 秘密鍵で署名を生成し、公開鍵証明書は登録や Metadata を通じて GSSO に提供して検証に使用します。 |
| 接收加密内容时必需／暗号化された内容を受信する場合に必須 | SP 解密私钥及证书／SP の復号用秘密鍵と証明書 | `decryption.credentials[0].private-key-location`；`decryption.credentials[0].certificate-location` | GSSO 使用 SP 公钥加密，EDP 使用私钥解密；加密断言可能表现为 EncryptedAssertion。 | GSSO が SP の公開鍵で暗号化し、EDP が秘密鍵で復号します。暗号化されたアサーションは EncryptedAssertion として返される場合があります。 |
| 可选／按要求；任意／要件による | 强制重新认证、禁止交互、指定认证强度／再認証の要求、対話の禁止、認証コンテキストの指定 | 通常通过请求定制代码处理／通常はリクエストのカスタマイズで設定 | 对应 ForceAuthn、IsPassive、RequestedAuthnContext 等标准字段。不能假定每项都有同名 YAML key。 | ForceAuthn、IsPassive、RequestedAuthnContext などの標準項目に対応します。それぞれに同名の YAML キーがあるとは限りません。 |

**中文**

- 私钥不会发送给 GSSO。
- 公钥证书可以通过申请登记或 Metadata 交换，但本地文件路径不会成为 SAML 参数。
- 仅在 EDP 配置解密证书，不会自动让 GSSO 开启加密；GSSO 还需要取得 SP 公钥证书并配置相应策略。
- 请求签名与响应验签是不同方向的处理，不能因为“请求签名可选”就理解为“响应可以不验签”。

**日本語**

- 秘密鍵を GSSO に送信することはありません。
- 公開鍵証明書は申請時の登録や Metadata で交換できますが、ローカルファイルパスが SAML パラメーターになるわけではありません。
- EDP に復号用証明書を設定するだけで、GSSO 側の暗号化が自動的に有効になるわけではありません。GSSO 側でも SP の公開鍵証明書の取得とポリシー設定が必要です。
- リクエストへの署名と、レスポンスの署名検証は方向が異なる処理です。「リクエスト署名が任意」であることは、「レスポンスの署名検証が不要」という意味ではありません。

参考／参照：[S5](#references)、[S6](#references)、[S7](#references)、[S8](#references)。

## 6. 最外层 HTTP 传输参数／HTTP の外側の送信パラメーター

**中文：** 浏览器里看到的 HTTP 参数，与参数内部的 SAML XML 字段属于不同层。  
**日本語：** ブラウザで確認できる HTTP パラメーターと、その中に含まれる SAML XML の項目は、異なる階層の情報です。

| 区分 | HTTP 参数／HTTP パラメーター | 中文说明 | 日本語説明 |
| --- | --- | --- | --- |
| 本文请求传输需要／本文のリクエスト送信で必要 | `SAMLRequest` | 编码后的整个认证请求 XML；HTTP-Redirect 时还涉及压缩和 URL 编码。 | エンコードされた認証リクエスト XML 全体です。HTTP-Redirect では圧縮と URL エンコードも行います。 |
| 本文响应传输需要／本文のレスポンス送信で必要 | `SAMLResponse` | HTTP-POST 表单中，Base64 编码后的整个认证响应 XML。 | HTTP-POST のフォームに格納される、Base64 エンコード済みの認証レスポンス XML 全体です。 |
| 可选，框架常用／任意、フレームワークでよく使用 | `RelayState` | 携带或关联登录前后状态；IdP 按绑定规则原样返回。它不是 SAML XML 内部字段。 | ログイン前後の状態を引き継いだり対応付けたりします。IdP はバインディングの規則に従って同じ値を返します。SAML XML 内部の項目ではありません。 |
| Redirect 请求签名时需要／Redirect リクエストに署名する場合に必要 | `SigAlg`、`Signature` | 分别表示签名算法和签名值。与响应 XML 内部的 ds:Signature 所在位置不同。 | 署名アルゴリズムと署名値を示します。レスポンス XML 内部の ds:Signature とは格納場所が異なります。 |

**中文：** 浏览器中可能只看到一个 `SAMLResponse` 参数，但解码后，里面才有 `Issuer`、`NameID`、`Audience`、时间、签名及用户属性等内容。  
**日本語：** ブラウザ上では `SAMLResponse` という一つのパラメーターに見えても、デコードすると、その内部に `Issuer`、`NameID`、`Audience`、時刻、署名、ユーザー属性などが含まれています。

参考／参照：[S1](#references)、[S4](#references)。

## 7. 本次调查重点／今回の調査で重点的に確認する内容

| 中文 | 日本語 |
| --- | --- |
| 请求编号、时间、XML 组装、签名计算等运行时处理，主要交给框架；调查重点是其配置是否满足接入要求。 | リクエスト ID、時刻、XML の組み立て、署名計算などの実行時処理は主にフレームワークが担当します。調査では、その設定が接続要件を満たすかを確認します。 |
| 确定双方 Entity ID、GSSO SSO URL 和 EDP ACS URL。 | 双方の Entity ID、GSSO の SSO URL、EDP の ACS URL を確認します。 |
| 确定 GSSO Metadata／可信签名证书的获取方式及更新方式。 | GSSO の Metadata／信頼する署名検証用証明書の取得方法と更新方法を確認します。 |
| 确定请求是否签名、响应／断言如何签名、是否加密以及相关证书要求。 | リクエスト署名の要否、レスポンス／アサーションの署名方式、暗号化の要否、必要な証明書を確認します。 |
| 确定 NameID 的来源、格式及实际含义；如需额外属性，明确属性名称和值格式。 | NameID の取得元、形式、値の意味を確認します。追加属性が必要な場合は、属性名と値の形式を明確にします。 |
| 确定认证用户如何映射到 EDP 本地账号，以及如何读取或赋予 EDP 权限。 | 認証ユーザーを EDP 内のアカウントにどう対応付け、EDP の権限をどう取得・付与するかを確認します。 |
| 表中“可选”项目，如果 GSSO 或 EDP 业务明确要求，就成为本次接入的必需项。 | 表で「任意」とした項目でも、GSSO または EDP の業務要件で要求される場合は、今回の接続では必須になります。 |

<a id="references"></a>

## 8. 官方参考资料／公式参考資料

以下链接对应上述说明的协议与框架依据；不替代 GSSO 内部接入手册及 EDP-Web 的实际设计。  
以下は上記の説明に関係するプロトコルとフレームワークの公式資料です。GSSO の内部接続手順書や EDP-Web の実際の設計に代わるものではありません。

- **S1 — Microsoft Entra：SAML 单点登录协议／SAML シングルサインオンプロトコル**  
  https://learn.microsoft.com/zh-cn/entra/identity-platform/single-sign-on-saml-protocol
- **S2 — OASIS：SAML 2.0 Core**  
  https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf
- **S3 — OASIS：SAML 2.0 Profiles**  
  https://docs.oasis-open.org/security/saml/v2.0/saml-profiles-2.0-os.pdf
- **S4 — OASIS：SAML 2.0 Bindings**  
  https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf
- **S5 — Spring Security：SAML 2.0 Login Overview**  
  https://docs.spring.io/spring-security/reference/servlet/saml2/login/overview.html
- **S6 — Spring Security：SAML 2.0 Authentication Requests**  
  https://docs.spring.io/spring-security/reference/servlet/saml2/login/authentication-requests.html
- **S7 — Spring Security：SAML 2.0 Response Authentication**  
  https://docs.spring.io/spring-security/reference/servlet/saml2/login/authentication.html
- **S8 — Spring Boot：SAML 配置属性 API／SAML 設定プロパティ API**  
  [Registration](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/security/saml2/autoconfigure/Saml2RelyingPartyProperties.Registration.html)  
  [Acs](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/security/saml2/autoconfigure/Saml2RelyingPartyProperties.Registration.Acs.html)  
  [AssertingParty](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/security/saml2/autoconfigure/Saml2RelyingPartyProperties.AssertingParty.html)  
  [Singlesignon](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/security/saml2/autoconfigure/Saml2RelyingPartyProperties.AssertingParty.Singlesignon.html)













# **有一点重心偏移：之前更多是在整理“使用 GSSO 的申请条件和通用流程”；而这份针对 EDP-Web 的调查，已经进入“本系统具体要改哪里、怎样接入”的核心部分。接下来应该把重点转向完善这部分。**

之前的工作仍然有用，但仅靠申请流程、参数名称和 SAML 原理，还不能完整回答最初的任务。

**1．这份系统调查，确实属于你应该得出的核心结论**

最初对方要求的是：

> 実装する際に、必要な手順などを調査してまとめていただきたい
> 调查并整理实际实施时需要的步骤。

因此，最终成果需要能够回答：

- EDP-Web 使用什么组件实现 SAML？
- 现有程序已经有什么，还缺什么？
- 哪些只需要修改配置，哪些需要修改代码？
- 需要从 GSSO 或其他担当那里取得什么？
- 按什么顺序实施，怎样确认成功？

**那份系统调查已经回答了其中相当一部分。** 例如，它指出已有 Spring Security SAML 依赖，但还存在认证放行、用户信息依赖 Cookie、认证成功处理尚未衔接等问题。这些直接决定了 EDP-Web 的实施步骤。

**2．它具体补齐了原来哪部分？**

对照[之前的调查表](https://chatgpt.com/library?file_id=libfile_173d25ceef748191b5f52203d6810f61&account_id=personal)，对应关系如下：

| 原来的调查项                        | 这份系统调查补充了什么                                       | 目前仍缺什么                                   |
| ----------------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| **03／＋02-1：SP 的实际组件**       | 次期版程序已有 Spring Security SAML 库，提出由应用内框架承担 SP 功能的方案 | 确认团队正式采用该方案，以及调查对应的源码版本 |
| **＋02-2：ACS、响应验证、身份交付** | 指出了 Spring Security 过滤器接收响应的方式、ACS 路径，以及通过服务端认证信息向业务处理提供用户身份的方案 | 实际配置、处理衔接及运行验证                   |
| **＋02-3：用户与权限处理**          | 指出了 GSSO 用户与 EDP 的“公司代码＋用户 ID”需要对应，以及现有 Cookie 依赖需要调整 | 正式映射规则、属性名称、未注册用户等处理规则   |
| **＋04：系统影响与改修点**          | **补充最多的部分。** 已定位到 `SecurityConfig`、`LoginUserResolver`、直接读取 Cookie 的 Controller、权限处理等 | 核对源码依据、确认修改范围和具体方案           |
| **04／＋03：双方配置和环境参数**    | 指出了配置位置、占位值及部分标准路径                         | QA／本番实际 URL、Entity ID、证书、用户属性等  |
| **05：到实装的手顺**                | 给出了 EDP 侧实施顺序、测试与迁移关注点                      | 每一步的输入、具体操作、完成条件和担当         |
| **02／＋01：申请与手续**            | 基本没有补充本系统的申请事实                                 | 申请记录、提交／受理／批准状态仍需另外确认     |

所以，它主要补上了之前最薄弱的 **“EDP-Web 专属的实现方式和改修影响”**。

**3．之前的“60 分／90 分”划分需要修正一点**

这两个分数是我们为整理成果设置的层次，**不是对方正式给出的验收标准**。

我之前把“现行系统影响”整体放在 90 分的补充项里，容易让你理解成：申请和通用流程整理好，核心任务就基本完成了。**对于这次“调查到实施为止的步骤”的要求，这个划分不够准确。**

更合适的区别是：

| 最低应交付的内容                                             | 更完善的内容                                               |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| 能说清 EDP 用什么接入、主要改哪里、缺什么信息、实施顺序是什么 | 能定位到具体文件／处理、配置项、前置条件、负责人和验证方法 |
| 尚未确定的地方，有明确的问题和确认对象                       | 接手人可以据此继续设计或实施，少做重复调查                 |

**本系统“主要改哪里”应当进入最低交付；详细到每个操作和验证步骤，才属于进一步完善。** 这也不意味着你必须亲自把代码改完。

**4．但那份系统调查还不能直接作为全部“已确认结论”**

照片里明确说明：它依据的是**提供的次期版源码的静态检查**，尚未启动应用、连接实际 IdP 或实施运行测试。

其中应区分三类：

- **资料报告的现状**：已有依赖、配置为占位值、某些处理读取 Cookie。
- **提出的实施建议**：使用服务端会话保存用户信息、调整权限处理、采用某种身份映射方式。
- **仍需决定的事项**：具体用户属性、MACCS 是否纳入迁移、退出范围、会话期限等。

因此，“资料已经给出建议”可以更新为进展；**不能直接更新成“正式设计已确定”或“已经可以接入”。** 也不要把次期版源码的状态写成现行生产系统的实际状态。

**5．接下来确实应该沿着这份结果完善，重点做这三件事**

1. **核实它的依据。**
   确认对应哪一版 EDP 源码，核对关键文件和处理。已有定位可以直接使用，不必从头重新调查。
2. **把发现转成实施事项。**
   每项整理成：
   **当前状态 → 需要达到的状态 → 修改配置还是代码 → 缺少的信息 → 验证方法。**
   例如，“已有 SAML 依赖”还不够，要继续明确“如何启用、如何要求业务请求必须登录、登录成功后如何接上 EDP 用户和权限”。
3. **集中确认会影响方案的未决事项。**
   优先确认应用内 Spring Security 的实现方案、用户标识映射、旧登录及 MACCS 的迁移范围，以及环境参数。申请状态和证书资料可以并行追踪。

你现在更准确的进度是：**申请与通用流程已经整理；本系统的实现方式和主要改修点已有调查材料；接下来要核实这些材料，并把它们整理成 EDP-Web 可执行的实施步骤。**
