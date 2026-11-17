# ktav を作ると決めるまでの経緯と調査（2026-11-17〜18）

この文書は、ktav を作ると決めるまでの経緯と、そのときに調べたことの記録である。
調べた事実の多くは、項目ごとに [docs/roadmap.md](../roadmap.md) に書いた。
この文書には、ロードマップに載せない経緯、決めたこととその日付、名前の調査、外部への問い合わせの文面を残す。

## 始まり

2026-11-17 に、tett23 が「OSS のフォントの Adobe Fonts のようなもの」が作れるかを調べるよう頼んだ。
あわせて、採用するフォントの組みで、最終的なライセンスがどうなるかも確かめたいとした。
調べた結果、フォントを集めて取り込むことも、使い方ごとの義務を示すことも、公開された条文と FAQ から組み立てられると分かった。
ただし、グリフを合成したフォントの扱いには公式の資料が無く、外部に問い合わせた（「問い合わせ」の節）。

## 決めたこと

次のことは、tett23 が会話の中で決めた。
ロードマップの冒頭は、これに合わせて書いている。
ロードマップの各項目のうち、この表に無いもの（v0.1.0 で OFL のフォントだけを取り込むこと、Web フォントの配信を v0.1.0 の後に回すことなど）は、調べた結果からの提案であり、まだ決めていない。

| 日付 | 決めたこと | 理由 |
| --- | --- | --- |
| 2026-11-17 | 初めの版では OS へのフォントの同期を扱わない | Golem はプロジェクトの中のフォントだけを使うため |
| 2026-11-17 | Golem 本体には入れない | Golem と分けて、Aleph のプロジェクトにもフォントのファイルとライセンスの文書を置きやすくするため |
| 2026-11-17 | 独立したプロダクトにするか、CLI にして Aleph に組み込む | 同上。どちらにするかは未決事項 |
| 2026-11-18 | インターフェースは CLI とライブラリにし、GUI は Aleph で作る | |
| 2026-11-18 | ソフトウェアの名前は ktav、リポジトリと公開するライブラリの名前は ktav-fonts にする | 「名前」の節 |
| 2026-11-18 | ソフトウェアを ktav、ktav が持つフォントの集まりをカタログと呼ぶ | |
| 2026-11-18 | リポジトリは公開し、MIT License にする | |

## 調べたときの基準

調べたときに読んだリポジトリと版は次のとおりである。
ロードマップに書いた数と行の番号は、この版のものである。

- google/fonts：コミット 7af0d90（必要なファイルだけを sparse checkout で取得した）
- Golem：コミット 85bc0377
- Aleph：コミット 3c08218
- OFL FAQ：Version 1.1-update7（2023 年 11 月）

Fontsource と Bunny Fonts の一覧の API、Google Fonts の CSS は、2026-11-17 に取得した。

## 名前

名前は、tett23 のほかのプロダクト（Golem、Aleph など）と同じく、ゴーレムの伝承とヘブライ語から選んだ。
候補は shem、otiyot、ktav、sofer、heter だった。
shem は tett23 がすでに別のものの名前に使っているので外し、ktav（כְּתָב、「書体」「書くこと」）にした。
発音は、今のイスラエルの発音で [ˈktav] である。

2026-11-18 に、名前が空いているかを次のとおり確かめた。

| 場所 | ktav | ktav-fonts |
| --- | --- | --- |
| GitHub の tett23/ 以下 | 空いている | 空いている |
| npm | 空いている | 空いている |
| crates.io | 使われている | 空いている（ktav_fonts も空いている） |
| Hackage | 空いている | 空いている |
| Homebrew の formula | 空いている | 確かめていない |
| CTAN | 空いている | 確かめていない |

crates.io の ktav は、設定ファイルの形式「Ktav」の Rust の実装である（版 0.8.1、2026-10-21 に更新、リポジトリは GitHub の ktav-lang）。
そのため、リポジトリと公開するライブラリの名前を ktav-fonts にした。

