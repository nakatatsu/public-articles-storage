# 最適化したCLAUDE.mdを使うための親設定・子設定ルール

CLAUDE.mdをどこに配置すれば読まれるのか・読まれないのかを検証しました。

## 拝啓

Claude Codeの/initコマンドでCLAUDE.mdを作成すると、プロジェクト全体の要約を詰め込んだCLAUDE.mdがリポジトリルートに作成されます。このまま利用すると、内容がおおざっぱかつ、個別の作業に必ずしも必要ない情報を毎回読み込ませることになります。これは各作業の精度の低下や無駄なトークン費用の発生につながります。CLAUDE.mdは作業に最適化したものを使うべきです。そしてそのためには、CLAUDE.mdが読まれる条件を把握している必要があります。

REF: https://www.anthropic.com/engineering/claude-code-best-practices

## リポジトリルートに配置したCLAUDE.md

まず基本から。

リポジトリルートに配置したCLAUDE.mdは、サブディレクトリにいようと、同一階層に別のCLAUDE.mdがあろうと、読まれます。


```console
~/repositories/ClaudeCodeTest$ cat CLAUDE.md
# CLAUDE.md

Reply with 'Awesome, Inc.' when asked for the company name.


~/repositories/ClaudeCodeTest$ echo "What is the company name?" | yolo -p
Awesome, Inc.
~/repositories/ClaudeCodeTest$ cd GeneralAffairs/GeneralServices/DocumentManagement/Digitization
~/repositories/ClaudeCodeTest/GeneralAffairs/GeneralServices/DocumentManagement/Digitization$ echo "What is the company name?" | yolo -p
Awesome, Inc.
~/repositories/ClaudeCodeTest/GeneralAffairs/GeneralServices/DocumentManagement/Digitization$ cd ../../../
~/repositories/ClaudeCodeTest/GeneralAffairs$ echo "What is the company name?" | yolo -p
Awesome, Inc.
~/repositories/ClaudeCodeTest/GeneralAffairs$ ls
CLAUDE.md  GeneralServices
~/repositories/ClaudeCodeTest/GeneralAffairs$
```

※ yolo はClaude Codeのエイリアスです。.bashrcに右記のように設定しています。`  alias yolo='claude  --dangerously-skip-permissions'`

従ってリポジトリルートのCLAUDE.mdには、プロジェクト全体に関わる内容を厳選して配置するのがよいでしょう。サブディレクトリで作業する場合も読まれてしまうため、ここに雑多な内容を大量に詰め込むと、必要ない情報がコンテキストに毎回含まれてしまう恐れがあります。

## サブディレクトリに配置したCLAUDE.md

サブディレクトリに配置したCLAUDE.mdも、そのサブディレクトリから実行すれば読むことができます。

```console
~/repositories/ClaudeCodeTest/GeneralAffairs$ cat CLAUDE.md
# CLAUDE.md

Say 'Mr. Yamaguchi' when asked about the boss of the department.

Reply "General Affairs" when asked about the name of the department.
~/repositories/ClaudeCodeTest/GeneralAffairs$ echo "Who is Mr.Yamaguchi?" | yolo -p
Mr. Yamaguchi is the boss of the General Affairs department.
```

このため、コンテキストをディレクトリで表現できている状態、例えば下記のようなディレクトリになっている場合が該当しますが、そのようなケースでは各サブディレクトリごとにCLAUDE.mdを配置し、それぞれにあわせて最適化すると効果的です。

```console
backend/
frontend/
spec/
```


## 親階層のCLAUDE.mdも読まれる

リポジトリルートに配置していなくても、親階層にいれば読まれます。

例えばこのような状態になっているとしまして

```console
├── GeneralAffairs
│   ├── CLAUDE.md
│   └── GeneralServices
│       └── DocumentManagement
│           └── Digitization
│               └── CLAUDE.md
```

下記の通り、GeneralAffairs/ にあるCLAUDE.mdを、GeneralAffairs/GeneralServicesから起動したClaude Codeからちゃんと読めています。

