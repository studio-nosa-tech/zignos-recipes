# desktop (stable)

GUI パッケージのカテゴリ。**現在 212 パッケージ(M50: KDE Plasma 6.4.5 / KF6 6.18.0 / Qt 6.8.5 / UDisks2 / KDE アプリ / Spectacle / kio-extras)**。

このチャンネルは **安定版 / LTS** を採用する。Qt 6.8 LTS / KF6 6.18 と
組み合わせられる **Plasma 6.4.5** を固定する(Plasma 6.6 系は Qt 6.10 /
KF6 6.22 を要求するため Testing 以降)。長期サポートと保守性を優先し、
Testing で検証済みの変更だけを LTS へバックポートする。

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
- PAM は M44 で `core/linux-pam` へ移し、shadow を PAM 対応にした。
  kscreenlocker / Plasma Login Manager のセッション作成に使う
- カーネルは DRM/KMS と virtio-gpu を持つが、音声(`CONFIG_SND*`)は未設定。
  PipeWire の前に追加が必要
- レシピ形式は `main/` と同じ(version / sources / checksums / build)
