# Unixシェル制御構造の補足
## for 文
「[シェルスクリプト](shellscript.md)」で扱いました。より詳細は、マニュアルページを見てください。

## case文
与えられた値がどのようなパターンに当てはまるかに応じてN（≧１）方向に処理を分岐する構文です。書き方は：
```
case 式 in
パターン)  文;;
パターン)  文;;
...
esac
```
これはちょっと不思議な感じの構文ですが、まず各パターンの分岐の終わりが;;と二つ重ねになっているのは、「文」のところに複数の文を;で区切って書くことができ、それと区別するためです。もう一つ、case文の終端はesac（逆に読んでみて）であることに注意してください。

Trivialな例ですが、次のシェルスクリプトを見てください。
```
$ cat evenodd
#! /bin/sh
case `expr $1 % 2` in
0) echo "even";;
1) echo "odd";;
esac
```
（問：何をしている？）

このままだと
```
$ ./evenodd 3
odd
$ ./evenodd 8
even
```
はOKですが、
```
$ ./evenodd
expr: 文法エラー
```
となってしまいます。自分だけがちょこっと使うために作るシェルスクリプトならこれでも結構ですが、もし他の人が使う可能性があるときは（注：３ヶ月前の自分は他人）、もう少し親切にしたいものです。

そこで、次のようにしてみます。
```
$ cat evenodd1
#! /bin/sh

case $# in
0) echo "Usage: evenodd1 number"; exit 1;;
esac

case `expr $1 % 2` in
0) echo "even";;
1) echo "odd";;
esac
$ ./evenodd1
Usage: evenodd1 number
```

最初のcase文で、`$#`（シェルスクリプトに与えられた引数やオプションの数）を調べています。`$#`が0のとき、すなわち何の引数も与えずに起動されたとき、このスクリプトは使い方を表示（echo）して終了（exit）します。exitに引数1が与えられていることに注意してください。これは、「フィルタ」の回で説明した、コマンドの返り値を与えます。何らかのエラーが起きたときには、1以上の値を返します。

（問　evenodd1が正常に終了したときはどうなる？正常終了時に確実に0を返すようにするには？）

このcase文は選択肢が一つしかありませんが、`$#`が0でないときは選択肢にマッチせず、そのまま実行が続きます。この機能は次のif文でも実現可能ですが、引数が２個のとき、３個のとき、、、とスクリプトの機能を拡張することが多いので、このようにするのが定石です。

## if文
条件の正否による２方向の枝分かれをします。基本形は
```
if 条件
then
 文
fi
```
終端が"fi"であることに注意してください。条件が成り立たなかったときの動作も指定するには
```
if 条件
then
 文
else
 文
fi
```
複数の条件で次々に枝分かれするときは
```
if 条件
then
 文
elif 条件
 文
else
 文
fi
```
elifはいくつでも増やすことができます。

ここで、「条件」の場所には、（ここがシェルスクリプトのちょっと面白いところ）コマンドが入ります。そして、そのコマンドが実行された後の返り値が０の ときTRUE,  それ以外のときFALSE、となります。

例：`/etc/passwd`ファイルには、そのUnixマシンに登録されているユーザのログイン名などの情報が格納されています。（その名に反して、最近のUnixでは、パスワードそのものの情報は含まれていません。）ログイン名は各行の先頭にあります。そこで、
```
$ grep ^ichii /etc/passwd
ichii:x:500:501:Shingo Ichii:/home/ichii:/bin/bash
$ grep ^shingo /etc/passwd
$
```
とすると、ある文字列をログイン名とするユーザが登録されているかどうかがわかります。また、grepはパターンにマッチした行が見つかったとき0を、そうでないとき1を返します（マニュアルページで確かめること）。すると、次のようなことができます。
```
$ cat areyouthere
#! /bin/sh
if grep $1 /etc/passwd > /dev/null
then
 echo "yes"
else
 echo "no"
fi
$ ./areyouthere ichii
yes
$ ./areyouthere shingo
no
```
問　grep の、出力をしないようにするオプションを調べ、`/dev/null`へのリダイレクトを不要にせよ。（ここまでの実習のどこかでやったはず）

