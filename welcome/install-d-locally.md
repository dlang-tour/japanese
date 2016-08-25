# D言語をローカルにインストール

D言語のウェブサイト [dlang.org](https://dlang.org) 
から、リファレンスコンパイラ **DMD** (Digital Mars D)
の最新バージョンを [ダウンロード](http://dlang.org/download.html) してインストールできます:

### Windows

* [インストーラ](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd-{{latest-release}}.exe)
* または: [アーカイブ](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.windows.7z)
* [chocolatey](https://chocolatey.org/packages/dmd)を使って: `choco install dmd`

### macOS

* `.dmg` [パッケージ](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.dmg)
* または: [アーカイブ](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.osx.tar.xz)
* [Homebrew](http://brew.sh)を使って: `brew install dmd`

### Linux / FreeBSD / macOS

ユーザディレクトリにDMDを素早くインストールするなら、, 以下を実行してください: `curl -fsS https://dlang.org/install.sh | bash -s dmd`

様々なディストリビューションに向けてパッケージが提供されています:

* [ArchLinux](https://wiki.archlinux.org/index.php/D_(programming_language))
* [Debian/Ubuntu](http://d-apt.sourceforge.net).
* [Fedora/CentOS](http://dlang.org/download.html#dmd)
* [Gentoo](https://wiki.gentoo.org/wiki/Dlang)
* [OpenSuse](http://dlang.org/download.html#dmd)

### その他のコンパイラ

それ自身をバックエンドとするDMDリファレンスコンパイラの他に、
[dlang.org](https://dlang.org) のダウンロードセクションで２種類のコンパイラが入手できます:

* GCCをバックエンドとする[**GDC**](http://gdcproject.org/downloads)
* LLVMバックエンドを元にした[**LDC**](https://github.com/ldc-developers/ldc#installation)

GDCとLDCは常に最新のDMDフロントエンドバージョン、とはなりませんが、
ARMのようなその他のプラットフォームのサポートだけでなく、より良い最適化を受けられます。

[さらなる情報は](https://wiki.dlang.org/Compilers) wikiを参照してください。

