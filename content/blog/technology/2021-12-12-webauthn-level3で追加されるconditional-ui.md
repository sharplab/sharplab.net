---
layout: blog
date: 2021-12-22T00:29:01.626Z
title: WebAuthn Level3で追加されるConditional UI
thumbnail: ""
---
[Digital Identity技術勉強会 #iddance Advent Calendar 2021](https://qiita.com/advent-calendar/2021/iddance) 22日目の記事です。今回は、WebAuthn Level3で追加に向けて仕様策定が進められている、Conditional UIについて紹介します。なお、iddance Advent Calendarの他に、他にも[Quarkus翻訳Advent Calendar](https://qiita.com/advent-calendar/2021/quarkusio)も書いてますので興味がある方はそちらもご覧ください。

さてWebAuthn、パスワード認証と組み合わせた第二認証要素としてのセキュリティキーの利用は段々広まってきましたが、WebAuthnのDiscoverable Credentialを用いた、ID、パスワードレス認証は、まだまだこれからの状況です。普及を阻害している要因の一つとして、既存のWebAuthnのAPIではスムーズなユーザー体験の実現が難しいことが指摘されています。

WebAuthnは、利用可能なクレデンシャルの存在有無をRelying Partyがサイレントに取得することは出来ないAPI設計となっています。`navigator.credentials.get`メソッドでWebAuthnのクレデンシャルの取得が試みられると、認証用のモーダルダイアログが表示され、ユーザーが承認した場合のみ、クレデンシャルが呼び出し元に返却されます。これは、フィンガープリンティングを防止し、ユーザーのプライバシーを担保するために必要な制約です。

ただ一方で、利用可能なクレデンシャルが存在する場合のみ、WebAuthnでの認証オプションをユーザーに提示したい（利用可能なクレデンシャルが存在しないのにユーザー操作に割り込む認証用モーダルダイアログの表示をしたくない）というニーズも強くあります。勿論、WebAuthnで認証したいユーザーに明示的にボタンをクリックさせるという方法もありますが、その場合、その操作を表現するボタンのラベル表記をどうするのかという別の問題があります。Touch IDだったり、Windows Helloだったり、セキュリティキーであったり、様々なフォームファクターをとるAuthenticatorを利用した認証をユーザーに対して何と表現するのか。FIDO認証や、WebAuthn認証といった、テクノロジ名を出した表現では一般ユーザーには理解されず、「よく分からない『WebAuthn認証』のボタンを押したら変なエラーダイアログが出た、怖っ」というユーザー体験に陥るのを避けられません。

利用可能なクレデンシャルが存在する場合だけ（＝WebAuthnを既に利用開始している場合だけ）WebAuthnによる認証というオプションをスムーズにユーザーに提示したい、でもユーザーのプライバシーは担保したい、という要求を実現する為に、WebAuthn Level3では、利用可能なクレデンシャル（のユーザー名）のリストをフォームのinput要素のオートフィルのリストとして表示するモードの追加が計画されており、これがConditional UIと呼ばれている機能です。

参考：[WebAuthn仕様へのPull Request（未マージ）](https://github.com/w3c/webauthn/pull/1576)

![Conditional UI](/img/conditional-ui.png)

Conditional UIのイメージ（[Explainer: WebAuthn Conditional UI](https://github.com/w3c/webauthn/wiki/Explainer:-WebAuthn-Conditional-UI)より）

APIとしては、WebAuthnでクレデンシャルの取得を要求する際の `navigator.credentials.get` メソッドを呼び出す際、以下のように`mediation`引数で`conditional`を指定します。

```
navigator.credentials.get({
  mediation: 'conditional',
  publicKey: { /*----*/  }
});
```

これをページロード時に呼び出しておくと、input要素にフォーカスをあてた時に、ユーザー名の入力補助リストの一部としてWebAuthnのDiscoverable Credentialが表示されるようになります。
ユーザーがDiscoverable Credentialを選択してはじめて`navigator.credentials.get`のPromiseが完了し、Relying Party側でユーザー名を含むAssertionが得られるので、プライバシー上の問題は生じません。
なお、従来のWebAuthn APIでは、呼び出し時にtimeoutパラメーターで、指定時間内に認証操作が完了しないとキャンセルされる挙動でしたが、Conditional UIモードの場合、ページロード時に呼び出しておき、ユーザー操作を長時間待つことが想定されますので、timeoutパラメータは無効（タイムアウトは行われない）となります。

WebAuthn、まだまだ普及しているとは言い難い仕様ですが、少しづつ使い勝手が改善されており、今後普及が進むと良いですね。