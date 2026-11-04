# EDP-Web × GSSO／Microsoft Entra ID：SAML 接入填值手顺

> 2026-11-04 整理。用途：在项目已经决定 SP 实装组件、QA／本番参数和担当后，按顺序完成申请、双方配置、联调与验收。本文没有使用其他系统的示例值，也不代表 EDP-Web 已提交申请或完成实装。

## 先看结论：这份手顺能直接做到哪一步

Microsoft 的三个参考页分别说明：SAML 的 AuthnRequest／Response、如何在 Entra 管理中心查找企业应用、IdP／SP 交换元数据和端点的原则。它们**没有提供 EDP-Web 专属的组件、代码、用户表和 GSSO 内部申请记录**。因此，无法保证“只换几个值就能在任何架构中实装”。

本文提供两层：

1. **共通执行手顺**：只要项目参数和担当都确定，GSSO／Entra 配置、资料交换、QA 验证和本番切换可逐项照做。
2. **条件式 SP 范例**：仅当“EDP-Web 是 Spring Boot 3.5＋Spring Security 6.5 的 Servlet 应用，而且程序本身承担 SP”时，可参考文末的配置范例。若 SP 在反向代理或共通 PF，不使用这段程序配置，改按该产品的管理手册填写同一组参数。

文中【待填】是实际执行前必须填入的项目值；【停止】表示缺少设计决定或权限，不能靠猜测继续。

## 0. 一次性填值表

先复制此表，分别做 QA 和本番两份。不要把其他系统的 URL、租户 ID 或申请号复制过来。证书栏只记录**公开证书的受控交付位置**，不在本表贴私钥。

| 项目 | QA 值 | 本番值 | 由谁确定／提供 |
| --- | --- | --- | --- |
| SP 实装组件和版本 | 【待填】 | 【待填】 | EDP-Web 基盘／应用担当 |
| 对外入口 URL（HTTPS） | 【待填】 | 【待填】 | EDP-Web 基盘担当 |
| SP Entity ID（識別子） | 【待填】 | 【待填】 | SP 担当定义，GSSO 登记 |
| ACS URL（応答URL） | 【待填】 | 【待填】 | SP 担当提供，GSSO 登记 |
| SP 发起登录 URL（如适用） | 【待填】 | 【待填】 | SP 担当提供 |
| SP metadata XML／URL（如有） | 【待填／无】 | 【待填／无】 | SP 担当 |
| IdP Entity ID／Issuer | 【待填】 | 【待填】 | GSSO 担当 |
| IdP SSO URL | 【待填】 | 【待填】 | GSSO 担当 |
| IdP metadata XML／签名公开证书 | 【待填】 | 【待填】 | GSSO 担当交付 |
| AuthnRequest 是否须签名 | 【待填】 | 【待填】 | GSSO 规则与 SP 设计 |
| NameID 来源、格式 | 【待填】 | 【待填】 | GSSO 与 EDP-Web 应用担当共同确定 |
| 其他必需 Attribute 名称和来源 | 【待填／无】 | 【待填／无】 | GSSO 与应用担当 |
| EDP-Web 用户对应字段 | 【待填】 | 【待填】 | EDP-Web 应用担当 |
| 用户分配范围／测试账号 | 【待填】 | 【待填】 | GSSO／业务担当 |
| 外部接続申請编号与批准状态 | 【待填】 | 【待填】 | 申请／网络担当 |
| GSSO 用 SIR／ServiceNow 记录 | 【待填】 | 【待填】 | 申请／GSSO 担当 |
| 证书有效期和更新通知担当 | 【待填】 | 【待填】 | GSSO／基盘／运维担当 |
| 切换、回退、验收担当 | 【待填】 | 【待填】 | 项目担当 |

**填值核对：**SP Entity ID 必须与 Entra 里的 Identifier 一致；SP 对外 ACS URL 必须与 Entra 里的 Reply URL 一致；SP 验证时使用 GSSO 提供的实际 IdP Issuer 和签名证书；NameID／Attribute 的实际值必须能对应 EDP-Web 的用户。Entity ID 是系统识别值，不要求一定能在浏览器打开。

**环境注意：**现有 GSSO FAQ 说 Entra ID 侧没有独立 QA，SP 的 QA 可按规定接入 Entra 本番 IdP 进行测试。这不等于把 QA 的 Entity ID、ACS 或测试用户直接当成本番值；登记方式和用户范围仍以 GSSO 担当答复为准。

## 1. 固定实装路径（项目／基盘／应用担当）

