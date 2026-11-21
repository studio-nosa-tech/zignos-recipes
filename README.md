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
| `stable/` | **382** | 安定版。`zpkg` の既定 |
| `testing/` | **389** | stable 候補。まずここで検証する |
| `unstable/` | **389** | 開発中。破壊的変更を含み得る |
| **合計** | **1158** | stable は LTS 系列、testing/unstable は新しい依存を含む |

`core/` と `main/` はチャンネル間で同じ内容を基本とする。ただし **`core/linux`
はチャンネルごとに系列が分岐**する: stable は Linux LTS、testing は最新安定版、
unstable は mainline / RC。**`desktop/` も分岐**し、stable は LTS/旧安定系列、
testing は最新安定版、unstable は RC / beta / git snapshot を含む挑戦的な構成になる。

変更は必ず **Unstable → Testing → Stable** の順に検証する。Stable は LTS
ブランチのため、Testing の最新版をそのまま入れるのではなく、検証済みの
変更を LTS へバックポートする。逆流はさせない。

各チャンネル内の内訳:

| カテゴリ | stable | testing / unstable | 内容 |
|---|---|---|---|
| `core/` | **77** | **77** | ベースシステム + ビルドツール一式(base.list の大半) |
| `main/` | **100** | **102** | CLI・システム管理パッケージ + firmware(好みで追加インストール) |
| `desktop/` | **205** | **210** | GUI(KDE Plasma 6 / KF6 / Qt6 / Xwayland+GLX / Mesa / PipeWire / plasma-nm / powerdevil / UDisks2 / KDE アプリ) |
| **合計** | **382** | **389** | うち **37 が `zignos-build/base.list` のトップレベル** |

`base.list` はトップレベルだけを列挙する(Gentoo の `@system` 相当の最小構成)。
依存と makedepends は zpkg が解決して自動で入れる。sudo / doas / SSH / エディタは
すべて任意インストール。

### core/(77)

autoconf bash bc binutils bison bubblewrap busybox bzip2 ca-certificates chrony cmake coreutils dbus
dhcpcd diffutils doas e2fsprogs eudev elfutils file findutils flex gawk gcc gdbm gettext gperf
glibc gmp go grep groff grub gzip inetutils initscripts initramfs kbd kmod libcap
libevent libffi libpipeline libxcrypt linux linux-headers linux-pam m4 make
man-db man-pages meson mpc mpdecimal mpfr ncurses ninja openrc openssh
openssl patch pcre2 perl pkgconf procps python3 readline sed shadow tar
texinfo tzdata util-linux xz zlib zpkg zstd

### main/(98)

curl expat fastfetch git htop iproute2 less libmnl nano rsync sudo tmux vim wget which
dmidecode dosfstools efibootmgr efivar iw jq libnl libpcap lshw linux-firmware lsof ncdu pciutils
smartmontools socat strace sysstat tcpdump tree unzip wpa_supplicant zip

networkmanager networkmanager-wifi
# CLI tools added in M30 (network / build / archive / monitoring)
ethtool iputils iperf mtr nmap whois
automake ccache fzf libtool lua sqlite
7zip brotli entr lz4 nnn pigz pv
age btop moreutils restic wireguard-tools xxhash zsh

# System tools added in M31 (signing / cron / firewall / debug)
cronie cpio fuse3 gdb glib gnupg
libassuan libgcrypt libgpg-error libksba libnftnl lm-sensors
logrotate nftables npth pinentry popt screen sshfs

# M46 storage / graphics libraries
cryptsetup exiv2 json-c keyutils libarchive libatasmart libblockdev
libbytesize libnvme lvm2 poppler

### desktop/(203)

M44 で KF6 + KWin + Plasma 最小、M45 で音声・ネットワーク・電源、M46 で
UDisks2 と KDE アプリを実装。
最初のデスクトップ環境として **KDE Plasma 6 (Wayland)** を導入する。
デスクトップ本体と KDE アプリは **`plasma-meta`** と **`kde-applications`** に
分ける(`plasma-meta` はアプリを含めず、`kde-applications` は Plasma 本体に
依存しない)。

チャンネル方針:

