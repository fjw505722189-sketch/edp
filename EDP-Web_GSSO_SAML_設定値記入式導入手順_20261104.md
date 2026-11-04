# EDP-Web × GSSO／Microsoft Entra ID：SAML連携 設定値記入式導入手順

> 2026-11-04作成。目的：プロジェクトでSPの実装コンポーネント、QA／本番の設定値、担当者を決定した後、申請、双方の設定、連携検証、受入確認を順に進めるための手順。本書には他システムの記入例の値を使用していない。また、EDP-Webの申請提出や実装完了を示すものではない。

## 最初に確認：この手順でそのまま進められる範囲

Microsoftの三つの参考ページで分かるのは、SAMLのAuthnRequest／Response、Entra管理センターでのエンタープライズアプリケーションの検索方法、IdPとSPがメタデータやエンドポイント情報を交換する原則である。**EDP-Web固有のコンポーネント、コード、ユーザーテーブル、GSSO内部の申請記録は記載されていない。**したがって、どの構成でも「値だけ差し替えれば実装できる」とは言えない。

本書は次の二段構成とする。

1. **共通の作業手順**：プロジェクト固有の設定値と担当者が確定していれば、GSSO／Entraの設定、情報の受け渡し、QA検証、本番切替を項目ごとに進められる。
2. **条件付きのSP設定例**：「EDP-WebがSpring Boot 3.5＋Spring Security 6.5のServletアプリケーションで、アプリケーション本体がSPを担う」場合に限り、第4章の設定例を参考にできる。SPをリバースプロキシや共通PFが担う場合、このプログラム用の設定例は使用せず、該当製品の手順に従って同じ設定値を登録する。

【要記入】は実作業前に確定するプロジェクト固有の値、【作業停止】は設計上の決定や権限が不足しており、推測で先へ進めない箇所を示す。

## 0. 設定値一覧を作成する

以下の表をコピーし、QA用と本番用の値をそれぞれ記入する。他システムのURL、テナントID、申請番号は流用しない。証明書欄には**公開証明書の承認された受け渡し場所**のみ記載し、秘密鍵を貼り付けない。

| 項目 | QAの値 | 本番の値 | 決定／提供する担当 |
| --- | --- | --- | --- |
| SPを実装するコンポーネントとバージョン | 【要記入】 | 【要記入】 | EDP-Web基盤／アプリ担当 |
| 外部公開URL（HTTPS） | 【要記入】 | 【要記入】 | EDP-Web基盤担当 |
| SP Entity ID（識別子） | 【要記入】 | 【要記入】 | SP担当が定義し、GSSO側に登録 |
| ACS URL（応答URL） | 【要記入】 | 【要記入】 | SP担当が提示し、GSSO側に登録 |
| SP起点のログインURL（該当する場合） | 【要記入】 | 【要記入】 | SP担当 |
| SP metadata XML／URL（存在する場合） | 【要記入／なし】 | 【要記入／なし】 | SP担当 |
| IdP Entity ID／Issuer | 【要記入】 | 【要記入】 | GSSO担当 |
| IdP SSO URL | 【要記入】 | 【要記入】 | GSSO担当 |
| IdP metadata XML／署名用公開証明書 | 【要記入】 | 【要記入】 | GSSO担当から受領 |
| AuthnRequestへの署名要否 | 【要記入】 | 【要記入】 | GSSOの要件とSP設計による |
| NameIDの取得元・形式 | 【要記入】 | 【要記入】 | GSSO担当とEDP-Webアプリ担当 |
| その他必要なAttributeの名前・取得元 | 【要記入／なし】 | 【要記入／なし】 | GSSO担当とアプリ担当 |
| EDP-Webのユーザー照合項目 | 【要記入】 | 【要記入】 | EDP-Webアプリ担当 |
| 利用者の割当範囲／テストアカウント | 【要記入】 | 【要記入】 | GSSO／業務担当 |
| 外部接続申請番号・承認状況 | 【要記入】 | 【要記入】 | 申請／ネットワーク担当 |
| GSSO用SIR／ServiceNowの記録 | 【要記入】 | 【要記入】 | 申請／GSSO担当 |
| 証明書の有効期限・更新通知先 | 【要記入】 | 【要記入】 | GSSO／基盤／運用担当 |
| 切替・切戻し・受入確認の担当 | 【要記入】 | 【要記入】 | プロジェクト担当 |

