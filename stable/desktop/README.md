# desktop (stable)

GUI パッケージのカテゴリ。**現在 0 パッケージ(設計中)**。

このチャンネルは **安定版 / LTS** を採用する。Linux は **6.18 LTS**、
KDE は **Plasma 6.6 LTS** / Qt 6.8 LTS / KF6 6.18 / Gear 25.04.3。
長期サポートと保守性を優先し、Testing で検証済みの変更だけを LTS へ
バックポートする。

- 設計メモ: `zignos` リポジトリの `docs/desktop-kde.md`
- 公開サマリ: サイトの `/docs/desktop/`

## パッケージ構成(設計)

デスクトップ本体と KDE アプリは別のメタパッケージに分ける。

- `plasma-meta` — Plasma セッション(kwin / plasma-workspace / plasma-desktop /
  systemsettings / plasma-nm / plasma-pa / powerdevil / PLM など)。
  KDE アプリは含めない
- `kde-applications` — KDE Gear のアプリ(Dolphin / Konsole / Kate / Ark /
  Gwenview / Okular / Spectacle / KCalc / Filelight / KWalletManager など)。
  Plasma 本体には依存しない

将来は `kde-applications-{utilities,graphics,multimedia,network,office,pim,sdk}`
のサブメタへ分割できる。メタパッケージは `/usr/share/zignos/meta/<name>` の
マーカーファイルを持ち、`depends` でコンポーネントを束ねる。

## 昇格順

**Unstable → Testing → Stable** を守る。逆流させない。Stable は LTS
ブランチなので、Testing の最新版をそのまま入れるのではなく、検証済みの
修正を LTS へバックポートする形で反映する。

## 追加するときの前提

- デスクトップは stage3 / 既定イメージには含めない。`desktop/` から
  `zpkg install` する任意パッケージとする
- ベースは最小構成なので、Wayland / Mesa / フォント / Qt6 / KF6 などを
  すべてレシピ化する
- KWin 6 は logind API に依存するため、OpenRC では **elogind が必須**。
  seatd との併用はしない
- PAM は現在ベースに無い。Plasma Login Manager / kscreenlocker / elogind の
  セッション作成に必要になるため、PoC では `desktop/` に限定して導入し、
  製品化時に `core` へ昇格する方針
- カーネルは DRM/KMS と virtio-gpu を持つが、音声(`CONFIG_SND*`)は未設定。
  PipeWire の前に追加が必要
- レシピ形式は `main/` と同じ(version / sources / checksums / build)
