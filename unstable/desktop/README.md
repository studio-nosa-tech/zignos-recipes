# desktop (unstable)

GUI パッケージのカテゴリ。**現在 220 パッケージ(Plasma 6.8.3 / KF6 6.32 / Qt 6.11.3 / Gear 26.11.80 beta / Spectacle / kio-extras)**。

このチャンネルは **Testing よりさらに挑戦的**な構成にする。設計時点では
Linux 7.4-rc / mainline、Plasma 6.9 dev / beta、Qt 6.12 dev / beta、
KF6 6.33 / master、Gear 26.12 beta。新機能・新バージョンはまずここに入れ、
Testing → Stable の順に昇格させる。

- 設計メモ: `zignos` リポジトリの `docs/desktop-kde.md`
- 公開サマリ: サイトの `/docs/desktop/`

## パッケージ構成(設計)

デスクトップ本体と KDE アプリは別のメタパッケージに分ける。

- `plasma-meta` — Plasma セッション(kwin / plasma-workspace / plasma-desktop /
  systemsettings / plasma-nm / plasma-pa / powerdevil / UDisks2 など)。
  KDE アプリは含めない
- `kde-applications` — KDE Gear のアプリ(Konsole / Dolphin / KCalc / Filelight /
  Kate / Ark / Gwenview / Okular / KWalletManager / Spectacle / kio-extras の
  11 アプリ)。Plasma 本体には依存しない

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