**値の照合：**SP Entity IDはEntra側のIdentifierと一致させる。SPの外部公開ACS URLはEntra側のReply URLと一致させる。SPはGSSOから受領した実際のIdP Issuerと署名証明書を使って検証する。NameID／Attributeの実際の値がEDP-Webのユーザーと対応することを確認する。Entity IDはシステムを識別する値であり、ブラウザーで開けるURLである必要はない。

**環境上の注意：**既存のGSSO FAQによると、Entra ID側に独立したQA環境はなく、SPのQA環境を所定の方法でEntra本番IdPに接続して検証できる。ただし、QAのEntity ID、ACS、テストユーザーを本番用の値として扱ってよいという意味ではない。登録方法と利用者の範囲はGSSO担当の回答に従う。

## 1. SPの実装箇所を確定する（プロジェクト／基盤／アプリ担当）

1. EDP-Webの最新の目標構成図、ログイン処理設計、実際のプロジェクトを開く。
2. ブラウザーからPOSTされるSAML Responseを受信する箇所を構成図上で示し、コンポーネント名、バージョン、設定ファイルまたは管理画面、担当者を記入する。
3. ACSをそのコンポーネントが受けるか、検証後の信頼できるユーザー情報をどのようにEDP-Webアプリケーションへ渡すかを確定する。
4. SP起点、IdP起点、または両方のログインが必要かを決め、SAMLによるシングルログアウトを使用するかも決める。
5. 第0章に従い、QA／本番のEntity ID、ACS URL、ログインURLを確定する。既存のSP製品を使う場合は、製品が実際に生成するエンドポイントを先に確認してから申請書に記入する。

**完了条件：**「ブラウザー→SP→GSSO／Entra ID→ACS→EDP-Webのユーザー・権限確認」の経路が図で説明でき、各処理箇所の担当者が決まっている。

**【作業停止】**SPをアプリケーション、リバースプロキシ、共通PFのどれが担うか未決定の場合、以降のEntra画面は調査の参考にできるが、第4章のSpring設定例をEDP-Webの実装方式として採用しない。

## 2. 事前申請を確認し、SIRを提出する（申請担当）

1. EDP-Web自身の外部接続申請、承認資料、有効期限、適用範囲を取得し、QA／本番で必要な通信を確認する。他システムの承認番号は使用しない。
2. 最新版のGSSO用SIRを開く。表紙・確認事項、基本情報、システム構成図を記入し、第0章のSP Entity ID、ACS URL、認証方式、ユーザー識別子、担当者情報を転記する。
3. SP metadata XMLが求められる場合、SP側で出力して添付する。XMLがない場合は、GSSO FAQに示されたEntity ID、ACS、認証方式などの主要項目による申請方法を使い、GSSO担当に受理可能か確認する。
4. GSSO指定のServiceNowの分類・添付資料に従って提出し、申請番号、受付状況、補正事項、承認結果を記録する。

**完了条件：**EDP-Web固有のSIRと申請記録を追跡でき、申請書のEntity ID／ACSがSPの最終設定と一致する。

**【作業停止】**様式や他システムの記入例しかない場合、「EDP-Webは申請済み／承認済み」と記載しない。

## 3. GSSO／Entra ID側を設定する（権限を持つGSSO担当）

以下はMicrosoft Entra管理センターの**画面操作**の経路である。社内GSSOに専用の受付・設定手順がある場合は、その運用に従う。EDP-Webアプリ担当は設定値を提出し、返却された設定結果を照合する。

1. https://entra.microsoft.com を開き、**Entra ID → Enterprise applications → All applications**へ進む。本システムのエンタープライズアプリケーションを検索し、表示名と登録状況を記録する。見つからない場合はGSSOの申請手順で作成担当を確認し、独断で重複登録しない。
2. 対象アプリケーションの**Single sign-on → SAML → Basic SAML Configuration → Edit**を開く。
3. **Identifier (Entity ID)**に当該環境のSP Entity ID、**Reply URL (ACS URL)**に当該環境のHTTPS ACS URLを入力する。SP起点のログインを使用し、この項目が適用される場合は、**Sign on URL**に当該環境のSPログイン入口を入力して保存する。
4. **Attributes & Claims → Edit**で、NameIDの取得元と形式を第0章で決めた値に設定する。EDP-Webが実際に必要とするAttributeのみを追加し、claim名と取得元を一つずつ照合して保存する。メールアドレスやUPNを無条件にEDP-WebのユーザーIDと見なさない。
5. 利用範囲に従い、**Users and groups**でQAのテストユーザー／グループを割り当てるか、GSSOのアクセスポリシーを設定する。割当対象外のユーザーに期待する動作も記録する。
6. 対象アプリケーションの**SAML Certificates**と設定画面から、IdP署名用公開証明書、IdP Identifier、Login URL、必要に応じてアプリケーション固有のmetadataを取得する。所定の方法でSP担当へ渡し、有効期限と更新通知先を記録する。

