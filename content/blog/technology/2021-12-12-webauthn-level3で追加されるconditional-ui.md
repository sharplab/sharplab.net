---
layout: blog
date: 2021-12-22T00:29:01.626Z
title: WebAuthn Level3で追加されるConditional UI
---
[Digital Identity技術勉強会 #iddance Advent Calendar 2021](https://qiita.com/advent-calendar/2021/iddance) 22日目の記事です。今回は、WebAuthn Level3で追加に向けて仕様策定が進められている、Conditional UIについて紹介します。なお、iddance Advent Calendarの他に、他にも[Quarkus翻訳Advent Calendar](https://qiita.com/advent-calendar/2021/quarkusio)も書いてますので興味がある方はそちらもご覧ください。

さてWebAuthn、パスワード認証と組み合わせた第二認証要素としてのセキュリティキーの利用は段々広まってきましたが、WebAuthnのDiscoverable Credentialを用いた、ID、パスワードレス認証は、まだまだこれからの状況です。普及を阻害している要因の一つとして、既存のWebAuthnのAPIではスムーズなユーザー体験の実現が難しいことが指摘されています。

WebAuthnは、サイレントに利用可能なクレデンシャルの一覧をRelying Partyが取得することは出来ないAPI設計となっています。これは、フィンガープリンティングを防止し、ユーザーのプライバシーを担保するために必要な制約ですが、一方で、利用可能なクレデンシャルが存在するかどうかも分からないのに、ユーザー操作に割り込む認証用モーダルダイアログの表示をRelying Partyに強いることになり、Relying Partyのユーザー体験は大きく損なわれています。

利用可能なクレデンシャルが存在する場合だけWebAuthnの利用をユーザーに提示したい、でもユーザーのプライバシーは担保したい、という要求を実現する為に、WebAuthn Level3では、モーダルダイアログの代わりに、ユーザー名（＝利用可能なクレデンシャル）の一覧をフォームのinput要素のオートフィルのリストとして表示するモードの追加が計画されており、これがConditional UIと呼ばれている機能です。

APIとしては、WebAuthnでクレデンシャルの取得を要求する際の `navigator.credentials.get` メソッドを呼び出す際、以下のように`mediation`引数で`conditional`を指定します。

```
navigator.credentials.get({
  mediation: 'conditional',
  publicKey: { /*----*/  }
});
```

これをページロード時に呼び出しておくと、input要素にフォーカスをあてた時に、ユーザー名の入力補助リストの一部としてWebAuthnのDiscoverable Credentialが表示されるようになります。
ユーザーがDiscoverable Credentialを選択してはじめてnavigator.credentials.getのPromiseが完了し、Relying Party側でユーザー名を含むAssertionが得られるので、プライバシー上の問題は生じません。
なお、従来のWebAuthn APIでは、呼び出し時にtimeoutパラメーターで、指定時間内に認証操作が完了しないとキャンセルされる挙動でしたが、Conditional UIモードの場合、ページロード時に呼び出しておき、ユーザー操作を長時間待つことが想定されますので、timeoutパラメータは無効（タイムアウトは行われない）です。