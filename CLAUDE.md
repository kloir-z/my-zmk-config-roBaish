# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

roBa という自作型分割ワイヤレスキーボード用の ZMK ファームウェア設定リポジトリ。

- **マイコン**: Seeeduino XIAO BLE (nRF52840)
- **左手側 (roBa_L)**: ペリフェラル、ロータリーエンコーダー搭載可能
- **右手側 (roBa_R)**: セントラル、PMW3610 トラックボール搭載
- **ZMK フォーク**: 公式ではなく `kloir-z/zmk` を使用

## ビルド

ファームウェアは GitHub Actions で自動ビルドされる（push/PR 時）。ローカルビルドは ZMK の west ワークスペースが必要で、このリポジトリ単体では行わない。

ビルドターゲット（`build.yaml` で定義）:
- `roBa_R` + `seeeduino_xiao_ble/zmk` (studio-rpc-usb-uart スニペット付き)
- `roBa_L` + `seeeduino_xiao_ble/zmk`
- `settings_reset` + `seeeduino_xiao_ble/zmk`

## ファイル構成

```
config/
  west.yml                        # ZMK 依存管理（kloir-z フォーク指定）
  roBa_L.keymap                   # メインキーマップ（左右共通）
  roBa_R.keymap                   # 右手側設定（roBa_L.keymap をインクルード + トラックボール設定）
  boards/shields/Test/
    roBa.dtsi                     # 共通ハードウェア定義（マトリクス、物理レイアウト、エンコーダー）
    roBa_L.overlay                # 左手側列GPIO定義
    roBa_R.overlay                # 右手側列GPIO定義 + SPI/トラックボール設定
    Kconfig.defconfig             # シールド設定（分割キーボード、セントラル/ペリフェラル）
    Kconfig.shield                # シールド選択定義
    roBa.zmk.yml                  # ZMK シールドメタデータ
build.yaml                        # GitHub Actions ビルドマトリクス
zephyr/module.yml                 # ZMK モジュール定義
```

## キーマップ構成

**レイヤー定義**（`roBa_L.keymap`）:
- 0: `default_layer` — QWERTY
- 1: `Sym_Num` — 記号・数字
- 2: `Func_Navi` — ファンクション・カーソル移動
- 3: `Other` — メディア・Bluetooth
- 4: `MOUSE` — オートマウスレイヤー（トラックボール移動で自動活性化）
- 5: `SCROLL` — スクロールレイヤー

**日本語キーボード変換定義**: OS設定を日本語キーボードのまま使う `JP_*` マクロが `roBa_L.keymap` 冒頭に定義されている。

**オートマウスレイヤー (AML)**: トラックボール操作で MOUSE レイヤーが自動的に ON になる（`roBa_R.keymap` の `zip_temp_layer` 入力プロセッサ）。`excluded-positions` 以外のキーを押すと自動で OFF になる。`require-prior-idle-ms` でタイピング中の誤発動を防ぐ。クリック/スクロール用キー(I/O/P/K/右親指)とモディファイア(Ctrl/Win/Alt/Shift)は `excluded-positions` に入れて AML を維持する。

## キーマトリクス

4行 × 11列、合計43キー。`roBa.dtsi` で定義。右手側オーバーレイは `col-offset = <6>` を使用。

## 変更時の注意

- キーマップ変更は `config/roBa_L.keymap` を編集する
- シールド定義を変更した場合、`build.yaml` のターゲット名（`roBa_R`, `roBa_L`）は `Kconfig.shield` の shields_list 名と一致している必要がある
