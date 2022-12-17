---
layout: blog
date: 2022-12-17T14:26:13.775Z
title: Web IDLのdiffで読むWebAuthn仕様の最新の変化
---
この記事は[Digital Identity技術勉強会 #iddance Advent Calendar 2022](https://qiita.com/advent-calendar/2022/iddance "https\://qiita.com/advent-calendar/2022/iddance") 17日目の記事です。今年の[ Advent Calendar](https://qiita.com/advent-calendar/2022/iddance "https\://qiita.com/advent-calendar/2022/iddance") では、[WebAuthn/Passkeyを扱った素晴らしい記事が他にあり](https://qiita.com/kokukuma/items/7e4856b616506d1e3618)、何を書こうか迷いましたが、最近あまり追いかけられていなかったWebAuthn仕様の最新の変化をこの機会に追いかけてみようかと思います。

ところで、皆さんはWebAuthnしかり、どんどん改訂が進む仕様書を追いかける際に、どのように改定内容をチェックしていますか？

一番確実なのは仕様書の改訂が議論されている場（WebAuthnであれば、GitHubのIssue）をウォッチし続けることで、議論の背景・経緯等も掴めて最も望ましい方法ですが、相当な分量になる為負担も大きく、中々容易ではありません。

## 仕様書のDiffを読みたい

中々MLやフォーラムでの議論を追いかけられていないけど、キャッチアップの為に、以前通読したWorking Draftと、最新のEditor's Draftの差分を知りたいなぁという場合、あるのではないでしょうか。

IETFのInternet Draft/RFCだと [IETF Author Tools - iddiff](https://author-tools.ietf.org/iddiff "https\://author-tools.ietf.org/iddiff") というツールでdiffの出力が出来ますが、W3Cの仕様書の場合、W3Cから [HTML Diff service](https://services.w3.org/htmldiff) というツールが提供されており、そちらでdiffを取ると良いでしょう。

例えば、[WebAuthn Level3 WD1と最新のEditor's Draftの差分](https://services.w3.org/htmldiff?doc1=https%3A%2F%2Fwww.w3.org%2FTR%2F2021%2FWD-webauthn-3-20210427%2F&doc2=https%3A%2F%2Fw3c.github.io%2Fwebauthn%2F "https\://services.w3.org/htmldiff?doc1=https%3A%2F%2Fwww.w3.org%2FTR%2F2021%2FWD-webauthn-3-20210427%2F&doc2=https%3A%2F%2Fw3c.github.io%2Fwebauthn%2F") もこのように簡単に出力できます。このDiff Serviceは、W3CのGitHubレポジトリに対してPull-Requestを送った際に自動で出力されるdiffプレビューの出力でも使用されています。

さて、Diff自体は簡単に出力出来るツールが揃っている状況ですが、実際 [WebAuthn Level3 WD1と最新のEditor's Draftの差分](https://services.w3.org/htmldiff?doc1=https%3A%2F%2Fwww.w3.org%2FTR%2F2021%2FWD-webauthn-3-20210427%2F&doc2=https%3A%2F%2Fw3c.github.io%2Fwebauthn%2F "https\://services.w3.org/htmldiff?doc1=https%3A%2F%2Fwww.w3.org%2FTR%2F2021%2FWD-webauthn-3-20210427%2F&doc2=https%3A%2F%2Fw3c.github.io%2Fwebauthn%2F") を眺めてみて如何だったでしょうか。全部通読するよりマシとは言え、まだまだ結構分量あるな、追加された部分、削除された部分入り乱れていて文章が読みにくいな、と感じた人も多いかと思います。何かいい方法は無いのでしょうか。

## IDLのdiffを読もう

仕様書全体のDiffを読むのが大変な場合は、WebAuthnのようにW3Cの仕様書の場合、IDLのdiffから読むのがお勧めです。W3Cの仕様書では、仕様書内のインタフェース定義をWeb IDLという言語で記述しています。WebAuthn仕様の場合、末尾にIDL Indexとして登場したインタフェースのまとめがあり、その部分のdiffを取ることで、仕様に新たに追加されたAPI、削除されたAPIを簡単に把握することが可能です。そこから、興味を持った変更について、仕様書のDiffや最新のEditor's Draftの該当箇所を掘っていくようにすると効率的に仕様のキャッチアップが可能です。勿論、Web IDLの表面に現れない仕様変更については取りこぼすことになりますが、そこはトレードオフとして諦めるか、一旦IDLのdiffを取っかかりに全体感を掴んだ上で、仕様書全体のdiffを読むと良いでしょう。

## WebAuthn IDLのdiff

では、WebAuthn IDLには、去年と比べてどのようなdiffがあったのでしょうか。WebAuthn Level3 WD1と最新のEditor's Draftのdiffを各データ型毎に一つ一つ見ていきましょう。

### PublicKeyCredential

```
 interface PublicKeyCredential : Credential {
     [SameObject] readonly attribute ArrayBuffer              rawId;
     [SameObject] readonly attribute AuthenticatorResponse    response;
+    [SameObject] readonly attribute DOMString?               authenticatorAttachment;
     AuthenticationExtensionsClientOutputs getClientExtensionResults();
+    static Promise<boolean> isConditionalMediationAvailable();
+    PublicKeyCredentialJSON toJSON();
+};
```

#### authenticatorAttachment

まず、authenticatorAttachmentがPublicKeyCredentialに追加されています。authenticatorAttachmentは元々、AuthenticatorSelectionCriteriaのメンバで、クレデンシャルを作成する際に、Platform Authenticatorか、Cross-Platform Authenticatorかを限定する為に使用されていましたが、PublicKeyCredentialのメンバにもなったことで、登録時や認証時にAuthenticatorがPlatform Authenticatorなのか、Cross-Platform Authenticatorなのか確認できるようになりました。

I﻿ssue: [Authenticator Attachment in Public Key Credential · Issue #1666 · w3c/webauthn](https://github.com/w3c/webauthn/issues/1666)

P﻿ull-Request: [Add authenticator attachment used during authentication to assertion payload by z11h · Pull Request #1668 · w3c/webauthn](https://github.com/w3c/webauthn/pull/1668)

#### isConditionalMediationAvailable()

Conditional UIが使用できるかどうかを示すstaticメソッドとしてisConditionalMediationAvailable()が追加されました。Conditional UIについては、去年のAdvent Calendarの記事で触れているので、そちらを参照下さい。

#### toJSON()

PublicKeyCredentialのJSON表現を返すメソッドとして、toJSON()メソッドが追加されました。Relying PartyとしてWebAuthnのJS APIを呼び出して得られたPublicKeyCredentialは、Relying Partyのサーバーサイドに対して送信する必要がありますが、PublicKeyCredentialのメンバにはArrayBufferが含まれている一方で、標準のJSのAPIにはArrayBufferをbase64url等にエンコードするAPIが無かったため、サードパーティーのライブラリを導入する等、シリアライズにこれまでひと手間必要でした。

今回、toJSON()メソッドが導入されたことで、Relying Partyのクライアント側を記述するのがこれから少し楽になりそうです。

戻り値のPublicKeyCredentialJSONの定義は、以下のように追加されています。PublicKeyCredentialでArrayBufferとして表現されていた箇所がBase64URLStringに変更されたものと捉えると良さそうです。

```
+
+typedef DOMString Base64URLString;
+typedef (RegistrationResponseJSON or AuthenticationResponseJSON) PublicKeyCredentialJSON;
+
+dictionary RegistrationResponseJSON {
+    Base64URLString id;
+    Base64URLString rawId;
+    AuthenticatorAttestationResponseJSON response;
+    DOMString? authenticatorAttachment;
+    AuthenticationExtensionsClientOutputsJSON clientExtensionResults;
+    DOMString type;
+};
+
+dictionary AuthenticatorAttestationResponseJSON {
+    Base64URLString clientDataJSON;
+    Base64URLString attestationObject;
+    sequence<DOMString> transports;
+};
+
+dictionary AuthenticationResponseJSON {
+    Base64URLString id;
+    Base64URLString rawId;
+    AuthenticatorAssertionResponseJSON response;
+    DOMString? authenticatorAttachment;
+    AuthenticationExtensionsClientOutputsJSON clientExtensionResults;
+    DOMString type;
+};
+
+dictionary AuthenticatorAssertionResponseJSON {
+    Base64URLString clientDataJSON;
+    Base64URLString authenticatorData;
+    Base64URLString signature;
+    Base64URLString? userHandle;
+};
+
+dictionary AuthenticationExtensionsClientOutputsJSON {
 };
```

#### parseCreationOptionsFromJSON

```
+partial interface PublicKeyCredential {
+    static PublicKeyCredentialCreationOptions parseCreationOptionsFromJSON(PublicKeyCredentialCreationOptionsJSON options);
+};
```

また、PublicKeyCredentialには、staticメソッドとして、parseCreationOptionsFromJSONが追加されています。

サーバーサイドでPublicKeyCredentialCreationOptionsを生成し、JSONとしてクライアントに渡した値をnavigator.credentials.create()メソッドに渡す実装をすることがありますが、その際JSONからデシリアライズするのに役立つメソッドですね。

PublicKeyCredentialCreationOptionsJSONの定義は以下のように追加されています。やはり、PublicKeyCredentialCreationOptionsのArrayBufferの箇所をBase64URLStringに差し替えた構造になっています。

```
+dictionary PublicKeyCredentialCreationOptionsJSON {
+    required PublicKeyCredentialRpEntity                    rp;
+    required PublicKeyCredentialUserEntityJSON              user;
+    required Base64URLString                                challenge;
+    required sequence<PublicKeyCredentialParameters>        pubKeyCredParams;
+    unsigned long                                           timeout;
+    sequence<PublicKeyCredentialDescriptorJSON>             excludeCredentials = [];
+    AuthenticatorSelectionCriteria                          authenticatorSelection;
+    DOMString                                               attestation = "none";
+    AuthenticationExtensionsClientInputsJSON                extensions;
+};
+
+dictionary PublicKeyCredentialUserEntityJSON {
+    required Base64URLString        id;
+    required DOMString              name;
+    required DOMString              displayName;
+};
+
+dictionary PublicKeyCredentialDescriptorJSON {
+    required Base64URLString        id;
+    required DOMString              type;
+    sequence<DOMString>             transports;
+};
+
+dictionary AuthenticationExtensionsClientInputsJSON {
+};
```

#### parseRequestOptionsFromJSON

```
+partial interface PublicKeyCredential {
+    static PublicKeyCredentialRequestOptions parseRequestOptionsFromJSON(PublicKeyCredentialRequestOptionsJSON options);
+};
```

parseCreationOptionsFromJSONがあれば、当然parseCreationOptionsFromJSONも追加されています。JSONをデシリアライズし、PublicKeyCredentialRequestOptionsを返すメソッドです。

合わせて追加されたPublicKeyCredentialRequestOptionsJSONの定義は以下の通りです。

```
+dictionary PublicKeyCredentialRequestOptionsJSON {
+    required Base64URLString                                challenge;
+    unsigned long                                           timeout;
+    DOMString                                               rpId;
+    sequence<PublicKeyCredentialDescriptorJSON>             allowCredentials = [];
+    DOMString                                               userVerification = "preferred";
+    AuthenticationExtensionsClientInputsJSON                extensions;
+};
```

### AuthenticatorAssertionResponse

```
 [SecureContext, Exposed=Window]
 interface AuthenticatorAssertionResponse : AuthenticatorResponse {
     [SameObject] readonly attribute ArrayBuffer      authenticatorData;
     [SameObject] readonly attribute ArrayBuffer      signature; 
     [SameObject] readonly attribute ArrayBuffer?     userHandle;
+    [SameObject] readonly attribute ArrayBuffer?     attestationObject;
 };
```

AuthenticatorAssertionResponseには、attestationObjectフィールドが追加されています。attestationObjectといえば、これまでは登録時のAuthenticatorAttestationResponseに含まれるもので、認証時のAuthenticatorAssertionResponseには含まれないものでしたが、オプションとして含まれるようになりました。Device PublicKey 拡張のPull-Requestで一緒に追加されたようです。

Pull-Request：[device public key extension by equalsJeffH · Pull Request #1663 · w3c/webauthn](https://github.com/w3c/webauthn/pull/1663)

### PublicKeyCredentialCreationOptions

```
 dictionary PublicKeyCredentialCreationOptions {
     required PublicKeyCredentialRpEntity         rp;
     required PublicKeyCredentialUserEntity       user;
 
     required BufferSource                             challenge;
     required sequence<PublicKeyCredentialParameters>  pubKeyCredParams;
 
     unsigned long                                timeout;
     sequence<PublicKeyCredentialDescriptor>      excludeCredentials = [];
     AuthenticatorSelectionCriteria               authenticatorSelection;
     DOMString                                    attestation = "none";
+    sequence<DOMString>                          attestationFormats = [];
     AuthenticationExtensionsClientInputs         extensions;
 };
```

PublicKeyCredentialCreationOptionsには、attestationFormatsが追加されました。Relying Partyは、望ましい順にAttestation Statementのフォーマットを指定することが出来るようになりました。ということは、CTAP側でも同様の指定が出来るようになったのでしょうか？イマイチ使いどころがよく分からない機能ですが、複数のAttestation Statement Formatに対応するAuthenticatorが今後現れるんでしょうか？色々謎な機能です。

これもDevice PublicKey 拡張のPull-Requestで追加されたようです。

Pull-Request：[device public key extension by equalsJeffH · Pull Request #1663 · w3c/webauthn](https://github.com/w3c/webauthn/pull/1663)

### PublicKeyCredentialRequestOptions

```
 dictionary PublicKeyCredentialRequestOptions {
     required BufferSource                challenge;
     unsigned long                        timeout;
     USVString                            rpId;
     sequence<PublicKeyCredentialDescriptor> allowCredentials = [];
     DOMString                            userVerification = "preferred";
+    DOMString                            attestation = "none";
+    sequence<DOMString>                  attestationFormats = [];
     AuthenticationExtensionsClientInputs extensions;
 };
```

PublicKeyCredentialRequestOptionsには、attestationとattestationFormatsが追加されました。認証時でもAttestationが取得できるようになったので、その関係で追加されたようです。

Pull-Request：[device public key extension by equalsJeffH · Pull Request #1663 · w3c/webauthn](https://github.com/w3c/webauthn/pull/1663)

### CollectedClientData

```
 dictionary CollectedClientData {
     required DOMString           type;
     required DOMString           challenge;
     required DOMString           origin;
     boolean                      crossOrigin;
-    TokenBinding                 tokenBinding;
 };
```

CollectedClientDataからは正式にtokenBindingが削除されました。TokenBinding、、てしまったの、勿体ないなぁと今でも感じてしまいます。

### AuthenticatorTransport

```
 enum AuthenticatorTransport {
     "usb",
     "nfc",
     "ble",
+    "hybrid",
     "internal"
 };
```

AuthenticatorTransportには、hybridという値が追加されています。これは何かというと、元々cable（cloud-assisted-BLE）という名前で推進されていた、スマートフォンをAuthenticatorにし、ログイン先デバイスとクラウド経由で接続しつつ、物理的な近接性はBLEでチェックし担保する方式が"hybrid“という名前にリネームの上、マージされたものです。

### Pseudo-random function拡張

Pseudo-random function拡張が追加され、関連するデータ構造が追加されています。この拡張を使用することで、クレデンシャルから派生した対称鍵を生成し、データの暗号化等に使用することが出来るようです。[詳しくはEditor's Draftの該当箇所をご覧ください。](https://w3c.github.io/webauthn/#prf-extension "https\://w3c.github.io/webauthn/#prf-extension")

```
+dictionary AuthenticationExtensionsPRFValues {
+    required ArrayBuffer first;
+    ArrayBuffer second;
+};
+
+dictionary AuthenticationExtensionsPRFInputs {
+    AuthenticationExtensionsPRFValues eval;
+    record<USVString, AuthenticationExtensionsPRFValues> evalByCredential;
+};
+
+partial dictionary AuthenticationExtensionsClientInputs {
+    AuthenticationExtensionsPRFInputs prf;
+};
+
+dictionary AuthenticationExtensionsPRFOutputs {
+    boolean enabled;
+    AuthenticationExtensionsPRFValues results;
+};
+
+partial dictionary AuthenticationExtensionsClientOutputs {
+    AuthenticationExtensionsPRFOutputs prf;
+};
```

### Device-bound public key拡張

Device-bound public key拡張が追加され、関連するデータ構造が追加されています。Passkey導入により、クレデンシャルが複数のデバイスにSyncされるようになりましたが、クレデンシャルとは別に、デバイス毎にデバイス識別の為に公開鍵を持たせることで、登録時のデバイスと、認証時のデバイスが同一であるか、等を確認出来るようにする拡張です。

```
+dictionary AuthenticationExtensionsDevicePublicKeyInputs {
+    DOMString attestation = "none";
+    sequence<DOMString> attestationFormats = [];
+};
+
+partial dictionary AuthenticationExtensionsClientInputs {
+    AuthenticationExtensionsDevicePublicKeyInputs devicePubKey;
+};
+
+dictionary AuthenticationExtensionsDevicePublicKeyOutputs {
+    ArrayBuffer authenticatorOutput;
+    ArrayBuffer signature;
+};
+
+partial dictionary AuthenticationExtensionsClientOutputs {
+    AuthenticationExtensionsDevicePublicKeyOutputs devicePubKey;
+};
```

Pull-Request：[device public key extension by equalsJeffH · Pull Request #1663 · w3c/webauthn](https://github.com/w3c/webauthn/pull/1663)



### まとめ

以上、Web IDLから分かるWebAuthnの最新の変化でしたが、如何でしたでしょうか。よくよく見てみると、WebAuthnに関するデータ構造でも、Web IDLに現れてこない部分、例えばBEフラグやBSフラグの追加などは抜け落ちてしまっており、完全とはいきませんが、大まかな追加機能については把握できたのではないでしょうか。2022年、WebAuthnでは、Passkey周りが最も大きな変化でしたが、それ以外にも色々継続的な改善が行われています。Passkeyも出てきたし、最近のWebAuthnについて調べなおそうか、という方の一助になったなら幸いです。