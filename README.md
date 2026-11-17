# zignos-recipes

ZIGNOS のレシピツリー(Gentoo の gentoo/gentoo リポジトリ相当)。

## パッケージ数(2026-11-17 時点)

| カテゴリ | パッケージ数 | 内容 |
|---|---|---|
| `core/` | **68** | ベースシステム + ビルドツール一式(base.list の大半) |
| `main/` | **27** | CLI 実用パッケージ(好みで追加インストール) |
| `desktop/` | **0** | GUI(未整備) |
| **合計** | **95** | うち **81 が `zignos-build/base.list`**(ベースシステムの唯一の定義)に含まれる |

`main/` の `sudo` のみ意図的に base.list 外(特権昇格のデフォルトは doas)。

### core/(66)

autoconf bash bc binutils bison busybox bzip2 ca-certificates coreutils dhcpcd
diffutils doas e2fsprogs elfutils file findutils flex gawk gcc gdbm gettext
glibc gmp go grep groff grub gzip inetutils initscripts kbd libcap libevent
chrony libevent libffi libpipeline libxcrypt linux linux-headers m4 make
man-db man-pages meson mpc mpdecimal mpfr ncurses ninja openrc openssh
openssl patch pcre2 perl pkgconf procps python3 readline sed shadow tar
texinfo util-linux xz zlib zpkg zstd

### main/(27)

curl expat git htop iproute2 less libmnl nano rsync sudo tmux vim wget which
dosfstools jq libpcap lsof ncdu pciutils socat strace sysstat tcpdump tree
unzip zip

### desktop/(0)

未整備。GUI 要素(X/wayland、フォント、デスクトップ環境)は検討段階。

## 構成

- `core/` — ベースシステム + ビルドツール一式(bootstrap の最終システムに加え、
  python3/perl/man 等の実用ツールも含む)。**base.list が「ベースシステムとは何か」の
  唯一の定義**(`zignos-build/base.list` 参照)
- `main/` — CLI 実用パッケージ。インストールは `zpkg install <pkg>`(レシピツリーは
  システムに `/usr/share/zignos/recipes` として同梱済み)
- `desktop/` — GUI(未整備)

## レシピ形式

各パッケージはディレクトリ型(KISS 式)。

| ファイル | 必須 | 内容 |
|---|---|---|
| `version` | 必須 | 版数(単一行) |
| `sources` | 必須* | ソース URL(1行1個、`#ファイル名` fragment でキャッシュ名指定可) |
| `checksums` | 必須* | sha256(sources と同数) |
| `build` | 必須 | ビルドスクリプト(恒久 shell、destroot 契約) |
| `depends` | 任意 | 実行時依存(**1行1パッケージ**)。`flag?pkg` の条件付き依存可 |
| `makedepends` | 任意 | ビルド時のみの依存(1行1パッケージ) |
| `revision` | 任意 | レシピ改訂番号(`-rN`)。**内容を変えたら上げる**(binpkg キーが変わり upgrade で配信される) |
| `slot` / `provides` / `conflicts` | 任意 | スロット / virtual 提供 / 競合(記録のみ、解決は将来) |
| `iuse` | 任意 | このレシピが理解する USE フラグ(タイポ検出に使われる) |
| `post_install` / `post_remove` | 任意 | インストール/削除直後に実行されるフック |
| 資産ファイル | 任意 | `qemu-x86_64.config`・`doas.conf`・パッチなど(build から `$ZPKG_RECIPE_DIR` で参照) |

詳細は zignos リポジトリの [docs/design-pkg.md](https://github.com/studio-nosa-tech/zignos/blob/main/docs/design-pkg.md)。

## 現在の状態

- ツールチェーン(LFS 13.1-systemd 由来: GCC 16.2.0 / Glibc 2.44 / Binutils 2.47 /
  Bash 5.3 / Coreutils 9.11 / Go 1.27.4)は **全 sha256 を実ダウンロードして検証済み**
  (2026-11-15)。以降に追加したパッケージも同様に実ダウンロードで sha256 を確定している
- `build` スクリプトは LFS の手順を zpkg の destroot 契約(`$ZPKG_DESTROOT` に成果物を
  置く)に翻訳したもの。**テストスイートは一律スキップ**
- OpenSSL 4.0 互換パッチを `python3` / `wget` がレシピ同梱している(OpenSSL 4 で削除された
  fixed-version TLS method のガード。`makedepends: patch` で自家パッチを当てる)
- LFS からの意図的な差分: coreutils の i18n パッチと autoconf/automake チェーン省略、
  tar の acl パッチ省略、bzip2 の docs パッチ省略、systemd 系なし(init は OpenRC)。
  詳細は zignos-build/bootstrap.sh のヘッダーコメント
- cross ビルドの一時ツール群(LFS ch5/ch6/ch7)はレシピ化せず zignos-build の
  `stage0.sh` / `stage1.sh` が担当する。zpkg は最終システムにのみ使う
