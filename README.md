# zignos-recipes

ZIGNOS のレシピツリー(Gentoo の gentoo/gentoo リポジトリ相当)。

## チャンネル構成

レシピツリーは3つのリリースチャンネルを持つ。各チャンネルは同じ
`core/` / `main/` / `desktop/` 構造を持つ。

```
zignos-recipes/
├── stable/{core,main,desktop}     # 安定版。既定。stage3/イメージに同梱
├── testing/{core,main,desktop}    # 次期 stable の検証
└── unstable/{core,main,desktop}   # 開発中の最新版
```

| チャンネル | パッケージ数 | 内容 |
|---|---|---|
| `stable/` | **162** | 安定版。`zpkg` の既定 |
| `testing/` | **162** | stable 候補。まずここで検証する |
| `unstable/` | **162** | 開発中。破壊的変更を含み得る |
| **合計** | **486** | 初期状態では3チャンネルとも同一内容 |

`core/` と `main/` はチャンネル間で同じ内容を基本とする。KDE デスクトップを
導入した後は **`desktop/` だけ系列が分岐**する: stable / testing は KDE Plasma
LTS、unstable は最新版を追従する。レシピ数もチャンネルで異なり得る。

各チャンネル内の内訳:

| カテゴリ | パッケージ数 | 内容 |
|---|---|---|
| `core/` | **76** | ベースシステム + ビルドツール一式(base.list の大半) |
| `main/` | **86** | CLI・システム管理パッケージ + firmware(好みで追加インストール) |
| `desktop/` | **0** | GUI(設計中: KDE Plasma 6 / Wayland) |
| **合計** | **162** | うち **37 が `zignos-build/base.list` のトップレベル**(依存込みのベースは 64) |

`base.list` はトップレベルだけを列挙する(Gentoo の `@system` 相当の最小構成)。
依存と makedepends は zpkg が解決して自動で入れる。sudo / doas / SSH / エディタは
すべて任意インストール。

### core/(76)

autoconf bash bc binutils bison bubblewrap busybox bzip2 ca-certificates chrony cmake coreutils dbus
dhcpcd diffutils doas e2fsprogs eudev elfutils file findutils flex gawk gcc gdbm gettext gperf
glibc gmp go grep groff grub gzip inetutils initscripts initramfs kbd kmod libcap
libevent libffi libpipeline libxcrypt linux linux-headers m4 make
man-db man-pages meson mpc mpdecimal mpfr ncurses ninja openrc openssh
openssl patch pcre2 perl pkgconf procps python3 readline sed shadow tar
texinfo tzdata util-linux xz zlib zpkg zstd

### main/(86)

curl expat fastfetch git htop iproute2 less libmnl nano rsync sudo tmux vim wget which
dmidecode dosfstools efibootmgr efivar iw jq libnl libpcap lshw linux-firmware lsof ncdu pciutils
smartmontools socat strace sysstat tcpdump tree unzip wpa_supplicant zip

# CLI tools added in M30 (network / build / archive / monitoring)
ethtool iputils iperf mtr nmap whois
automake ccache fzf libtool lua sqlite
7zip brotli entr lz4 nnn pigz pv
age btop moreutils restic wireguard-tools xxhash zsh

# System tools added in M31 (signing / cron / firewall / debug)
cronie cpio fuse3 gdb glib gnupg
libassuan libgcrypt libgpg-error libksba libnftnl lm-sensors
logrotate nftables npth pinentry popt screen sshfs

### desktop/(0)

設計中。最初のデスクトップ環境として **KDE Plasma 6 (Wayland)** を導入する。
デスクトップ本体と KDE アプリは **`plasma-meta`** と **`kde-applications`** に
分ける(`plasma-meta` はアプリを含めず、`kde-applications` は Plasma 本体に
依存しない)。

チャンネル方針:

- `stable/desktop` — KDE Plasma 6.6 LTS / Qt 6.8 LTS / KF6 6.18 / Gear 25.04.3
- `testing/desktop` — stable と同じ LTS 系列。LTS ポイントリリースを先に検証
- `unstable/desktop` — Plasma 6.8 / Qt 6.11 / KF6 6.32 / Gear 26.08 以降の最新版

設計メモは `zignos` リポジトリの `docs/desktop-kde.md`、公開サマリは
サイトの `/docs/desktop/`。実装は M43 以降。

## 構成と運用

- `stable/` — 既定。stage3 / イメージにはこのサブツリーだけが同梱される
- `testing/` — stable 候補。まずここでビルド・起動を確認する
- `unstable/` — 開発中。破壊的変更を含み得る
- 選択は zpkg 側の `/etc/zpkg/make.conf` の `CHANNEL=`(既定 `stable`)。
  `zpkg channel testing` で切り替え、`zpkg upgrade -p` で差分を確認する
- 変更は **unstable → testing → stable** の順に昇格させる。各チャンネルで
  version / revision は独立して上げる。**同じ version+revision のまま内容を
  変えると binpkg キャッシュが衝突する**ため、内容を変えたら必ず revision を
  上げる
- `base.list` が「ベースシステムとは何か」の唯一の定義
  (`zignos-build/base.list` 参照)。チャンネルには依存しない