- `stable/desktop` — Linux 6.18.64 LTS / **Plasma 6.4.5** / Qt 6.8.5 LTS /
  KF6 6.18 / Gear 25.04.3 / Mesa 26.2.7
- `testing/desktop` — Linux 7.3.5 / Plasma 6.8.3 / Qt 6.11.3 / KF6 6.32 /
  Gear 26.08.3 / Mesa 26.3.1
- `unstable/desktop` — Linux 7.4-rc3、Gear 26.11.80 beta。
  Qt 6.12 / KF6 6.33 / Plasma 6.9 は Plasma 6.9 と同時に移行するため、
  現時点では Testing と同じ Qt 6.11.3 / KF6 6.32 / Plasma 6.8.3 / Mesa 26.3.1
  を使う(Mesa main は QEMU softpipe の KWin 検証が通るまで保留)

昇格順は **Unstable → Testing → Stable**。Stable は LTS へバックポートする。

実装済み(M43 基盤 + M44):

- `duktape 2.7.0` — polkit 128 の JavaScript エンジン
- `python-markupsafe 3.0.3` / `python-jinja2 3.1.6` — elogind の Meson ビルド用
- `elogind 257.16` — logind/seat マネージャ(KWin の DRM/seat に必須)
- `polkit 128` — 認可フレームワーク(polkitd アカウント付き)
- **Linux-PAM 1.7.3 は `core/` に移動**し、shadow を PAM 対応にした
  (`/etc/pam.d` の login/su/passwd 等も shadow が同梱)
- `wayland 1.25.0` / `wayland-protocols 1.49` / `libxkbcommon 1.13.2` /
  `xkeyboard-config 2.45` — Wayland クライアント基盤
- `libdrm 2.4.134` / `libevdev 1.13.4` / `mtdev 1.1.7` / `libinput 1.32.0` /
  `pixman 0.46.4` — DRM・入力・描画
- `libpng 1.6.59` / `libjpeg-turbo 3.2.1` / `freetype 2.15.0` /
  `harfbuzz 15.1.0` / `fontconfig 2.18.3` / `dejavu-fonts 2.37` — フォント
- `libglvnd 1.7.0` + `mesa 26.2.7` — EGL/GLES(virgl / softpipe、LLVM なし)。
  Mesa 26 の DRI megadriver (`dril`)を作るため `platforms=wayland,x11` でビルドし、
  `virtio_gpu` / `kms_swrast` / `swrast` を導入
- `qtbase 6.8.5` — QtCore/Gui/Widgets/DBus/Network。Plasma 向けに xcb を有効化し、
  EGL + OpenGL ES 2 でビルド。`qtshadertools` / `qtdeclarative` / `qtwayland` /
  `qtsvg` / `qttools` / `qt5compat` / `qtpositioning` / `qtlocation` /
  `qtsensors` も追加
- **ECM 6.18.0 + KF6 6.18.0** — KConfig / KCoreAddons / KIO / KCMUtils /
  KRunner / KWallet / KPackage / KDeclarative / KNewStuff / KSvg など
- **Plasma 6.4.5** — KWin / plasma-workspace / plasma-desktop /
  plasma-integration / breeze / kscreenlocker / kscreen / systemsettings /
  kglobalacceld / kactivitymanagerd / aurorae / qqc2-breeze-style など
- **Xwayland 24.1.9** / xkbcomp 1.4.7 / libXfont2 2.0.7 / libfontenc 1.1.8 /
  libxshmfence 1.3.3 — ksmserver は Wayland でも xcb を要求するため必須
- **M45 音声** — `alsa-lib 1.2.16` / `libpulse 17.0` / `libsndfile 1.2.2` /
  `pipewire 1.6.7` / `wireplumber 0.5.19` / `pulseaudio-qt 1.7.0` /
  `plasma-pa 6.4.5`。PipeWire は Plasma の XDG autostart から起動する
- **M45 ネットワーク/電源** — `networkmanager` の polkit 化、`plasma-nm`、
  `modemmanager` + `modemmanager-qt`、`upower` + OpenRC サービス、`powerdevil`
- **M45 アプリ/設定** — `kirigami-addons`、壁紙、`polkit-kde-agent-1`、
  Konsole / Dolphin / KCalc / Filelight 25.04.3
