# enterキー押下時に次項目へフォーカス移動するjavascript(jQuery)

[テストページ](https://murasuke.github.io/focus-next-jquery/public/index.html)

## 目的

似たようなスクリプトは探せば結構あるのですが、問題が多いので1から作りました

* enterキー押下時に次項目へ移動するjavascript(jQuery利用)
* フォーカス移動概要 
  * tabindex順に移動します(tabindex1以上⇒tabindex未指定の順)
  * tabindexがマイナスの項目へは移動しません
  * tabindexがマイナスの項目から移動する場合は、直近前後の項目へ移動します。
  * 通常移動しない項目<div>や<span>にtabindexを付けると移動可能になることを考慮
  * 上記に関連して、フォーカス移動可能な項目が入れ子になっていても移動できる

## 使い方
```html
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script>    
      $(() => {
        const elements = ':focusable:not(a)';
        $(elements).keypress((e) => {
          if (e.key === 'Enter') {
            // submitしない
            e.preventDefault();
            //　focus可能な項目が入れ子になっている場合、targetのみで処理する
            e.stopPropagation();

            // tabindex順に移動するためソート
            let sortedList = $(elements).sort((a,b) => {
              if(a.tabIndex && b.tabIndex) {
                return a.tabIndex - b.tabIndex; 
              } else if(a.tabIndex && !b.tabIndex) {
                return -1;
              } else if(!a.tabIndex && b.tabIndex) {
                return 1;
              }
              return 0;
            });
            
            if (e.target.tabIndex < 0) {
              // tabindexがマイナスの場合、DOM上で次の項目へ移動するためソート前の項目から検索する
              sortedList = elements;
            }

            const index = $(sortedList).index(e.target);
            const nextFilter = e.shiftKey ? `:lt(${index}):last` : `:gt(${index}):first`;            
          　const nextTarget = $(sortedList).filter(nextFilter);

            // shift + enterでtagindexがマイナスの項目へ移動するのを防ぐ
            if (!nextTarget.length || nextTarget[0].tabIndex < 0) return;

            // フォーカス移動＋文字列選択
            nextTarget.focus();
            if (typeof nextTarget.select === 'function') nextTarget.select();
          }
        });
      });
    </script>
```

## フォーカス移動の仕様(手で動かしながら調べたので違っているかも)
1. input, button, select, textarea, aがフォーカス移動可能なelement
1. 上記以外でも、tabindex属性を与えると移動対象となる
1. disabled、見えない項目は移動対象外
1. tabindex(1以上)の順に移動 ⇒ 最大まで行ったらtabindex未指定(or 0,空白)をDOM出現順に移動 ⇒ URL入力欄に移動
1. 同じtabindexが複数ある場合はDOM出現順に移動
1. `tabindex=0`や`tabindex=""`は未指定と同じ扱い
1. tabindexがマイナスの項目には移動しない。
1. tabindexがマイナスの項目にフォーカスがある場合、次項目はDOM出現順に移動可能な項目へ移動
1. Shiftを押下時は逆順に移動
1. ラジオボタンは同じnameを持つ場合、グループ化される。  ・・・【制御がややこしいため、未サポート】
  * グループがチェックを持つ場合は、そこへフォーカスする。
  * チェックがない場合は、グループの先頭にフォーカスする。
  * 同一グループ内で異なるtabindexを持つ場合
    * チェックがなければ、それぞれのtabindex毎にグループ化されるような動き
    * チェックがあれば、チェックがある箇所のみが移動対象となる


## その他
* ~~anchorも移動対象~~ 移動しても、Enterでリンク先に移動してしまうので対象からしています。

## 制限事項

* タブ移動時、ラジオボタンは同一nameでグループ化されるのがTabでの動作ですが、そこまで細かい制御はできていません。

## 参考資料
https://html.spec.whatwg.org/multipage/interaction.html#sequential-focus-navigation



## jqueryを使った場合の移動について

セレクタで「:focusable」「:tabbable」があるが、tabindexの順番まで考慮して並べ替えはしてくれない模様。
  ⇒ 画面の項目順に移動する前提であれば使える。

[api-focusable-selector](http://www.w3big.com/ja/jqueryui/api-focusable-selector.html)

[api-tabbable-selector](http://www.w3big.com/ja/jqueryui/api-tabbable-selector.html)