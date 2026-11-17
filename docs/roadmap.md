# ktav のロードマップ（仮）

この文書は、ktav の仮のロードマップである。
ktav は、OSS のライセンスのフォントを集めたカタログを持ち、カタログのフォントを Golem や Aleph のプロジェクトに取り込み、使うフォントの組みと出力の種類から、成果物に課される義務を示すソフトウェアである。
名前はヘブライ語で「書体」を表す語で、発音は [ˈktav] である。
リポジトリと公開するライブラリの名前は ktav-fonts とし、それ以外では ktav と呼ぶ。
crates.io の ktav は、別のプロジェクトの設定ファイルの形式が使っているためである。
ktav は、独立したプロダクトか、Aleph に組み込む CLI にし、Golem 本体には入れない。
Golem のプロジェクトだけでなく、Aleph のプロジェクトにも、フォントのファイルとライセンスの文書を置きやすくするためである。
ktav のインターフェースは CLI とライブラリにし、GUI は Aleph で作る。

ktav はまだ何も実装していないので、仕様書は無い。
そのため、この文書に載る項目は、どれもまだ無い。
版ごとの節のほかに、v0.1.0 の後に回した事項、決まっていない事項（未決事項）と、扱わないと決めた事項（課題にしなかった事項）もまとめる。

どの項目も、着手するときに ADR を作る。
実装を終えたら、その項目をこの文書から消し、振る舞いを仕様書に書く。

調べた経緯と、問い合わせの記録は [docs/research/2026-11-18-font-catalog-licenses.md](research/2026-11-18-font-catalog-licenses.md) にある。

## リリースの予定

最初の版 v0.1.0 では、Golem のプロジェクトにフォントを取り込み、PDF、HTML、EPUB の出力ごとに成果物に課される義務を示す。
OS へのフォントの同期と Web フォントの配信は、v0.1.0 では扱わない。
Golem がプロジェクトの中のフォントだけを使うので、どちらも要らないためである。

これは今の時点の計画であり、変わりうる。
版の付け方はまだ決めていない（未決事項）。

## v0.1.0

v0.1.0 のリリースまでに、次の項目を扱う。
節と項目は、上から順に実装できるように並べる。

### カタログとライセンスのデータ