1. 打开最新的 EDP-Web 目标构成图、登录处理设计和实际工程。
2. 在图上圈出“接收浏览器 POST 的 SAML Response”的位置，并写出组件名称、版本、配置文件或管理画面、负责担当。
3. 确定 ACS 是否由该组件接收，以及验证成功后如何把可信用户身份交给 EDP-Web 应用。
4. 确定是 SP 发起登录、IdP 发起登录，还是都需要；决定是否使用 SAML 单点注销。
5. 对照第 0 节，确定 QA／本番的 Entity ID、ACS URL 和登录 URL。若用既有 SP 产品，先以其实际生成的端点为准，再填申请表。

**完成判定：**可画出“浏览器→SP→GSSO／Entra ID→ACS→EDP-Web 用户与权限检查”的路径，且每个处理点有负责担当。

**【停止】**如果还不知道 SP 在程序、反向代理还是共通 PF，后续 Entra 界面可作为调查参考，但不能把文末 Spring 配置当成 EDP-Web 实装方案。

## 2. 核对前置申请与提交 SIR（申请担当）

1. 取得 EDP-Web 自己的外部接続申請、批准件、有效期和适用范围；核对 QA／本番所需通信。不要使用其他系统的批准号。
2. 打开最新版 GSSO 用 SIR。填写表紙・確認事項、基本情報、システム構成図，并填入第 0 节的 SP Entity ID、ACS URL、认证方式、用户标识及担当信息。
3. 如果要求 SP metadata XML：由 SP 导出并附上；若没有，按 GSSO FAQ 的方式用 Entity ID、ACS、认证方式等主要字段办理，最终由 GSSO 担当确认是否接受。
4. 按 GSSO 指定的 ServiceNow 类别和附件要求提交，记录申请号、受理状态、补正事项、批准结果。

**完成判定：**EDP-Web 专属 SIR 和申请记录可追溯；申请书里的 Entity ID／ACS 与 SP 最终配置一致。

**【停止】**只有样表或其他系统的填写例，不能写成“EDP-Web 已申请／已批准”。

## 3. GSSO／Entra ID 侧配置（由有权限的 GSSO 担当操作）

以下为 Microsoft Entra 管理中心的**图形界面**路径。公司 GSSO 若封装了专用受理和设置流程，以其实际管理流程执行；EDP-Web 应用担当只提交参数并核对回传结果。

1. 打开 https://entra.microsoft.com ，进入 **Entra ID → Enterprise applications → All applications**，搜索本系统的企业应用。记录显示名称和登记状态。若不存在，先通过 GSSO 申请流程确认由谁创建，勿自行重复建一份。
2. 打开该应用的 **Single sign-on → SAML → Basic SAML Configuration → Edit**。
3. 填 **Identifier (Entity ID)**＝本环境的 SP Entity ID；填 **Reply URL (ACS URL)**＝本环境的 HTTPS ACS URL；若采用 SP 发起且此字段适用，填 **Sign on URL**＝本环境的 SP 登录入口。保存。
4. 在 **Attributes & Claims → Edit**，把 NameID 的来源和格式设为第 0 节已确定的值；仅添加 EDP-Web 实际需要的 Attribute，逐一核对 claim 名称与来源。保存。不要想当然地把邮箱或 UPN 当成 EDP-Web 用户 ID。
5. 按使用范围在 **Users and groups** 分配 QA 测试用户／组，或按 GSSO 的访问策略配置。记录未分配用户的预期行为。
6. 从该应用的 **SAML Certificates** 和设置区取得 IdP 签名公开证书、IdP Identifier、Login URL、需要时的应用专属 metadata。将这些资料按规定交给 SP 担当，并记下有效期与轮换通知对象。

**完成判定：**GSSO 侧可回传“实际已登记的 Identifier、Reply URL、NameID／Attribute、IdP Issuer／SSO URL、证书和用户范围”。不得仅凭申请表推断后台已经配置好。

**不要直接套用** Microsoft 页面里的 contoso／SAML Toolkit 示例 URL；也不要自行用租户通用 metadata URL 推测本系统的证书和实际 Issuer，应以 GSSO 对本企业应用的交付内容为准。

## 4. SP 侧配置与用户处理（EDP-Web 基盘／应用担当）

