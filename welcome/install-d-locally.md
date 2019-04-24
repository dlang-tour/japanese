# D言語をローカルにインストール

D言語のリファレンスコンパイラはDMD (Digital Mars D)と呼ばれています。
[LDC](https://github.com/ldc-developers/ldc)
([LLVM](http://llvm.org)ベースのDコンパイラ)も使えます。
[GDC](https://gdcproject.org) ([GCC](https://gcc.gnu.org/)ベースのDコンパイラ)
もあります。
より詳細な情報が[WikiのCompilersページ](https://wiki.dlang.org/Compilers)にありますが、
D言語初心者で何をインストールすればいいか迷っているならば、DMDをインストールしましょう。

## ダウンロードとインストール

[D言語のダウンロードページ](https://dlang.org/download.html)
は様々なD言語の実装の概要を提示しており、
OS固有のビルド済みDMDパッケージがすぐにダウンロードし、インストールできます。

もうひとつのOS固有のパッケージとして、Posix系システム(Linux、FreeBSD、MacOS)で動作する
[インストールスクリプト](https://dlang.org/install.html)があり、
これは管理者権限なしに様々な実装(それらの複数のバージョンを含む)をローカルにインストールできます。

## エディタを設定する

D言語の美点は、定型的なコードが非常に少なく派手なIDEを使う必要がないというところです。
しかしながら、あなたはお気に入りのエディタで気持ちよくD言語を使うと良いでしょう。
少なくとも以下のエディタ用のD言語のプラグインがあります:

- [Atom](https://github.com/Pure-D/atomize-d)
- [Eclipse](http://ddt-ide.github.io)
- [Emacs](https://github.com/Emacs-D-Mode-Maintainers/Emacs-D-Mode)
- [IntelliJ](https://github.com/intellij-dlanguage/intellij-dlanguage)
- [Sublime Text](https://github.com/yazd/DKit)
- [Vim](https://wiki.dlang.org/D_in_Vim)
- [VS Code](https://marketplace.visualstudio.com/items/webfreak.code-d)
- [Visual Studio](http://rainers.github.io/visuald/visuald/StartPage.html)

D言語専用のIDEを試してみることもできます:

- [Coedit](https://github.com/BBasile/Coedit)
- [Dlang IDE](https://github.com/buggins/dlangide)

D Wikiでは利用可能な [エディタ](https://wiki.dlang.org/Editors) や [IDE](https://wiki.dlang.org/IDEs)について、更に詳しい概要を確認できます。

