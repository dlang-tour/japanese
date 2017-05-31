# DIETテンプレート

ウェブページを簡単に書くために、
vibe.dはHTMLページを書くための簡略化された構文の
[DIETテンプレート](https://vibed.org/templates/diet)
をサポートしています。
DIETは[Jade templates](http://jade-lang.com/)をもとにしています。

    doctype html
    html(lang="en")
      head
        // Dのコードが評価されます
        title #{pageTitle}
        // 属性
        script(type='text/javascript')
          if (foo) bar(1 + 5)
        // ID = body-id
        // style = the-style
        body#body-id.the-style
          h1 DIET template

構文はインデントを認識し、閉じタグを挿入する必要はありません。

すべてのDIETテンプレートはコンパイルされ、パフォーマンスを最大に高めるためメモリに保持されます。
DIETテンプレートではページがレンダリングされる時に評価されるDのコードを使うことができます。
単体の式は`#{ 1 + 1 }`のように囲まれ、テンプレート内のどこででも使うことができます。
Dのコードライン全体はその行に`-`接頭辞がつけられます:

    - foreach(title; titles)
      h1 #{title}

複雑な式はこのような方法で使うこともでき、
最終的なHTMLを出力するために使われる関数が定義されることもあります。

DIETテンプレートは**CTFE**を用いてコンパイルされ、
標準的なvibe.dプロジェクトの`view`フォルダになければなりません。
実際にDIETテンプレートをレンダリングするためには、URLハンドラの中で
`render`関数を使います:

    void foo(HTTPServerResponse res) {
        string pageTitle = "Hello";
        int test = 10;
        res.render!("my-template.dt", pageTitle, test);
    }

すべてのDIETテンプレートで利用できるDの変数は
`render`へのテンプレートパラメータとして渡されます。

## {SourceCode:disabled}

```d
doctype html
html
  head
    title D statement test
  body
    - import std.algorithm : min;
    p Four items ahead:
    - foreach( i; 0 .. 4 )
      - auto num = i+1;
      p Item
        b= num
    p Prints 8:
    p= min(10, 2*6, 8)
```