- **M45 GLX** — `libglvnd` の X11/GLX、Mesa `glx=dri`、`libXxf86vm`、
  Xwayland `glx=true`。Mesa は `libGL`/`libGLX_mesa` を導入
- **M46 ストレージ** — `libatasmart 0.19` / `libblockdev 3.2.2`
  (part/fs/loop/swap/mdraid/crypto/nvme)/ `udisks2 2.10.2`。暗号・NVMe のため
  `cryptsetup 2.8.8` / `lvm2 2.03.44` / `libnvme 1.16.2` / `json-c 0.19` /
  `keyutils 1.6.3` / `libbytesize 2.12` を追加。D-Bus 起動を確認済み
- **M46 アプリ** — `libarchive 3.8.11` + Ark、Kate、Gwenview(Exiv2 0.28.9 /
  LCMS2 2.16 / kImageAnnotator 0.7.2 / kColorPicker 0.3.1)、Okular
  (Poppler 25.04.0)、KWalletManager。Spectacle(OpenCV 必須)と kio-extras は
  次の増分へ
- **M46 クリーン root の依存修正** — stage3 からデスクトップを入れると
  開発 chroot では見えなかった実行時依存が欠けるため、`libtool` に libltdl
  (`--enable-ltdl-install`)を追加して `libcanberra` から依存、`kwin` に
  `xwayland` / `kglobalacceld` / `qttools`、`xwayland` に `xkbcomp`、
  `kiconthemes` に `breeze-icons`、`breeze` に `frameworkintegration`、
  `plasma-workspace` に `kunitconversion`、`libksysguard` に `lm-sensors`、
  `python3` に `mpdecimal` を追加。`plasma-meta` に `dejavu-fonts` を入れ、
  `fc-cache -f` を post_install で実行する
- **M46 日本語フォント** — `desktop/ipaex-fonts 00401`(IPAex Gothic /
  Mincho)を追加し、`plasma-meta` から依存。`fc-cache -f` を post_install で
  実行する。`fc-match :lang=ja` が IPAexGothic を返し、Kate で日本語表示を確認
- **M46 MIME データベース** — `desktop/shared-mime-info 2.4` を追加し、
  `kservice` から依存。post_install で `update-mime-database` を実行する。
  MIME データベースが無いと KService が `Service type not found` を出し、
  Dolphin のファイル種別/アイコンが解決できない
- `plasma-meta` / `kde-applications` — Plasma 本体とアプリのメタレシピ。
  M46 の `kde-applications` は Konsole / Dolphin / KCalc / Filelight / Kate /
  Ark / Gwenview / Okular / KWalletManager。`plasma-meta` は UDisks2、
  DejaVu / IPAex フォントを含む

`initscripts` は r17 で `elogind` / `polkit` の OpenRC サービスを同梱する
(どちらも opt-in)。r18 では `/usr/sbin/zignos-fstab`(マウント情報から
`/etc/fstab` を生成)と boot の `root-rw` サービス(fstab の UUID 不一致で
`/` が read-only になった場合に実デバイスで remount)を追加した。
r19 は `/etc/os-release` を 2026.11.20-alpha に、r20 は SDDM 用の
`sddm` アカウントを、r21 は sshd 起動時の OpenSSH 未導入チェックを追加した。
r22 は `video` / `render` / `input` / `audio` などの標準グループ、r23 は
`shadow` グループ、r25 は SDDM と Plasma Login Manager が共有する
`org.freedesktop.DisplayManager.conf` の D-Bus ポリシーを追加した。
r26 は `/etc/zpkg/sync.conf`(既定の GitHub リポジトリ、git モード)を
同梱し、`/etc/passwd` / `/etc/group` をパッケージで上書きせず
`/usr/share/zignos/base-accounts/` から不足分だけを post_install で
マージするようにした。M48 で SDDM 0.21 を全チャンネル、
`plasma-login-manager` 6.8.3 を testing / unstable に追加している。
M49 で zpkg 0.3.1 の git sync と base の git 2.56.1 を追加した。

設計メモは `zignos` リポジトリの `docs/desktop-kde.md`、公開サマリは
サイトの `/docs/desktop/`。

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