- インストール済みシステムでは `/usr/share/zignos/recipes/<channel>/` として
  使う。`zpkg` は `ZPKG_REPOS` 未設定時に選択チャンネルの `core` / `main` /
  `desktop` を自動的に探す

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
| `build_network` | 任意 | `1` / `0` でビルド時ネットワークをグローバル設定より優先 |
| `upgrade_warning` | 任意 | `zpkg upgrade` が実行前に表示する注意メッセージ(複数行可) |
| `revision` | 任意 | レシピ改訂番号(`-rN`)。**内容を変えたら上げる**(binpkg キーが変わり upgrade で配信される) |
| `slot` / `provides` / `conflicts` | 任意 | スロット / virtual 提供 / 競合(依存解決で使用) |
| `iuse` | 任意 | このレシピが理解する USE フラグ(タイポ検出に使われる) |
| `post_install` / `post_remove` | 任意 | インストール/削除直後に実行されるフック |
| 資産ファイル | 任意 | `x86_64-generic.config`・`logo.txt`・`config.jsonc`・`sudoers`・パッチなど(build から `$ZPKG_RECIPE_DIR` で参照) |

詳細は zignos リポジトリの [docs/design-pkg.md](https://github.com/studio-nosa-tech/zignos/blob/main/docs/design-pkg.md)。

## 署名

配布用のレシピツリーは `zpkg sign-recipes` で署名する。

```sh
zpkg sign-recipes --key ~/zignos-signing.key \
  --generation 42 --expires 90d .
```

`recipes.manifest`(全 regular file の path / sha256 / size / mode)と
`recipes.manifest.sig` が生成される。この 2 ファイルはリリース時に生成する
もので、リポジトリにはコミットしない(`.gitignore` 済み)。`zpkg sync` は
manifest の署名・全ファイル・余計なファイルを検証してからツリーを置換する。
公式署名鍵の運用開始までは、未署名ツリーに `--allow-unsigned` が必要。

## 再現性

zpkg は build 時に `SOURCE_DATE_EPOCH`(既定 `946684800` = 2000-01-01)と
`KBUILD_BUILD_TIMESTAMP` / `KBUILD_BUILD_USER` / `KBUILD_BUILD_HOST` /
`PYTHONHASHSEED=0` を渡し、レシピを `umask 022` で実行する。binpkg の
tar/gzip の mtime は固定される。

レシピ側でも、時刻やホスト名を埋め込むツールには次のように明示する。

- Go: `go build -trimpath -buildvcs=false`(`stable/core/zpkg`。各チャンネル共通)
- initramfs: `find | sort`、mtime を `SOURCE_DATE_EPOCH` に統一、`gzip -n`
  (`stable/core/initramfs/zignos-mkinitramfs`。各チャンネル共通)
- 生成物に日付を書くスクリプトは `SOURCE_DATE_EPOCH` を使う

```sh
zpkg build --repro-check <pkg>   # 2 回ビルドして manifest を比較
```

詳細は zignos リポジトリの
[docs/reproducibility.md](https://github.com/studio-nosa-tech/zignos/blob/main/docs/reproducibility.md)。

## 現在の状態

- ツールチェーン(LFS 13.1-systemd 由来: GCC 16.2.0 / Glibc 2.44 / Binutils 2.47 /
  Bash 5.3 / Coreutils 9.11 / Go 1.27.4)は **全 sha256 を実ダウンロードして検証済み**
  (2026-11-15)。以降に追加したパッケージも同様に実ダウンロードで sha256 を確定している
- `build` スクリプトは LFS の手順を zpkg の destroot 契約(`$ZPKG_DESTROOT` に成果物を
  置く)に翻訳したもの。**テストスイートは一律スキップ**
- OpenSSL 4.0 互換パッチを `python3` / `wget` がレシピ同梱している(OpenSSL 4 で削除された
  fixed-version TLS method のガード。`makedepends: patch` で自家パッチを当てる)
- **カーネルは `x86_64_defconfig` + `stable/core/linux/x86_64-generic.config`**。起動系
  (ACPI/APIC、SATA AHCI、NVMe、USB、VT/VGA/simpledrm、ext4、virtio、主要 NIC)は組み込み、
  GPU/WiFi/追加 NIC はモジュール。`make modules_install` + `kmod depmod` までレシピが行う
- **initramfs は busybox ベース**(`stable/core/initramfs`)。`busybox` は initramfs 用に
  **静的リンク**(`CONFIG_STATIC=y`)、`zignos-mkinitramfs` が `/boot/initramfs.img` を生成し、
  `/init` が `root=UUID=` を解決して `switch_root` する
- **`stable/core/kmod 34.3`** が modprobe/depmod を提供。eudev が uevent を受けて
  libkmod 経由で起動時・ホットプラグ時のモジュールを自動ロードする
- **`stable/main/linux-firmware`(20261019)は base.list 外**: tarball 約 1.0 GiB、導入 約 2.0 GiB
  (7793 ファイル)で 12G イメージの空き(~3.9 GiB)に対して大きすぎる。実機では
  `zpkg install linux-firmware` で導入する(kernel 側は該当ドライバ + `CONFIG_FW_LOADER`
  が必要。`make dedup` は rdfind 未導入のため未使用)
- LFS からの意図的な差分: coreutils の i18n パッチと **bootstrap 時の**
  autoconf/automake チェーン省略、tar の acl パッチ省略、bzip2 の docs パッチ省略、
  systemd 系なし(init は OpenRC)。autoconf(core)/ automake・libtool(main)は
  最終システム用のレシピとして用意してある(M14/M30)。詳細は zignos-build/bootstrap.sh の
  ヘッダーコメント
- cross ビルドの一時ツール群(LFS ch5/ch6/ch7)はレシピ化せず zignos-build の
  `stage0.sh` / `stage1.sh` が担当する。zpkg は最終システムにのみ使う