1. 在第 1 节选定的 SP 组件中登记 GSSO 的 IdP Entity ID、SSO URL、签名公开证书／metadata。
2. 在同一组件设置 SP Entity ID、ACS、请求方式和签名要求。确认生成的 AuthnRequest 中 Issuer 与 GSSO 登记的 Identifier 一致，ACS 与 Reply URL 一致。Microsoft 说明 SP 发起请求通常经浏览器 Redirect 到 IdP，认证后 SAML Response 由浏览器 POST 到 ACS。
3. 由 SP 组件校验签名和证书信任、Issuer、Audience、Destination／Recipient、时间条件、InResponseTo 与重复使用等。这里是项目设计要求；**不能只因装了某个库就假定全部校验或重放防止已满足**，必须确认组件行为并做异常测试。
4. 只有校验成功后，才将约定的 NameID／Attribute 交给应用。应用按已确认的规则查询 EDP-Web 用户，判断是否存在、是否有业务利用资格和 Role，再建立本系统 Session。认证通过不等于业务权限通过。
5. 把现行本地登录入口、跳转、Session、Logout、未注册用户和异常提示列入改修范围。QA 验收前不要提前停用旧认证。

**完成判定：**可从 SP 发起一次登录，到 ACS 收到并验证响应，再以正确的 EDP-Web 用户和 Role 进入页面；失败情况不会创建有效 Session。

### 条件式配置范例：Spring Boot 3.5＋Spring Security 6.5，由程序直接承担 SP

这一小节只适用于上述技术组合及 Servlet 应用。若现有工程有自己的 SecurityFilterChain、其他认证框架或代理承担 SP，必须按现行设计整合，不能直接贴配置。下面的方括号全是占位符。

- 工程依赖（Maven 的 pom.xml 中）：

    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-saml2-service-provider</artifactId>
    </dependency>

- 在 QA 的受控配置文件（例如 application-qa.yml）中，使用类似下列配置。证书路径指向受控位置中的 **IdP 公开证书**；不要把 SP 私钥放进交付手册。

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

- 此范例固定 registrationId 为 edp-gsso。采用 Spring Security 默认 ACS 处理路径时，登记给 GSSO 的外部 ACS 通常形如 https://[QA_HOST]/login/saml2/sso/edp-gsso；SP 发起入口通常形如 https://[QA_HOST]/saml2/authenticate/edp-gsso。实际 URL 还要核对反向代理的外部 HTTPS 主机、路径前缀和转发配置。Entity ID 可单独指定，不能把默认生成值和申请表填入值混用。
- sign-request: false **仅是“GSSO 不要求已签名 AuthnRequest”时的范例值**。如 GSSO 要求签名，先取得签名证书／密钥管理方案，改按产品正式配置，不应把 false 照抄上线。
- 若应用已有 SecurityFilterChain，需由代码担当确认 saml2Login 已接入、ACS 路径由 SAML Filter 处理、现有业务权限和 Session 逻辑相容。新增依赖／配置后，用项目既有构建与启动方式验证；图形界面可在 IDE 的 Maven 工具窗口执行 Reload／Build。不能仅凭 YAML 证明业务用户映射已完成。
- 本番采用**独立**的 Entity ID、ACS、公开证书／IdP 资料和受控环境配置；切勿直接把 QA 整份配置复制到本番。

## 5. QA 联调：按顺序点，逐项留证据

**开始条件：**SIR 已受理、GSSO 登记已回传、QA SP 可从浏览器访问、测试用户已分配且 EDP-Web 中有对应用户／Role。GSSO FAQ 所述 Entra ID 无独立 QA 时，先让 GSSO 担当确认 QA SP 的注册方法及测试范围。

| 顺序 | 操作 | 预期结果／留存证据 |
| --- | --- | --- |
| 1 | 浏览器打开 QA 的 SP 发起登录 URL | 跳转到 GSSO；AuthnRequest 的 Issuer＝QA Entity ID，ACS＝登记的 QA Reply URL |
| 2 | 用已分配且有业务权限的测试用户登录 | 浏览器 POST 到 QA ACS；SP 校验成功；进入 EDP-Web 的预期页面 |
| 3 | 核对接收到的 NameID／必要 Attribute 与本系统用户 | 用户对应唯一且正确；只授予本系统允许的 Role |
| 4 | 用未注册或无业务权限的用户登录 | 认证与业务授权按设计分别处理；不能进入受保护功能 |
| 5 | 用未分配／不在允许范围的用户登录 | GSSO 或 SP 按访问策略拒绝，不得绕过 |
| 6 | 测试错误的签名／Issuer／Audience／ACS／过期响应／重放 | SP 拒绝，不建立 Session；核对日志中可定位原因 |
| 7 | 测试 Session 失效、直接访问受保护 URL、Logout | 未登录无法访问；退出后的行为符合确认的注销范围 |
| 8 | 核对日志、证书有效期、网络路径和回退准备 | 可定位故障但不在普通日志完整记录 SAML 断言或敏感属性 |