**完了条件：**GSSO側から、実際に登録されたIdentifier、Reply URL、NameID／Attribute、IdP Issuer／SSO URL、証明書、ユーザー範囲が返却されている。申請書だけを根拠に、管理画面での設定が完了したと判断しない。

Microsoftのページにあるcontoso／SAML ToolkitのサンプルURLを流用しない。テナント共通のmetadata URLから本システムの証明書や実際のIssuerを推測せず、GSSOが対象アプリケーション向けに引き渡した情報を使う。

## 4. SP側を設定し、ユーザーを処理する（EDP-Web基盤／アプリ担当）

1. 第1章で選定したSPコンポーネントに、GSSOのIdP Entity ID、SSO URL、署名用公開証明書／metadataを登録する。
2. 同じコンポーネントにSP Entity ID、ACS、要求方式、署名要否を設定する。生成されたAuthnRequestのIssuerがGSSOに登録したIdentifierと、ACSがReply URLと一致することを確認する。Microsoftの説明では、SP起点の要求は通常ブラウザーのRedirectでIdPへ送られ、認証後のSAML ResponseはブラウザーからACSへPOSTされる。
3. SPコンポーネントで署名と証明書の信頼性、Issuer、Audience、Destination／Recipient、時刻条件、InResponseTo、重複使用などを検証する。これらは本プロジェクトの設計要件である。**ライブラリを導入しただけで、すべての検証やリプレイ防止が満たされるとは判断しない。**コンポーネントの動作確認と異常系テストを行う。
4. 検証に成功した場合だけ、取り決めたNameID／Attributeをアプリケーションに渡す。アプリケーション側では確定したルールに従ってEDP-Webユーザーを検索し、存在・業務利用可否・Roleを確認してから本システムのSessionを作成する。本人認証の成功は業務権限の付与を意味しない。
5. 既存のローカルログイン入口、遷移、Session、Logout、未登録ユーザー、エラー表示を改修対象として洗い出す。QAの受入前に旧認証を停止しない。

**完了条件：**SPからログインを開始し、ACSで応答を受信・検証したうえで、正しいEDP-WebユーザーとRoleで所定の画面に入れる。失敗時には有効なSessionが作成されない。

### 条件付きの設定例：Spring Boot 3.5＋Spring Security 6.5でアプリケーション本体がSPを担う場合

この小節は上記の技術構成のServletアプリケーションに限る。既存工程に独自のSecurityFilterChainや別の認証フレームワークがある場合、またはプロキシがSPを担う場合は、現行設計に合わせて統合する必要があり、そのまま貼り付けて使用しない。角括弧内はすべてプレースホルダーである。

- Mavenのpom.xmlに追加する依存関係：

    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-saml2-service-provider</artifactId>
    </dependency>

- QA用の管理された設定ファイル（例：application-qa.yml）には、次のような設定を記載する。証明書のパスは管理された場所の**IdP公開証明書**を指す。SPの秘密鍵を引継資料に記載しない。

    spring:
      security:
        saml2:
          relyingparty:
            registration:
              edp-gsso:
                entity-id: "[SP_ENTITY_ID_QA]"
                acs:
                  location: "[SP_ACS_URL_QA]"
                assertingparty:
                  entity-id: "[IDP_ENTITY_ID]"
                  singlesignon:
                    url: "[IDP_SSO_URL]"
                    sign-request: false
                  verification:
                    credentials:
                      - certificate-location: "file:/[APPROVED_PATH]/idp-signing.cer"

