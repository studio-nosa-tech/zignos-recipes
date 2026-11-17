# main

CLI 実用パッケージのカテゴリ(63 パッケージ)。ベースシステムは
`zignos-build/base.list` の最小構成(Gentoo の @system 相当)なので、
エディタ・SSH・ネットワーククライアント・WiFi・ドキュメント・開発ツールは
ここから `zpkg install <pkg>` で入れる。

## 主な中身

- シェル環境: vim / nano / less / tmux / htop / zsh / nnn / moreutils
- ネットワーク: curl / wget / rsync / iproute2 / wpa_supplicant / iw /
  wireguard-tools / openssh / dhcpcd / chrony / dbus
- ネットワーク診断: iputils / ethtool / mtr / nmap / whois / iperf / tcpdump / socat
- 開発: git / automake / libtool / ccache / fzf / lua / sqlite
- 圧縮・ファイル: 7zip / lz4 / brotli / pigz / zip / unzip / pv / tree / ncdu / entr
- 監視・セキュリティ: btop / age / restic / lsof / strace / sysstat /
  smartmontools / pciutils / lshw / dmidecode
- ハードウェア/UEFI: linux-firmware / efibootmgr / efivar / dosfstools
- ドキュメント: man-db / man-pages / groff

`sudo` と `doas` はどちらも任意(ベースの特権昇格は `su`)。
