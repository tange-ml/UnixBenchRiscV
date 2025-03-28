# UnixBenchRiscV
<p>
UnixBench5 や、その実行に必要な perl の xs モジュールである
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
