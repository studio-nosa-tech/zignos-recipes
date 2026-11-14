# zignos-recipes

ZIGNOS のレシピツリー(Gentoo の gentoo/gentoo リポジトリ相当)。

## 構成

- `core/` — bootstrap の最終システム + 最小ユーザーランド(LFS 13.1-systemd 準拠)
- `main/` — CLI 実用パッケージ(M4 以降に拡充)
- `desktop/` — GUI(M4 以降に拡充)

## レシピ形式

各パッケージはディレクトリ型(KISS 式): `version` / `sources` / `checksums` / `depends` / `build`。詳細は zignos リポジトリの `docs/design-pkg.md`。

## 現在の状態

- `core/` は LFS 13.1-systemd のバージョンにピン留め済み(GCC 16.2.0 / Glibc 2.44 / Binutils 2.47 / Bash 5.3 / Coreutils 9.11 / Go 1.27.4 など)。**全 sha256 を 2026-11-15 に実ダウンロードして検証済み**。
- `build` スクリプトは LFS の手順を zpkg の destroot 契約(`$ZPKG_DESTROOT` に成果物を置く)に翻訳したもの。**テストスイートは v0 では一律スキップ**。
- LFS からの意図的な差分: coreutils の i18n パッチと autoconf/automake チェーン省略、tar の acl パッチ省略、bzip2 の docs パッチ省略、systemd 系なし(init は M3 で決定)。詳細は zignos-build/bootstrap.sh のヘッダーコメント。
- cross ビルドの一時ツール群(LFS ch5/ch6/ch7)はレシピ化せず zignos-build の `stage0.sh` / `stage1.sh` が担当する。zpkg は最終システムにのみ使う。
