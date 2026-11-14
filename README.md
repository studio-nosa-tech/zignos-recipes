# zignos-recipes

ZIGNOS のレシピツリー(Gentoo の portage tree 相当)。

## 構成

- `core/` — bootstrap に必要な最小セット(LFS 相当)
- `main/` — 実用パッケージ(M4 以降に拡充)

## レシピ形式

各パッケージはディレクトリ型(KISS 式): `version` / `sources` / `checksums` / `depends` / `build`。詳細は zignos リポジトリの `docs/design-pkg.md`。

## 現在の状態(骨格)

- バージョン・URL は暫定値。**正式ビルド前に LFS 13.1 の該当バージョンにピン留めし、`checksums` を必ず埋めること**(checksums が空のままの正式ビルドは禁止)
- `build` スクリプトは骨格。M2 で cross pass1 / pass2 など段階別のレシピ分割を行う
- `depends` は bootstrap 段階の順序制御を zignos-build 側で行うため、当面は空