- **カタログの取り込み元**：初めは [google/fonts](https://github.com/google/fonts) を取り込み元にし、各ファミリの `METADATA.pb` から名前、ファイル、ライセンスを読む。このリポジトリはライセンスごとのディレクトリ（`ofl`、`apache`、`ufl`）に分かれ、ディレクトリがその中のファイルのライセンスを示すので、最初の取り込み元に向く。v0.1.0 では、「出力ごとの義務の判定」で判定を決める OFL のフォント（ofl のディレクトリ）だけを取り込む。ofl の中の OFL 1.0 のフォント（Jomolhari と Uchen）を v0.1.0 で取り込むかは、ADR で決める。コミット 7af0d90 の METADATA.pb の license は、OFL が 1,995、APACHE2 が 42、UFL が 5 であり、日本語のサブセットを持つファミリは 69 ある。ファミリのディレクトリは ofl が 2,018、apache が 44、ufl が 5 の計 2,067 で、そのうち手元の checkout で METADATA.pb を持っていたのが上の 1,995、42、5 である。ほかの取り込み元を足すかは未決事項である。
- **ライセンスの正規化**：各フォントのライセンスを SPDX 識別子（`OFL-1.1-RFN`、`OFL-1.1-no-RFN`、`Apache-2.0`、`UFL-1.0` など）に正規化し、Reserved Font Name（以下、RFN）の名前とともに表に持つ。フォントの中の情報だけでは判定に足りないためである。`METADATA.pb` の `license` は `OFL` のように版も RFN の有無も表さず、name テーブルのライセンスの説明は自由記述で、OS/2 の `fsType` は埋め込みの許可しか表さない。実際、google/fonts（コミット 7af0d90）の Jomolhari と Uchen は、METADATA.pb では OFL だが、OFL.txt は OFL 1.0（2005-11-22）である。そのため、版はライセンスの文書から読み、OFL-1.0 と OFL-1.1 を分ける。RFN の名前は、ライセンスの文書の著作権表示とその後の宣言から読む。RFN はファミリではなく、取り込み元と版ごとに決まるので、表は取り込み元と版の組で引く。たとえば Source Serif 4 は、Adobe の配布元の 4.005R では RFN に ‘Source’ を宣言するが、google/fonts（コミット 7af0d90）のものは RFN を宣言しない。逆に Noto Sans JP は、google/fonts では RFN に ‘Source’ を宣言するが、Fontsource の npm パッケージ（@fontsource/noto-sans-jp 5.3.1）の LICENSE は著作権表示が「Google Inc.」だけで RFN を宣言しない。取り込み元のどちらが正しいかは、ktav では決めず、取り込んだ文書のとおりに扱う。カタログの取り込み元の後に置く。
- **取り込み元の固定**：取り込む google/fonts のコミットを固定し、フォントのファイルを中身のハッシュで指す。Golem は同じ入力から同じバイト列を出すので、取り込むフォントも、いつ取得しても同じバイト列でなければならないためである。

### Golem のプロジェクトへの取り込み

- **フォントの取得**：プロジェクトの設定で選んだフォントを、取り込み元から取得してプロジェクトの中に置く。ハッシュが合わなければ止める。Golem はプロジェクトの中のファイルだけからフォントを読み、システムのフォントを探さない（Golem の仕様書の「フォント」の節）。そのため、OS への同期ではなく、プロジェクトへの取り込みにする。取り込み元の固定の後に置く。
- **ライセンスの文書の同梱**：取得したフォントの隣に、ライセンスの全文と著作権表示を置く。Golem の HTML と EPUB はフォントのファイルをそのまま同梱するので、配布物にライセンスを添える必要があるためである（[OFL 1.1](https://openfontlicense.org/open-font-license-official-text/) の条件 2）。ファイルの名前は、golem new が置くもの（fonts/LICENSE-NotoSerifJP.txt など）と同じく LICENSE-<ファミリの名前> とする。Aleph は LICENSE で始まるファイルをフォントと一緒にコピーするので、この名前ならそのまま運ばれる。Aleph のテスト（app/library.test.ts）も、LICENSE-OFL.txt や LICENSE-A.txt の名前を使う。google/fonts のコミット 7af0d90 では、ofl の中の 4 つのファミリ（M PLUS Rounded 1c、Tinos、Kumar One Outline、Playwrite NZ Basic Guides）に OFL.txt が無い。同じく、apache の jsMath の 6 ファミリに LICENSE.txt が無く、ufl の Ubuntu Sans と Ubuntu Sans Mono に UFL.txt が無い。2,014 個の OFL.txt は、OFL 1.0 の 2 個も含めてどれも OFL の本文（PREAMBLE、PERMISSION & CONDITIONS、TERMINATION、DISCLAIMER）を含んでいた。ライセンスの文書を添えられないフォントは、取り込まずに知らせるか、配布元から文書を取るかを ADR で決める。フォントの取得の後に置く。

### ライセンスの判定

- **出力ごとの義務の判定**：使うフォントの組みと出力の種類から、成果物に課される義務を一覧にする。義務はフォントの種類よりも、フォントをどう使うかで決まるためである。OFL のフォントについての初めの判定は次のとおりである。
  - PDF：Golem はサブセットを埋め込む。埋め込んでも文書のライセンスは変わらない（[OFL FAQ](https://openfontlicense.org/ofl-faq/) 1.13）。サブセットの埋め込みには、改変と再配布の制限が及ばない（OFL FAQ 1.12）。そのため、PDF だけを出すなら成果物に義務は出ない。
  - HTML、EPUB：フォントのファイルをそのまま同梱するので、再配布に当たる。フォントの各コピーに、著作権表示とライセンスの全文を添える義務が出る（OFL 1.1 の条件 2）。EPUB のように、圧縮したフォルダの構造の文書に改変しないフォントを入れることは、埋め込みではなく同梱（bundling）に近いとされ、フォントは作者の情報とライセンスを失ってはならない（OFL FAQ 1.15、1.16）。
  - 和文と欧文の組み合わせ：Golem は役割ごとに和文と欧文の別のファイルを使い、グリフを合成しない。そのため、各フォントの義務が別々に並ぶだけで、ライセンスの組み合わせの問題は起きない。

  ライセンスの正規化の後に置く。
- **判定の結果の出力**：判定を、人が読む一覧と、CI が読む JSON の二つで出す。CI で、義務を満たさない配布物（ライセンスの文書を添えない HTML など）を止めるためである。出力ごとの義務の判定の後に置く。

判定は法的な助言ではなく、根拠の条文と FAQ へのリンクを添えた参考として示す。

## v0.1.0 の後に回した事項

- **Web フォントのサブセット化と改名**：HTML と EPUB に同梱するフォントをサブセット化するときに、RFN のあるフォントを改名する。Web フォントのサブセット化は改変であり（OFL FAQ 2.6）、改変版は RFN を名乗れないためである。WOFF2 への変換も、元のデータを圧縮のほかに変えないときだけ RFN を保てる（FAQ 2.2.1）。最適化しても Functional Equivalence を保てば RFN を名乗れるが（FAQ 2.7）、その条件には「同じ文字の全体を持つこと」と「著作権、ライセンス、RFN の宣言を保つこと」が含まれる（FAQ 2.8）。そのため、使う字だけに絞るサブセット化では RFN を名乗れない。Golem のロードマップは、Web 経路のフォントのサブセット化を v2 以降に回しているので、それに合わせる。golem new の英語の雛形の Source Serif 4 は Adobe の配布元から取得し、RFN に ‘Source’ があるので、サブセット化するなら改名が要る。日本語の雛形の Noto Serif JP（noto-cjk の Serif2.003）のライセンスの文書は、RFN を宣言していない。
- **Web フォントの配信**：CSS とフォントのファイルを CDN から配信する。Golem の HTML と EPUB はフォントを同梱するので、v0.1.0 では要らない。和文のフォントは `unicode-range` で分けて配信することになる（Google Fonts は、日本語の最初の 3,000 字を 20 個に分けたと[述べている](https://developers.googleblog.com/en/google-fonts-launches-japanese-support/)）。2026-11-17 に Google Fonts から Noto Sans JP の Regular の CSS を取得すると、124 個の @font-face に分かれていた。分けるのはサブセット化なので、サブセット化と改名の後に置く。OFL FAQ 2.9 は、Incremental Font Transfer のようにその場でサブセット化する技術も、十分な Functional Equivalence があれば使ってよいとする。ただし、SIL の論文「[Webfonts and Reserved Font Names](https://openfontlicense.org/webfonts-and-reserved-font-names/)」は、事前に作ったサブセットは完全な機能を届けないので Functional Equivalence を保てず、RFN の制限がかかるとする。unicode-range で分けてすべての文字を配る方式は名指しされていないが、いちばん近いのはこの事前のサブセット化である（意見）。そのため、ktav は unicode-range の分割を改変として扱い、RFN のあるフォントは改名するのがよい（意見）。論文は、WOFF と WOFF2 への変換は元のデータを変えなければ Functional Equivalence を保てるとし、その場でサブセット化する方式は条件しだいで保てるかもしれないとする。また、Web フォントは PDF のような埋め込みではない（FAQ 2.10）ので、配信には再配布の条件がかかる。配信先の候補は [Cloudflare R2](https://developers.cloudflare.com/r2/pricing/)（転送は無料）と [Bunny CDN](https://bunny.net/pricing/cdn/)（欧州と北米で $0.01/GB）である。
- **フォントを合成したときの判定**：グリフを合成した派生フォントを配布できるかを判定する。Golem はグリフを合成しないので、v0.1.0 では要らない。組み合わせごとの見込みは次のとおりである。
  - OFL と UFL、OFL と IPA：配布できない見込みが高い（推測）。OFL のグリフを含むフォントは OFL で配布しなければならず（OFL FAQ 3.2、OFL 1.1 の条件 5）、UFL 1.0 は UFL だけでの配布を求め（条件 4）、IPA フォントライセンス v1.0 は派生プログラムを同じ契約で許諾するよう求める（第 3 条第 1 項 (3)、(4)）ためである。実例に、Inconsolata（OFL）と Migu 1M（IPA ゴシックを含む）を合成する Ricty があり、生成したフォントの再配布を禁じて生成のスクリプトだけを配っている。また、UFL、OFL、Apache を合成する Summer Ghost の [issue](https://github.com/mi2428/font-summer-ghost/issues/7) は、ライセンスの文書を添えるだけでは両立しないと指摘し、結論を出さずに閉じている。
  - OFL と Apache 2.0：Apache のライセンスの写しと NOTICE を残したうえで OFL で配布できる見込みが高い（推測）。Apache 2.0 の第 4 条が、Apache の条件を守るかぎり派生物の全体を別の条件で配布することを許すためである。実例に、Noto Emoji（Apache 2.0）のグリフを含めて OFL 1.1 で配布している Cica がある。
  - OFL と Apache 2.0 の公式の資料は無い。OFL FAQ（Version 1.1-update7、2023 年 11 月）は、OFL のグリフをほかのフォントに写す場合（3.2）は扱うが、OFL 以外のフォントを OFL のフォントに合成する場合も、ほかのライセンスのフォントを OFL に許諾し直す場合も扱わない。gf-guide、Debian、Fontsource にも記述は無かった。Fedora は、読めたページには記述が無かった（一部のページは読めなかった）。google/fonts（コミット 7af0d90）の 2,014 個の OFL.txt のうち Apache に触れるのは Science Gothic だけで、それもフォントではないソースのファイルの話である。Noto、Open Sans、Roboto は Apache から OFL に移った（Noto は OFL の歴史のページ、Open Sans は google/fonts のコミット 57311b5、Roboto は同 06afa5c）が、どれも元のプロジェクトの許諾の変更で、合成の例ではない。一方、この問いはいま Google Fonts の審査に持ち込まれている。google/fonts の issue 11238（CSU Avva Shenouda、2026-11-05。ラテン文字とギリシャ文字は Apache 2.0 の Tinos にもとづく）と issue 11276（Dastkhat、2026-11-11。ラテン文字は Apache 2.0 の Roboto に由来する）は、どちらも 2026-11-17 の時点で開いており、ライセンスへの Google の回答はまだ無い。ただし Tinos は、2026-04-27 に google/fonts で OFL の版（1.340）に置き換わっている（コミット ba95515）ので、issue 11238 は OFL の版を使えば合成の問題にならない見込みである（推測）。この二つの審査の結果を見て、判定を書き直す。また、2026-11-18 に、この問いを SIL（OFL の保守者の Nicolas Spalinger）にメールで問い合わせた。Apache 2.0 のグリフを合わせたフォントを OFL 1.1 で配布できるか、Apache のライセンスの文をフォントに同梱すべきか、NOTICE が無いときに答えが変わるかを聞いた。2026-11-18 の時点で、返事はまだ無い。返事が来たら、その内容でも判定を書き直す。
  - そのため、合成の組み合わせは判定で「要確認」として示し、自動で可否を決めない。
- **OFL のほかのライセンスの判定**：Apache 2.0、UFL 1.0、IPA フォントライセンス v1.0、GPL と font exception の判定の表を足す。初めに足すのは、google/fonts にある Apache 2.0 と UFL 1.0 である。google/fonts（コミット 7af0d90）の Apache のファミリは 42（うち日本語は Kosugi と Kosugi Maru）、UFL のファミリは Ubuntu の 5 つである。Google Fonts は今は OFL だけを受け付ける（gf-guide の「License file」の節）ので、この数は増えない見込みである。各ライセンスの条項は、本文で次のとおり確かめた。
  - Apache 2.0：再配布のときにライセンスの写しを渡し（第 4 条 (a)）、改変したファイルには変更したことの目立つ通知を付け（同 (b)）、NOTICE があればその帰属表示を引き継ぐ（同 (d)）。名前の制限は無い。そのため、サブセット化して配るときは、改名は要らないが、変更の通知をフォントの中か隣に付ける。google/fonts の Apache のファミリは LICENSE.txt を持ち、NOTICE のファイルは一つも無い。
  - UFL 1.0（google/fonts の ufl/ubuntu/UFL.txt）：著作権表示とライセンスは、別のテキストファイルでも、利用者が容易に見られるメタデータの欄でもよい（条件 1）。文書は UFL にならない（条件 4）。一方、Modified Version の定義は削除と形式の変更（changing formats）を含み、改変版は、大きく変えないものも「<元の名前> derivative X」の名前にしなければならない（条件 2(c)）。そのため、WOFF2 への変換やサブセット化をしただけで改名が要り、OFL より厳しい。
  - IPA フォントライセンス v1.0（IPAex 明朝 004.01 の ipaexm00401.zip、SHA-256 9604c770…5237 に同梱された契約書。配布元はチェックサムを示さない）：PDF に埋め込み、その文書の表示にだけ使うなら、それ以上の義務は無い（第 2 条第 5 項）。無改変で再配布するなら、名前を変えず、改変せず、契約書の写しを添える（第 3 条第 2 項）。派生プログラムの定義は「deletion」を含むので（第 1 条第 3 号）、サブセット化は派生に当たり、元に戻す手段の提供など第 3 条第 1 項の条件がかかる。契約書のファイル名は IPA_Font_License_Agreement_v1.0.txt で LICENSE で始まらないので、Aleph の共有ライブラリに置くなら LICENSE-<ファミリの名前> の名前で置く。
  - GPL と font exception：未決事項の「GPL フォントの HTML への同梱」を参照。
- **OS への同期**：選んだフォントを OS に入れる。Golem はシステムのフォントを使わないので、ほかのアプリケーションでも使いたくなったときに扱う。

## 未決事項

- **ktav の置き場所**：ktav を独立したプロダクトにするか、CLI にして Aleph に組み込むか。どちらにしても Golem 本体には入れない（課題にしなかった事項）。版の付け方と、Aleph の共有ライブラリへの置き方は、この事項を決めた後に決める。
- **版の付け方**：ktav の版の付け方。Golem は X.Y.Z（プレリリースは -rc1 など）を使い、1.0.0 までは互換を壊す変更で Y を上げ、版の正本をリポジトリの根の VERSION に置く（Golem の仕様書の「バージョン」の節）。ktav を Aleph に組み込む CLI にするなら、ツールの版は Aleph に従う。どちらの場合も、取り込んだデータ（google/fonts のコミットとライセンスの表）はツールと別に更新されるので、データの版をツールの版と分けるかを決める。
- **プロジェクトの設定の形**：使うフォントをどこに書くか。Golem の `style.toml` のフォントの役割から読むか、別のファイルに書いて取得したファイルのパスを `style.toml` から指すか。参考になるのは Golem のプラグインの仕組みである（Golem の仕様書の「プラグイン」の節）。人がマニフェストに版ごとに固定した URL で宣言し、処理系が初めて取得したときにハッシュを plugins.lock に記録する。以後はハッシュが合わなければエラーにし、ハッシュをキーにキャッシュし、ロックはユーザーが明示したときだけ更新する。一方、path で指すプロジェクトの中のファイルは、版管理で固定されるのでロックにもキャッシュにも入れない。Golem から見ればフォントは path と同じくプロジェクトの中の入力である。そのため、取得の固定は、plugins.lock と同じ形の ktav のロックファイルが受け持つのが自然である（意見）。
- **ほかの取り込み元**：google/fonts に無いフォントをどこから取り込むか。[Fontsource の API](https://fontsource.org/docs/api/fonts) はフォントごとに `license` を SPDX に近い形で返す。2026-11-17 に一覧の API（/v1/fonts）を取得すると、2,103 のフォントのうち Google Fonts 由来が 1,984、そのほかが 119 であり、ライセンスは OFL-1.1 が 2,059、Apache-2.0 が 36、UFL-1.0 が 5、CC0-1.0、mit、Unlicense が 1 ずつだった。GPL のフォントは無く、Google Fonts 由来でない日本語のフォントも無い。値は mit のように SPDX の終わりの形（MIT）と文字の大小が違うものがあるので、取り込むなら正規化が要る。Bunny Fonts の一覧の API（/list、全 1,975）にはライセンスの欄が無く、CSS が指すフォントのファイルは jsDelivr の Fontsource の配布物（noto-sans-jp@5.3.1）だった（2026-11-17 に取得）。Bunny Fonts は Fontsource が作ったファイルを配っているので、取り込み元としては Fontsource と同じ扱いになる（推測。確かめたのは Noto Sans JP の 1 件）。google/fonts に無い日本語のフォント（IPA フォントや、和文と欧文を合成したプログラミングフォントなど）は、GitHub のリリースから直に取ることになる。そのときライセンスの文書の形がそろわない。Source Han Serif と Source Han Sans の LICENSE.txt は OFL の本文の形で RFN に ‘Source’ を宣言し、noto-cjk の Sans/LICENSE は RFN を宣言しない。HackGen と UDEV Gothic の LICENSE は OFL の本文ではなく、ソースのフォントごとの LICENSE_<名前> を指す要約である。そのため、google/fonts のための RFN の読み取り方は、そのままでは使えない。
- **RFN の読み取り方**：RFN はライセンスの文書の著作権表示の後に書かれる（OFL 1.1 の定義）が、書き方に決まった形が無い。`OFL.txt` の冒頭の "Reserved Font Name" を機械で読むか、人が表に書くかを決める。google/fonts のコミット 7af0d90 では、2,014 個の OFL.txt のすべてが "This Font Software is licensed under" の行を持ち、その前の著作権表示で RFN を宣言するのは 560 個だった（"Reserved Font Name" が行をまたぐものがあるので、改行を空白にしてから探す）。日本語のファミリで RFN を宣言するのは Noto Sans JP（‘Source’）と IBM Plex Sans JP（"Plex"）の二つである。ただし、宣言の書き方は引用符や複数形（Reserved Font Names）が揺れるので、名前の取り出しには人の確かめが要る。一方、機械で取り出す精度は高い。560 件について、「Reserved Font Name(s)」の後の引用符で囲まれた名前をすべて取り、引用符が無ければ最初の句点までをカンマと「and」で分けると、すべてから名前が取れた。引用符の形が 481 か所、引用符の無い形が 94 か所で、引用符の無い形のうち 6 件（Molengo と Nanum の 5 ファミリ）は複数の名前をカンマで並べる。URL や著作権の語が混じるような怪しい抽出は 0 件だった。Aref Ruqaa のように、元にした別のフォントの名前（EURM10）を RFN にするものもある。そのため、機械で取り出し、取り込み元の版を上げるときに差分だけを人が確かめる形ができる。RFN はその名前だけを縛るので（OFL 1.1 の条件 3）、Noto Sans JP のようにファミリ名が RFN を含まなければ、改変しても名前を変えなくてよい見込みが高い（推測）。
- **Golem の `fsType` の検査との分担**：Golem はすでに、`fsType` が埋め込みを禁じるフォントを G08004 のエラーにしている。ライセンスの表と `fsType` が食い違うときにどちらを信じるかを決める。google/fonts（コミット 7af0d90）の全 2,067 ファミリの各 1 ファイルの fsType を読むと、0（Installable）が 2,049、0x8（Editable）が 13、0x4（Preview & Print）が 4（Amstelvar Alpha、Decovar Alpha、Noto Color Emoji Compat Test、Zilla Slab Highlight）、予約のビットだけの 0x1 が 1（Bungee Color）だった。日本語の 69 ファミリと、golem new が取得する SourceSerif4-Regular.otf（4.005R）と NotoSerifJP-Regular.otf（Serif2.003）は、すべて 0 だった。Golem が埋め込みを禁じるとみなす値（ビット 1 だけ、ビット 8、ビット 9）は無いので、G08004 に当たるフォントは無い。一方、OFL は埋め込みも改変も許すのに fsType で Preview & Print を名乗るフォントが 4 つあり、ライセンスと fsType の食い違いは実在する。ただし、どれも fsType のほうが厳しい向きなので、Golem で組めなくなることは無い。
- **GPL フォントの HTML への同梱**：font exception は、文書にフォントを埋め込んだ（embed）ときに、そのフォントだけを理由に文書が GPL にならないと定める（[GNU の FAQ](https://www.gnu.org/licenses/gpl-faq.html#FontException)）。HTML や EPUB にフォントのファイルを同梱することがこの埋め込みに当たるかは、FAQ に書かれていない。FAQ が引く [FSF の記事](https://www.fsf.org/blogs/licensing/20050425novalis)は、CSS の @font-face でフォントを参照するだけなら文書はフォントの派生物にならず、埋め込んだときに派生物になると述べる。Golem の HTML は @font-face で fonts/ のファイルを参照するので、この考え方では HTML は GPL にならない。[GNU FreeFont の FAQ](https://www.gnu.org/software/freefont/FAQ.html) は、Web フォントとして使うことはフォントの配布に当たり、ライセンスと連絡先を添えれば GPL を満たし、サイトは GPL を継がないと述べる。これは FreeFont についての説明だが、同じ font exception のフォントにも当てはまる見込みが高い（推測）。そのため、GPL のフォントも、HTML と EPUB では OFL と同じく「ライセンスの文書を添える」義務として判定できる。さらに、FreeFont の配布物に入っている指針（Debian の fonts-freefont 20211204+svn4273-2 の notes/webfont_guidelines.txt）は、@font-face は GPL から見るとフォントのファイルの配布であり、フォントを参照するページもフォントを置くサイトも GPL の義務を負わないと述べる。届けるファイルには、ライセンスとソースの入手先の通知が要り、改変したら誰がいつ何を変えたかの通知も要る。FreeFont はこの通知をフォントのファイルの中に埋め込んでいるので、改変しないまま配れば義務を満たす。そのため、GPL のフォントの判定は、ライセンスとソースの入手先の通知が、フォントの中か隣の文書にあるかを確かめる形にする。なお、google/fonts（コミット 7af0d90）のフォントは ofl、apache、ufl のどれかにあり、GPL のフォントは無い（cc-by-sa のディレクトリは Google Fonts Knowledge の文書で、フォントではない）。そのため、この事項は v0.1.0 には影響せず、ほかの取り込み元を足すときに決めればよい。また、Linux Libertine のように GPL と font exception に加えて OFL でも配られるフォントがある（CTAN の fonts/libertine/doc/LICENCE.txt）。こうした二つのライセンスのフォントは、利用者が OFL を選べるので、ktav も OFL として判定でき、GPL についての推測に頼らずに済む。2026-11-18 に、FSF（licensing@gnu.org）にメールで問い合わせた。HTML や EPUB に改変しないフォントのファイルを同梱することが font exception の埋め込みに当たるか、GPL の本文と著作権表示を添えれば足りるか、フォントのソースの提供も要るかを聞いた。2026-11-18 の時点で、返事はまだ無い。
- **Aleph の共有ライブラリへの置き方**：ktav が取得したフォントを、各プロジェクトの `fonts/` に置くか、Aleph の共有ライブラリの `fonts/` に置くか。Aleph の共有ライブラリは、設定で選ぶ一つのフォルダである。Aleph は、`style.toml` が指すフォントのうちプロジェクトに無いものを、同じ名前で共有ライブラリから探してプロジェクトの `fonts/` にコピーし、`LICENSE` で始まるファイルも一緒にコピーする（Aleph の仕様書の「共有ライブラリとアセット」の節、Aleph の ADR 0014）。そのため、共有ライブラリに置けば、各プロジェクトへのコピーは Aleph が受け持つ。ただし、Aleph の実装（Aleph のコミット 3c08218 の app/library.ts の 69〜86 行）は、共有ライブラリの fonts/ からファイル名だけでフォントを探し、フォントを一つでもコピーしたら LICENSE で始まるファイルをすべて写し、同じ名前のファイルは上書きしない。そのため、共有ライブラリに置くフォントとライセンスの文書は、ファミリごとに名前が重ならないようにする必要がある。版の違う同じ名前のフォントも区別されない。ktav の置き場所を決めた後に決める。
- **Golem の HTML と EPUB へのライセンスの文書の同梱**：Golem の HTML と EPUB の出力は、`style.toml` が指すフォントのファイルだけを `fonts/` に写し、ライセンスの文書を写さない（Golem のコミット 85bc0377 の `modules/golem-html/src/Golem/Html.hs` の 340〜346 行。EPUB も同じ一覧を使う）。ライセンスの文書を写すのは、Golem のマニュアルを組む `build.sh` だけである（Golem の仕様書の「マニュアルの組み方とライセンス」の節）。そのため、Golem で組んだ HTML や EPUB をそのまま配布すると、OFL 1.1 の条件 2 を満たさない。Golem の出力が `fonts/LICENSE-*` も写すように Golem を変えるか、ktav の判定でこの配布物を CI で止めるかを決める。Golem を変えるなら、Golem の ADR で決める。Golem はこの欠けを知っている。Golem の root ADR 0082（採択）は「実装しないこと」で、Golem の本体が HTML と EPUB にフォントのライセンスの文書を写す扱いを、どのファイルがライセンスの文書かを知る手段と、EPUB の中のフォントを埋め込みとみなすかとあわせて v1.0.0 のころに決めるとしている。この項目は、2026-11-18 に Golem のロードマップの v1.0.0 に足した（[tett23/golem#1022](https://github.com/tett23/golem/pull/1022)）。ktav がライセンスの文書を LICENSE-<ファミリの名前> の名前で置けば、「どのファイルがライセンスの文書か」に名前で答えられる。ADR 0082 のもう一つの問い「EPUB の中のフォントを埋め込みとみなすか」には、OFL については OFL FAQ 1.15 が答えている。圧縮したフォルダの構造の文書に改変しないフォントを入れることは、埋め込みではなく同梱に近い。そのため、Golem の EPUB も、HTML と同じくライセンスの文書を添える必要がある。
- **`golem new` のフォントの取得との分担**：`golem new` は、雛形のフォント（日本語は Noto Serif JP、英語は Source Serif 4）とそのライセンスの文書を、配布元のタグで固定した URL から取得し、Golem のソースに固定した SHA-256 で検査して `fonts/` に置く（Golem の仕様書の「golem new」の節、golem-cli ADR 0003）。Aleph は新しいプロジェクトを `golem new --no-fonts` で作り、フォントを共有ライブラリからコピーする。ktav の取得と、`golem new` の取得を並べておくかを決める。

## 課題にしなかった事項

- **Golem 本体への組み込み**：ktav を Golem のサブコマンドやモジュールにはしない。Golem は、ktav が置いたフォントのファイルを、ほかのフォントと同じくプロジェクトの中から読む。判定の JSON を読むのは CI である。Golem と分けるのは、Aleph のプロジェクトにも同じ手順でフォントとライセンスの文書を置けるようにするためである。
- **有償のフォントとプロプライエタリのフォント**：扱わない。ライセンスがフォントごとに違い、公開された条文から義務を判定できないためである。扱う必要が出たら、新しい ADR で決め直す。
- **法的な助言としての判定**：判定は、根拠の条文と FAQ へのリンクを添えた参考にとどめる。判定を法的な助言として示したり、保証したりはしない。
- **フォントの利用を縛る仕組み**：DRM、難読化、ktav の利用条件によって、取得したフォントの利用を制限しない。OFL FAQ は、Adobe Fonts のようにフォントを届けるサービスが、OFL のフォントをほかの環境で使うことや、契約が終わった後も使い続けることを妨げてはならないとし（1.23）、サービスの利用条件が OFL を制限することも認めない（1.24）。難読化や DRM そのものは OFL の範囲外だが、使っても OFL の規則は変わらない（1.17）。ktav は取得したファイルをそのまま置くので、この制約を守れる。
- **ktav の GUI**：ktav 自身は GUI を持たず、CLI とライブラリだけを提供する。フォントを選んだり判定を見たりする GUI は、Aleph で作る。