```console
~/repositories/ClaudeCodeTest/GeneralAffairs$ cat CLAUDE.md
# CLAUDE.md

Say 'Mr. Yamaguchi' when asked about the boss of the department.

Reply "General Affairs" when asked about the name of the department.
~/repositories/ClaudeCodeTest/GeneralAffairs$ echo "Who is Mr.Yamaguchi?" | yolo -p
Mr. Yamaguchi is the boss of the General Affairs department.
~/repositories/ClaudeCodeTest/GeneralAffairs$ cd GeneralServices/
~/repositories/ClaudeCodeTest/GeneralAffairs/GeneralServices$ echo "Who is Mr.Yamaguchi?" | yolo -p
Mr. Yamaguchi is the boss of the General Affairs department.
~/repositories/ClaudeCodeTest/GeneralAffairs/GeneralServices$
```

より深いサブディレクトリにCLAUDE.mdを配置する気なら、親ディレクトリのCLAUDE.mdにあまり広範な内容を詰め込まないほうが良いでしょう。


## 兄弟階層のCLAUDE.mdは読まれない

このような状態になっているとしまして

```console
.
├── CLAUDE.md
├── GeneralAffairs
│   ├── CLAUDE.md
├── Sales
│   └── CLAUDE.md
```


Sales/ で起動したClaude Codeは、GeneralAffairs/CLAUDE.mdを読めません。兄弟階層のCLAUDE.mdは読まれないのです。

```
/mnt/repositories/ClaudeCodeTest/GeneralAffairs$ echo "Who is Mr.Yamaguchi?" | yolo -p
Mr. Yamaguchi is the boss of the General Affairs department.
/mnt/repositories/ClaudeCodeTest/Sales$ echo "Who is Mr.Yamaguchi?" | yolo -p
I don't have any information about Mr. Yamaguchi in the provided context or project documentation.
/mnt/repositories/ClaudeCodeTest/Sales$

```

これは良いことです。兄弟階層にしておけば、frontend/専用の設定をbackend/から読まれて思わぬ挙動を起こすことがないわけですから。



## 子階層のCLAUDE.mdは「必要に応じ」

子階層のCLAUDE.mdは、オンデマンドで読まれます。言い方を変えると、Claude Codeの判断で読まれたり読まれなかったりします。

```console

/mnt/repositories/ClaudeCodeTest$ echo "add empty file  GeneralAffairs/tmp.md, and who is Mr.Yamaguchi? " | yolo -p
Empty file created at GeneralAffairs/tmp.md.

I don't have any information about Mr. Yamaguchi in the current codebase or context.
/mnt/repositories/ClaudeCodeTest$ echo "add information file GeneralAffairs/yamaguchi.md, and who is Mr.Yamaguchi? " | yolo -p
Created yamaguchi.md in GeneralAffairs/. Mr. Yamaguchi is a Senior Manager in the General Affairs Division at Awesome, Inc., responsible for administrative operations, process optimization, and cross-departmental coordination.
/mnt/repositories/ClaudeCodeTest$

```

これは公式ドキュメントにも記載されています。

> Any child of the directory where you run claude. This is the inverse of the above, and in this case, Claude will pull in CLAUDE.md files on demand when you work with files in child directories

REF: https://www.anthropic.com/engineering/claude-code-best-practices


確実に読んでほしいならサブディレクトリのCLAUDE.mdを読むよう指定するなどの工夫が必要になります。

```console
/mnt/repositories/ClaudeCodeTest$ echo "read GeneralAffairs/CLAUDE.md, and who is Mr.Yamaguchi? " | yolo -p
Based on the GeneralAffairs/CLAUDE.md file, Mr. Yamaguchi is the boss of the General Affairs department.
/mnt/repositories/ClaudeCodeTest$
```


## 場所を問わず読ませたいなら .claude/CLAUDE.md

.claude/CLAUDE.mdに記載した内容は、場所に関係なく読んでくれます。

```console
~$ cat .claude/CLAUDE.md
# SHARED Claude.md

Say "the Medici family" when asked about the major shareholder.

~$ cd /tmp
/tmp$ echo "who is the major shareholder?" | yolo -p
The Medici family.
/tmp$
```

全プロジェクト共通の設定はここに書いておくと良いでしょう。