- この例のregistrationIdはedp-gssoで固定する。Spring Securityの標準ACS処理パスを使う場合、GSSOに登録する外部ACSは通常、https://[QA_HOST]/login/saml2/sso/edp-gsso の形式となる。SP起点の入口は通常、https://[QA_HOST]/saml2/authenticate/edp-gsso の形式となる。実際のURLはリバースプロキシの外部公開HTTPSホスト、パスのプレフィックス、転送設定と照合する。Entity IDは個別に指定できるため、標準生成値と申請書に記入した値を混同しない。
- sign-request: falseは、**GSSOが署名済みAuthnRequestを要求しない場合だけ**の例である。署名が必要なら署名証明書と秘密鍵の管理方法を先に決め、製品の正式な手順に従って設定する。falseをそのまま本番へ適用しない。
- アプリケーションに既存のSecurityFilterChainがある場合、saml2Loginが組み込まれ、ACSのパスをSAML Filterが処理し、既存の業務認可とSessionの仕組みと両立することをコード担当が確認する。依存関係と設定を追加した後は、プロジェクト既定の方法でビルド・起動を確認する。画面操作ではIDEのMavenツールウィンドウからReload／Buildを実行できる。YAMLだけでは業務ユーザーの照合が完成したことにはならない。
- 本番には**別の**Entity ID、ACS、公開証明書／IdP情報、管理された環境設定を使用する。QAの設定一式をそのまま本番にコピーしない。

## 5. QA連携検証：順番に操作し、証跡を残す

**開始条件：**SIRが受け付けられ、GSSO側の登録情報を受領済みで、QAのSPをブラウザーから利用できる。テストユーザーが割り当てられ、EDP-Web側にも対応するユーザーとRoleが存在する。GSSO FAQのとおりEntra ID側に独立したQA環境がない場合、QA SPの登録方法とテスト範囲をGSSO担当に確認してから進める。

| 順番 | 操作 | 期待結果／残す証跡 |
| --- | --- | --- |
| 1 | ブラウザーでQAのSP起点ログインURLを開く | GSSOへ遷移する。AuthnRequestのIssuer＝QAのEntity ID、ACS＝登録済みのQAのReply URL |
| 2 | 割当済みで業務権限のあるテストユーザーでログインする | ブラウザーがQAのACSへPOSTし、SPの検証が成功してEDP-Webの想定画面に入れる |
| 3 | 受信したNameID／必要なAttributeを本システムのユーザーと照合する | ユーザーが一意に正しく対応し、本システムで許可されたRoleだけが付与される |
| 4 | 未登録または業務権限のないユーザーでログインする | 本人認証と業務認可が設計どおり別々に処理され、保護対象の機能を利用できない |
| 5 | 未割当／対象範囲外のユーザーでログインする | GSSOまたはSPのアクセスポリシーに従って拒否され、迂回できない |
| 6 | 署名／Issuer／Audience／ACSが不正、応答の期限切れ、リプレイを試験する | SPが拒否し、Sessionを作成しない。ログから原因を特定できる |
| 7 | Session失効、保護対象URLへの直接アクセス、Logoutを試験する | 未ログインではアクセスできず、ログアウト後の動作が決定済みの範囲に一致する |
| 8 | ログ、証明書の有効期限、ネットワーク経路、切戻し準備を確認する | 障害を調査でき、通常のログにSAMLアサーション全体や機微な属性を記録しない |

SAML応答の検証では、Entraのエンタープライズアプリケーションの**Single sign-on → Test**からプラットフォーム側のテストを行うことができる。それに加え、EDP-Webの実際のSPログイン入口からエンドツーエンドで試験する。前者の成功だけで、業務ユーザー照合、Role、Sessionが確認できたことにはならない。ブラウザー拡張が許可されない場合は、プロジェクトで認められたログや通信調査の手段を使う。

**受入記録に最低限必要な内容：**環境、テストユーザーの区分、実施日時、Entity ID／ACSの実際の値、結果、失敗理由、証跡の場所、実施者、確認者。SAML Response全体や証明書の秘密鍵を報告書に貼り付けない。

### よくある不具合の最初の確認箇所

