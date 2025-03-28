# UnixBenchRiscV
<p>
UnixBench5 や，その実行に必要な perl の xs モジュールである
Time::HiRes を RISC-V 向けにクロスコンパイルしてターゲットへインストー
ルする手順について説明します．
</p>

## Time-HiRes のビルドとインストール

### クロスコンパイル環境での作業
<p>
下記サイトの左リスト中程にある TOOLS/Download から
Time-HiRes-1.9764.tar.gz(最新版)をダウンロードしてください．
</p>

[https://metacpan.org/pod/Time::HiRes](https://metacpan.org/pod/Time::HiRes)

ダウンロードしたアーカイブを展開してください．

```
tar -zxvf Time-HiRes-1.9764.tar.gz
cd ~/Time-HiRes-1.9764/
```

<p>
下記コマンドを実行して Makefile を生成してください．
</p>

```
perl Makefile.PL
```

<p>
生成された Makefile の 41-42 行, 49-52 行のツールを x86_64 から
riscv64 へ下記 diff のように書き換えてください．
</p>

```
41,42c41,42
< AR = ar
< CC = x86_64-linux-gnu-gcc
---
> AR = riscv64-linux-gnu-ar
> CC = riscv64-linux-gnu-gcc
49,52c49,52
< LD = x86_64-linux-gnu-gcc
< LDDLFLAGS = -shared -L/usr/local/lib -fstack-protector-strong
< LDFLAGS =  -fstack-protector-strong -L/usr/local/lib
< LIBC = /lib/x86_64-linux-gnu/libc.so.6
---
> LD = riscv64-linux-gnu-ld
> LDDLFLAGS = -shared -L/usr/riscv64-linux-gnu/lib -fstack-protector-strong
> LDFLAGS =  -fstack-protector-strong -L/usr/riscv64-linux-gnu/lib
> LIBC = /usr/riscv64-linux-gnu/lib/libc.so.6
```

<p>
ビルドしてください．
</p>

```
make
```

### ターゲットでの作業

<p>
ターゲットに下記ディレクトリを作成して，クロスコンパイル環境で生成した
HiRes.pm と HiRes.so をインストール(コピー)してください．
</p>

```
make -p /usr/local/lib/riscv64-linux-gnu/perl/5.34.0/
cp ~/Time-HiRes-1.9764/blib/lib/Time/HiRes.pm /usr/local/lib/riscv64-linux-gnu/perl/5.34.0/
cp ~/Time-HiRes-1.9764/blib/arch/auto/Time/HiRes/HiRes.so /usr/local/lib/riscv64-linux-gnu/perl/5.34.0/
```


## UnixBench5 のビルドとインストール

### クロスコンパイル環境での作業

<p>
下記サイトで UnixBench5.1.3.tgz をダウンロードしてください．
</p>

[https://code.google.com/archive/p/byte-unixbench/downloads](https://code.google.com/archive/p/byte-unixbench/downloads)

<p>
ダウンロードしたアーカイブを展開してください．
</p>

```
tar -zxvf UnixBench5.1.3.tgz
cd ~/UnixBench/
```

<p>
Makefile の 56 行のツールを x86_64 から riscv64 へ下記 diff のように書
き換えてください．
</p>

```
56c56
< CC=gcc
---
> CC=riscv64-linux-gnu-gcc
```

<p>
ビルドしてください．
</p>

```
make
```

<p>
./Run の main() がコールする preChecks():1767 を下記 diff のようにコメ
ントアウトしてください．
</p>

```
1767c1767
<     preChecks();
---
>     # preChecks();
```

### ターゲットでの作業

<p>
~/UnixBench/ をそのままターゲットの /root へコピーしてください．
</p>

## UnixBench5 の実行
### ターゲットでの作業

<p>
ターゲットで下記コマンドを実行してください．
</p>

```
cd ~/UnixBench/
./Run
```

### UnixBench5 の実行結果

| 実施条件     | 説明                       |
| ---          | ---                        |
| テスト日時   | 2025/03/27 22:30           |
| FPGA DATA    | 4014_03_rev13_sha_0624c4   |
| 標準出力ログ | UnixBench-202503272230.tar |

<p>
下記結果の INDEX は，基準となる Sun SPARCstation 20 SM61 を 10 とした
ときの値です．
</p>

```
System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0     163658.4     14.0
Double-Precision Whetstone                       55.0         39.4      7.2
Execl Throughput                                 43.0         10.9      2.5
File Copy 1024 bufsize 2000 maxblocks          3960.0       2161.0      5.5
File Copy 256 bufsize 500 maxblocks            1655.0        599.5      3.6
File Copy 4096 bufsize 8000 maxblocks          5800.0       5958.0     10.3
Pipe Throughput                               12440.0       2699.0      2.2
Pipe-based Context Switching                   4000.0        487.6      1.2
Process Creation                                126.0         21.1      1.7
Shell Scripts (1 concurrent)                     42.4         29.1      6.9
Shell Scripts (8 concurrent)                      6.0          7.3     12.1
System Call Overhead                          15000.0       4676.3      3.1
                                                                   ========
System Benchmarks Index Score                                           4.4
```

<p>
整数演算，8 つのシェルスクリプト実行は，Sun SPARCstation 20 SM61 と同
等以上のパフォーマンスと言えます．4 KByte バッファー 8000 ブロックのファ
イルコピーに関しては，HDD と TmpFs の比較となるため優劣を比較しにくい
と考えます．
</p>

| システム                 | コア数 | 周波数 |
| ---                      |    --- | ---    |
| Sun SPARCstation 20 SM61 |      1 | 60 MHz |
| 本ターゲット             |      8 | 25 MHz |

<p>
(1*60)/(8*25) = 0.3 倍すると本ターゲットコアの効率が予測できます．
</p>