## Golem に返したこと

調べる途中で、Golem の HTML と EPUB の出力がフォントのライセンスの文書を写さないことと、Golem の root ADR 0082 がこの扱いを Golem のロードマップの v1.0.0 に足すとしながら、足していなかったことが分かった。
2026-11-18 に、次の二つの項目を Golem のロードマップの v1.0.0 に足した（[tett23/golem#1022](https://github.com/tett23/golem/pull/1022)）。

- HTML と EPUB へのフォントのライセンスの文書の同梱
- HTML の出力の書き方を、XHTML から HTML Living Standard の HTML の書き方に変えること（ADR 0082 は、ライセンスの扱いをこの変更とあわせて決めるとしている）

## 問い合わせ

2026-11-18 に、公式の資料で答えが見つからなかった二つの問いを、tett23 がメールで送った。
問い合わせ先は、OFL を保守する SIL Global の Nicolas Spalinger（nicolas_spalinger@sil.org）と、GPL を管理する FSF のライセンスの窓口（licensing@gnu.org）である。
2026-11-18 の時点で、どちらも返事はまだ無い。
返事が来たら、この文書とロードマップの該当の項目を直す。

SIL の Contact のページのフォームは、送信の後の reCAPTCHA に接続できなかったので使っていない。
宛先は、そのページが急ぎの問い合わせの宛先として示すメンテナーのアドレスである。

### SIL（OFL）宛て

宛先は Nicolas Spalinger（nicolas_spalinger@sil.org）である。
ロードマップの「フォントを合成したときの判定」に関わる。

```text
Subject: Combining Apache-2.0 glyphs into an OFL-1.1 font

Hello,

I am building a tool that tells authors which obligations apply when they use a set of open fonts.
I could not find an answer to one case in the OFL text or the OFL-FAQ.

The OFL-FAQ covers copying glyphs from an OFL font into another font (3.2).
It does not cover the reverse: adding glyphs from a font under the Apache License 2.0 into an OFL font, and distributing the result.

Apache-2.0 section 4 allows a Derivative Work as a whole to be distributed under different terms, as long as the Apache conditions (a copy of the license, and the NOTICE file if any) are kept.
OFL-1.1 condition 5 requires the Font Software, modified or not, to be distributed entirely under the OFL.

Questions:

1. Does SIL consider a font that merges Apache-2.0 glyphs into an OFL-1.1 font distributable under OFL-1.1, provided the Apache license text is included alongside?
2. If yes, should the Apache license text be shipped with the font, or is it enough to mention the Apache-licensed source in the copyright statement?
3. Does the answer change when the Apache-licensed font has no NOTICE file?

Thank you.
```

### FSF（GPL font exception）宛て

宛先は licensing@gnu.org である。
ロードマップの「GPL フォントの HTML への同梱」に関わる。

```text
Subject: GPL font exception and fonts bundled with HTML or EPUB output

Hello,

The GPL FAQ (#FontException) says a document that embeds a font under the exception is not covered by the GPL because of that font.
The GNU FreeFont FAQ says that using a font as a web font is in effect distributing the font, and that the license and contact information must accompany it.

My case: a typesetting tool writes an HTML site or an EPUB file.
It copies the font files unchanged into a fonts/ directory next to the HTML, or inside the EPUB container, and refers to them from CSS @font-face.

Questions:

1. Is shipping the unmodified font file this way "embedding" for the purpose of the font exception, so that the HTML or EPUB is not covered by the GPL?
2. Since the font file itself is distributed, is it enough to ship the GPL text and the font's copyright notice next to the font, or must the distributor also offer the font's source (for example the FontForge .sfd files)?

Thank you.
```

## 確かめられなかったこと

次の資料は、調べた時点で読めなかった。

- Wayback Machine の古い版のページ（転送が繰り返されて読めなかった）
- listman.redhat.com のメーリングリストの記録（名前を引けなかった）
- Fedora のサイトの一部のページ（ボット対策のページで止められた）