测试 SAML 响应时可在 Entra 企业应用的 **Single sign-on → Test** 发起平台侧测试，也要从 EDP-Web 的真实 SP 登录入口做端到端测试。前者通过不代表业务用户对应、Role 和 Session 已通过。若项目不允许安装浏览器扩展，用平台许可的日志和抓包方式定位问题。

**验收记录最少包括：**环境、测试用户类别、测试时间、Entity ID／ACS 的实际值、结果、失败原因、证据位置、执行者和确认者。不要在报告里粘贴完整 SAML 响应或证书私钥。

### 常见故障先查哪里

| 现象 | 第一核对点 |
| --- | --- |
| 进入 GSSO 前就失败 | SP 登录入口、IdP SSO URL、AuthnRequest Issuer、请求签名要求 |
| GSSO 认证失败／找不到应用 | 应用是否存在、Identifier、用户分配／访问范围、GSSO 登录日志 |
| 认证后回不到系统／ACS 404 | Reply URL 与 SP 实际 ACS、代理路由、外部 HTTPS 主机与路径 |
| ACS 收到但校验失败 | IdP Issuer、签名证书、Audience、Destination、时间、InResponseTo |
| SAML 验证成功但无法使用业务功能 | NameID／Attribute 与 EDP-Web 用户 ID 的对应、用户状态、Role、Session |
| QA 成功、本番失败 | 两套 Entity ID／ACS／证书／应用登记与转发设置是否逐项对应 |

## 6. 本番切换（项目／GSSO／SP／运维担当）

1. 复制**步骤**，不复制 QA 的值；以本番填值表重新核对申请、Entra 登记、SP 设置、证书、用户范围。
2. 项目担当确认 QA 证据、切换窗口、负责人与批准、回退条件。先准备恢复旧入口或旧配置的具体方法。
3. GSSO 担当确认本番企业应用设置完成，SP 担当确认对外 ACS 可用，应用担当确认用户与权限数据。
4. 在批准的窗口做本番冒烟测试：正常登录、无权限、业务页面、退出与日志；符合完成条件后再按切换计划停用旧本地认证。
5. 运维记录证书有效期、更新通知和轮换方法、故障联络、日志保管与监控。

**完成判定：**本番证据和批准记录齐全；用户不再走旧本地密码认证，异常时有可执行的回退路径。

## 7. 这份手顺目前还差的“不能靠填值替代”的决定

- **SP 所在组件**及版本、ACS 所属、现有登录和 Session 代码的整合点。
- **GSSO 的正式申请与后台权限**：谁提交 SIR、谁建／改企业应用、是否要求签名 AuthnRequest、QA 如何登记。
- **用户对应和业务权限规则**：实际 NameID／Attribute 对应哪个 EDP-Web 字段，未注册／无权限如何处理。
- **组件的验证能力与运维设计**：特别是重放防止、证书更新、代理转发、注销、回退。

这些决定和第 0 节的实际值都填完后，本文的共通步骤可以变成现场执行记录。若确认 SP 确实由 EDP-Web 的 Spring Boot 程序承担，还需拿现有工程、Spring Boot／Security 版本和现行 SecurityFilterChain，才能把第 4 节范例改成“直接放入工程并运行”的代码级手顺。

## 参考来源

- [Microsoft：单一登录 SAML 协议](https://learn.microsoft.com/zh-cn/entra/identity-platform/single-sign-on-saml-protocol)：AuthnRequest、Response、绑定与主要字段。
- [Microsoft：查看企业应用程序](https://learn.microsoft.com/zh-cn/entra/identity/enterprise-apps/view-applications-portal)：在 Entra 管理中心查找应用。
- [Microsoft：SAML 协议参考](https://learn.microsoft.com/zh-cn/entra/identity-platform/saml-protocol-reference)：IdP／SP 信息交换和元数据。
- [Microsoft：企业应用的 SAML SSO 配置](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/add-application-portal-setup-sso)：Entra 图形界面、基本 SAML 配置、证书和测试。
- [Microsoft：自定义 SAML Claims](https://learn.microsoft.com/en-us/entra/identity-platform/saml-claims-customization)：NameID、属性来源与格式。
- [Microsoft：调试 SAML SSO](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/debug-saml-sso-issues)：平台测试与失败定位。
- [Spring Boot 3.5：Spring Security／SAML 2.0](https://docs.spring.io/spring-boot/3.5/reference/web/spring-security.html) 与 [Spring Security：SAML 2.0 Login](https://docs.spring.io/spring-security/reference/servlet/saml2/login/overview.html)：条件式程序 SP 范例；版本不匹配时以工程实际版本的文档为准。