| 現象 | 最初に確認する内容 |
| --- | --- |
| GSSOに到達する前に失敗する | SPのログイン入口、IdP SSO URL、AuthnRequestのIssuer、要求への署名要否 |
| GSSOで認証に失敗する／アプリケーションが見つからない | アプリケーションの登録、Identifier、ユーザー割当／アクセス範囲、GSSOのサインインログ |
| 認証後にシステムへ戻らない／ACSが404になる | Reply URLと実際のSP ACS、プロキシの経路、外部公開HTTPSホストとパス |
| ACSは受信するが検証に失敗する | IdP Issuer、署名証明書、Audience、Destination、時刻、InResponseTo |
| SAML検証は成功するが業務機能を利用できない | NameID／AttributeとEDP-WebユーザーIDの対応、ユーザー状態、Role、Session |
| QAは成功し、本番だけ失敗する | Entity ID／ACS／証明書／アプリケーション登録／転送設定が各環境で正しく対応しているか |

## 6. 本番切替（プロジェクト／GSSO／SP／運用担当）

1. QAの**作業手順**を踏襲し、値はコピーしない。本番用の設定値一覧に従い、申請、Entra登録、SP設定、証明書、ユーザー範囲を再確認する。
2. プロジェクト担当がQAの証跡、切替時間帯、担当者、承認、切戻し条件を確認する。旧入口や旧設定へ戻す具体的な方法を先に準備する。
3. GSSO担当が本番のエンタープライズアプリケーションの設定完了を、SP担当が外部公開ACSの利用可否を、アプリ担当がユーザー・権限データを確認する。
4. 承認された時間帯に本番のスモークテストを行う。正常ログイン、権限不足、業務画面、ログアウト、ログを確認し、完了条件を満たしてから切替計画に従って旧ローカル認証を停止する。
5. 運用担当が証明書の有効期限、更新通知とローテーション方法、障害時の連絡先、ログ保管、監視方法を記録する。

**完了条件：**本番の証跡と承認記録が揃い、ユーザーは旧ローカルパスワード認証を使用しない。異常時には実行可能な切戻し手順がある。

## 7. 設定値の記入だけでは埋められない決定事項

- **SPを担うコンポーネント**とバージョン、ACSの配置箇所、既存のログイン処理・Session処理との統合箇所。
- **GSSOの正式な申請・管理権限**：SIRの提出者、エンタープライズアプリケーションの作成・変更者、AuthnRequestへの署名要否、QAの登録方法。
- **ユーザー照合・業務認可のルール**：実際のNameID／AttributeをEDP-Webのどの項目に対応させ、未登録／権限不足をどう処理するか。
- **コンポーネントの検証能力と運用設計**：特にリプレイ防止、証明書更新、プロキシ転送、ログアウト、切戻し。

これらの決定事項と第0章の実際の値が揃えば、共通手順を現場の作業記録として使用できる。SPをEDP-WebのSpring Bootアプリケーション本体が担うと確認できた場合も、既存工程、Spring Boot／Securityのバージョン、現行のSecurityFilterChainを確認して初めて、第4章の例を「そのまま工程に組み込んで動かせる」コード単位の手順へ具体化できる。

## 参考資料

- [Microsoft：SAMLによるシングルサインオンのプロトコル](https://learn.microsoft.com/zh-cn/entra/identity-platform/single-sign-on-saml-protocol)：AuthnRequest、Response、バインディング、主な項目。
- [Microsoft：エンタープライズアプリケーションの表示](https://learn.microsoft.com/zh-cn/entra/identity/enterprise-apps/view-applications-portal)：Entra管理センターでのアプリケーション検索。
- [Microsoft：SAMLプロトコルリファレンス](https://learn.microsoft.com/zh-cn/entra/identity-platform/saml-protocol-reference)：IdP／SPの情報交換とメタデータ。
- [Microsoft：エンタープライズアプリケーションのSAML SSO設定](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/add-application-portal-setup-sso)：Entraの画面操作、基本SAML設定、証明書、テスト。
- [Microsoft：SAML Claimsのカスタマイズ](https://learn.microsoft.com/en-us/entra/identity-platform/saml-claims-customization)：NameID、属性の取得元・形式。
- [Microsoft：SAML SSOのデバッグ](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/debug-saml-sso-issues)：プラットフォーム側のテストとエラー調査。
- [Spring Boot 3.5：Spring Security／SAML 2.0](https://docs.spring.io/spring-boot/3.5/reference/web/spring-security.html)および[Spring Security：SAML 2.0 Login](https://docs.spring.io/spring-security/reference/servlet/saml2/login/overview.html)：条件付きのアプリケーションSP設定例。バージョンが異なる場合は、実際の工程のバージョンに対応する資料を参照する。
