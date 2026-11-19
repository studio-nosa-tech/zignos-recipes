# desktop

GUI パッケージのカテゴリ。**現在 0 パッケージ**(未着手)。

ZIGNOS はコンソール主体のソースディストリとして育ってきたため、GUI(X/Wayland、
フォント、デスクトップ環境)はまだレシピを持たない。追加する場合の前提:

- ベースは最小構成なので、X/Wayland の依存(mesa、libdrm、xcb、fontconfig など)を
  すべてレシピ化する必要がある
- カーネルは GPU ドライバ(i915/amdgpu/nouveau/radeon)をモジュールで同梱済み。
  DRM/KMS は `core/linux/x86_64-generic.config` で有効
- 追加するときは `main/` と同じレシピ形式(version / sources / checksums / build)
