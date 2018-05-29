# 一番最初のCVSから取ってきた
```sh
$ mkdir stone
$ cd stone
$ git cvsimport -v -d :pserver:anonymous@cvs.sourceforge.jp:/cvsroot/stone stone
```
# CVSのupdateを取ってくる
```sh
$ git cvsimport -v -d :pserver:anonymous@cvs.osdn.net:/cvsroot/stone stone
```
master,sengoku,orignブランチが更新される?


# Linuxビルド
linux
```sh
$ make linux-ssl
```

# Cygwin
## ビルド
```sh
$ make TARGET=stone ssl_stone
```

cygwinならば、TLS1.2が使える。 Windowsでは、
```c
#ifdef WINDOWS
#define OPENSSL_NO_TLS1_1
#define OPENSSL_NO_TLS1_2
```
されていて使えない。Windowsは、OpenSSL 1.0以前の古いSSL使っているから??

## 課題
しかし、SSH-SSL wrapしていると切断後、以下のログで溢れる
```
Aug 17 15:41:47.573582 5 TCP 10: recv MSG_OOB ret=0, err=119
Aug 17 15:41:47.573582 5 TCP 10, 11: doReadWrite Can't happen spin occured tx/rx: 6522/4560, 4699/6561
```


# Windowsでのstone 2.4 (TLSv1.2対応)
cygwinでもTLSv1.2対応ができるけど、Windowsのサービスにならないので
Vitual Studioの無料版を入れてコンパイルする。
まずは、前提の、OpenSSL, PCREを入手してインストールする。
Visutal Studioでは、どのパッケージを入れれば良いのか不明で cl, nmakeまでは入ったけど
wmc, wrcが見つからず、Mailefileをmc,rcに書き換えてコンパイルした。

参考: http://www.gcd.org/blog/2011/06/806/#postcomment
```
CVS レポジトリの最新版 stone.c は TLS1.2 に対応しております。

http://ja.osdn.net/cvs/view/stone/stone/stone.c?view=log

VisualC++ の場合は、

nmake win-svc

などと実行すればビルドできます。この場合、regex.h ではなく pcreposix.h
が参照されます。

PCRE は http://www.pcre.org/ からダウンロードできます。
```

## TLSv1.2対応したWindows版stone 2.4コンパイル
### コンパイル環境
* Visual Studio Community 2017

学生、オープン ソース、個人の開発者向けの無料版を
[Visual Studio のダウンロード](https://www.visualstudio.com/ja/downloads/)
からダウンロード。

Visual Studio Installerで「C++によるデスクトップ開発」と、mc,rcが必要なのでオプションの
* Windows XP Suppor for C++

を追加。

### 必要なライブラリ
* OpenSSL 1.0
* PCRE

が必要。
* 以下にコンパイル方法を示す
* OpenSSL 1.1にはstoneのソースがまだ対応していない。

#### OpenSSL 1.0
[OpenSSL 1.0.2](https://github.com/openssl/openssl/tree/OpenSSL_1_0_2-stable)のソースをコンパイルする。

##### 前提
[ActivePerl](http://www.activestate.com/ActivePerl)を入手しインストールする。

##### ソースを取得する。
1. Visual Studioで、「チーム(M)」→「接続の管理(N)…」で、チームエクスプローラを開く
2. 「ローカルGitリポジトリ」の「複製」メニューを選択して、複製するGitリポジトリのURに
    https://github.com/openssl/openssl.git
を入力しリポジトリを複製する。
3. 作成されたopensslを右クリックで「開く」
4. 「ブランチ」で、remote/originの「OpenSSL_1_0_2-stable」をチェックアウト
 
##### コンパイルとインストール
1. スタートメニューの「Visual Studio 2017」にある「VS2017用x86 NativeToolsコマンドプロンプト」を起動
2. コマンドプロンプトで、

``` cmd
C:/Users/.../source> cd openssl
C:/Users/.../openssl> perl Configure VC-WIN32 no-asm
```
msディレクトリ以下にファイルが準備される?
```cmd
C:/Users/.../openssl> ms\do_ms
C:/Users/.../openssl> nmake -f ms\ntdll.mak
C:/Users/.../openssl> nmake -f ms\ntdll.mak test
C:/Users/.../openssl> nmake -f ms\ntdll.mak install
```
C:\usr\local\ssl にインストールされる。

※OpenSSL 1.1とはコンパイル方法とインストール先が違うので注意。

#### PCRE
ソースからコンパイルしようとしたけど、OfficialなGitレポジトリもないし、うまくいかなかったので、
[GnuWin32](http://gnuwin32.sourceforge.net/) projectのコンパイル済みをインストールした。
* [Developer Files ZIP](http://gnuwin32.sourceforge.net/downlinks/pcre-lib-zip.php)
* [Binary Files ZIP](http://gnuwin32.sourceforge.net/downlinks/pcre-bin-zip.php)
を入手して、
* pcre-7.0-lib.zipからは、includeとlib
* pcre-7.0-bin.zipからは、bin
フォルダーの内容を、C:\usr\local\ssl以下にCopyする。

// /usr/local/pcreを作っても良かったけどバスが増えると面度なのでopenssl以下に入れた。

### Windows版stoneコンパイル

#### ソースを取得する。
1. Visual Studioで、「チーム(M)」→「接続の管理(N)…」で、チームエクスプローラを開く
2. 「ローカルGitリポジトリ」の「複製」メニューを選択して、複製するGitリポジトリのURに
3. GitリポジトリURLを入力しリポジトリを複製する

#### コンパイル
1. スタートメニューの「Visual Studio 2017」にある「VS2017用x86 NativeToolsコマンドプロンプト」を起動
2. コマンドプロンプトで、

``` cmd
C:/Users/.../source> cd stone
C:/Users/.../stone> nmake win-svc
```

#### インストール
コンパイルされた stone.exe と、C:/usr/local/binにある
* libeasy32.dll
* pcre3.dll
* pcreposix3.dll
* ssleary32.dll
をインストール先におけば、stone.exeが実行できるようになる。