問　areyouthereを、引数を与えないときに使い方のメッセージを出力するように書き換えてください。

問　上のareyouthereは、与えた文字列が、実在するログイン名に先頭からマッチする部分文字列であれば、yesを返します。Exact matchの場合にのみyesとなるよう、改良してください。（ヒント："`:`"）

## testコマンド
if文の条件として、シェル変数がある値をとるかどうか、などの（普通のプログラミング言語的な）条件を書くときには、testコマンドを使います。また、シェルスクリプトではファイルを扱うことが多くありますが、ファイルが存在するかどうか、どのような属性か、などを条件とするときにも、testコマンドが使えます。

>というわけで、自分で作るシェルスクリプトやその他のプログラムにtestという名前をつけてはいけません。はまります。

testコマンドは、引数として調べるべき条件をとります。その条件が成り立つとき0, そうでないとき1を返します。条件としては、

- n1 -eq n2 --- 数値n1とn2が等しい（他に -ne, -lt, -le, -gt, -ge）
- s1 = s2 --- 文字列s1とs2が等しい
- s1 != s2 --- 文字列s1とs2が等しくない
- -f ファイル名 　　--- その名前のファイルが存在する
- -d ディレクトリ名 --- その名前のディレクトリが存在する

などがあります。（他にも沢山あります。）

testコマンドについて注意することはあと二つ、
別名として `[` という形もある。このとき、最後の引数 ] は単に無視される（見た目をよくするためだけにある）。これを使うと、
```
if test n1 -eq n2
```
と書く代わりに
```
if [ n1 -eq n2 ]
```
と書け、シンプルで見やすい（場合もある）。

testコマンドは独立のプログラムとしても存在する（どこに？）が、bashではシェルの内部コマンドとなっている。文法はほぼ同じ。

testコマンドを使って、areyouthereを、文字列が与えられていなければ利用者に問い合わせるように変更してみます。
```
$ cat areyouthere1
#! /bin/sh
name=$1
if [ x$name = x ]
then
echo -n "name? "
read name
fi
if grep ^${name}: /etc/passwd > /dev/null
then
echo "yes"
else
echo "no"
fi
$ ./areyouthere1
name? ichii
yes
```
testコマンド（"["の表記を使っています）の引数はちょっとした工夫で、変数nameに何も入っていないとき（i.e., areyouthere1に引数がなかったとき）は$nameは空文字列なので"x$name"は"x"と同じになることを使っています。また、read コマンドはシェルの内部コマンドで、引数として与えた変数に標準入力から読み込んだ文字列を設定します。

## while文
繰り返しを条件によって制御するときに使います。構文は
```
while 条件
do
文
done
```
条件はif文と同様（もちろんtestも使える）。次のような場合に使えます。
```
$ cat times
#! /bin/sh
count=$1
message=$2
while [ $count -gt 0 ]
do
echo $message
count=`expr $count - 1`
done
$ ./times 5 hello
hello
hello
hello
hello
hello
```
trueコマンド（問　何をするもの？）を使うと無限ループが作れます。
```
$ cat zzz
#! /bin/sh
while true
do
echo "zzz..."
sleep 5
done
$ ./zzz
zzz...
zzz...
zzz...
zzz...
^C
 （最後はコントロールCで終了）
```
ここで紹介したものの他にも、シェルには沢山の仕掛けが用意されています。マニュアルを見ながら`/usr/bin`などにあるシェルスクリプトを読んで、いろいろ工夫をしてみてください。

ただし、シェルスクリプトであまりに複雑なことをしようとすると、trickyな技を使わなければならなくなったり（それはそれで面白いのですが）、動作が遅くて使い物にならないということにもなりかねません。そういうときは、あっさりあきらめて別の手段（他のスクリプト言語とか、CやJavaといった本格的な言語）を使いましょう。

このページを準備するにあたって、
久野靖『UNIXによる計算機科学入門』改訂２版 （丸善, 2004）

を参考にしました。
