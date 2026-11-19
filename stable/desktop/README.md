# desktop

GUI パッケージのカテゴリ。**現在 0 パッケージ(設計中)**。

ZIGNOS はコンソール主体のソースディストリとして育ってきたため、GUI
(X/Wayland、フォント、デスクトップ環境)はまだレシピを持たない。

## 現在の設計

最初のデスクトップ環境として **KDE Plasma 6 (Wayland)** を設計中。
LTS 基盤(Plasma 6.6 / Qt 6.8 LTS / KDE Frameworks 6.18)を目標に、
OpenRC + elogind + PAM の構成を検討している。

- 設計メモ: `zignos` リポジトリの `docs/desktop-kde.md`
- 公開サマリ: サイトの `/docs/desktop/`

## 追加するときの前提

- ベースは最小構成なので、X/Wayland の依存(mesa、libdrm、xcb、
  fontconfig など)をすべてレシピ化する必要がある
- カーネルは GPU ドライバ(i915/amdgpu/nouveau/radeon)をモジュールで同梱済み。
  DRM/KMS は `core/linux/x86_64-generic.config` で有効。ただし **音声
  (`CONFIG_SND*`)は未設定**で、PipeWire の前に追加が必要
- デスクトップは stage3 / 既定イメージには含めない。`desktop/` から
  `zpkg install` する任意パッケージとする
- 追加するときは `main/` と同じレシピ形式(version / sources / checksums / build)
- KWin 6 は logind API に依存するため、OpenRC では **elogind が必須**。
  seatd との併用はしない
- PAM は現在ベースに無い。Plasma Login Manager / kscreenlocker / elogind の
  セッション作成に必要になるため、PoC では `desktop/` に限定して導入し、
  製品化時に `core` へ昇格する方針
