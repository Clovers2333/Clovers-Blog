# MIT Missing Semester

> MIT 开设的入门课（？）看上去啥前置知识都不要会但其实啥都得会一点（x）。主要讲了 shell 的一些命令、常用工具以及一些领域的基本知识，由于在 Linux 虚拟机上操作并且我懒得装中文输入法，所以笔记大部分都是英文的，就当锻（zhe）炼（mo）一下自己吧。

## L1: Course Overview + The Shell

- `echo`：输出命令行参数：用空格分隔参数，不同的参数之间换行；`""` 可以使得 shell 输出带空格的参数。

- 环境变量：`PATH`；在 shell 中输入 `echo $PATH` ，会输出一系列以分号分隔的路径，每次 shell 运行的时候就会去这些 PATH 里面找代码。

  每次运行一条命令时就会遍历文件夹和PATH看看有没有符合的命令或者文件。

  在 Linux 中，可以用 `which <command>` 来查看该命令所在的文件夹。

- Windows 文件路径：用 `\` 反斜杠；Linux/macOS：用 `/` 

  在 Windows 中，每个驱动器都有一个单独的路径结构（C盘、D盘）；而在 Linux 或 macOS 上，所有文件都挂载在一个命名空间下，也叫做根目录。

- 绝对路径（absolute path）：可以精确确定一个文件的位置

  相对路径（relative path）：相对于你当前所在位置的文件位置

  打印当前位置：`pwd`(print working directory)

  `cd`: change directory

  假如当前位置为 ` C:\Users\Clovers\Desktop\code>`，可以通过 `cd \Users` 返回祖先位置；也可以 `cd <absolute directory>`

  `cd .`: 当前目录；`cd ..`：父亲目录

  所以可以看懂 `powershell` 自动补全里面 `cd .\Desktop\code` 是啥意思了；`cd ..\..\aaa`： 返回爷爷目录，进入文件夹 `aaa`。

  类似地，`ls ..` 是展示父亲文件夹中的文件。

- 在 Linux 中：`cd ~` ：返回根目录；`cd -` 返回上一次进入的目录（可以很方便地在两个目录间切换）

- 告诉你命令的参数：`<command> --help`(Linux)；`help <command>`(Windows)

  举例：`ls --help` 出现： `ls [OPTION]... [FILE]...`

  其中，`[]` 表示可选；`...` 表示可以有零个、一个或者多个

- `ls -l`：更详细地显示文件的信息，接下来介绍第一组字符串的阅读方式：（UGO 控制模型）

  前三个为一组是文件所有者设置的权限；第二组三个字符是拥有该文件的组的权限；最后三个字符：其他人的权限。

  三个字符为一组，分别表示是否可读、是否可写、是否可执行。

  （具体阐释：https://blog.csdn.net/hiolb/article/details/104778112）

  - 对于“可读、可写、可执行”的进一步阐释：

    文件和目录的含义不同，对于文件较容易理解；

    对于目录：不可读意味着你看不到目录里面的文件是什么，不能把它们 list 出来；

    对于目录：不可写意味着你不能添加、删除或者重命名目录里面的文件，但是改动文件是可以的。（你可以清空文件，但是不能删除）；

    对于目录：不可执行意味着你不能进入该目录，如果你要 `cd` 这个目录，或者读取这个目录里面的文件，你必须拥有该目录及其所有父目录的执行权限。

    `r-x` 意味着可读、可执行、但是不可写

- `mv`：更改文件路径；即既可以重命名文件，也可以移动文件（其实就是路径，但因为重命名都是在当前文件夹下执行的，所以长得像重命名操作）

- `cp`：将一个文件路径复制到另外一个路径。

- `rm`：删除文件路径，不能删除目录；要用 `rm -r` 进行递归删除，即删除这个路径以后删除这个路径下的所有文件。`rmdir` 可以删除一个空目录（比较安全）；`mkdir`：创建一个空目录（`mkdir Hello world` 会创建两个文件夹！）

- `man <command>`：更详细的 `--help`，更适合阅读。（敲 q 退出）

- `Ctrl+L`：相当于 `clear` 命令。



- 每个程序都有两个流：输入流和输出流，一般情况下，我们在终端输入，然后输出流在终端显示；但是 `shell` 给我们提供了重定向这些流的方法（rewire these streams），改变这些流的指向。

  最简单的方法是 `<` 和 `>`，如 `echo hello > hello.txt`，就是将输出流定向到 `hello.txt` 这个文件中（本质就是：把本来终端要出现的一切东西定向过去 / 把本来要在终端输入的东西从文件里读过来）

  用 `cat hello.txt` 来查看  `hello.txt` 里面的东西；`touch` 创建文件

  如果把 `>` 改成 `>>` 则表示输出流追加而不是覆盖，即在输出的目标文件后面再加上我的输出流。

- `|` ：管道符，将左边程序的输出作为右边程序的输入

  如：`ls -l / | tail -n1`：把根目录所有文件具体信息作为输入，然后执行 `tail` 操作，输出最后一行。

- 有关 `root` 用户：可以干任何事情。大多数情况下，你不会是 `root` 用户；假如你要使用，那么你需要执行 `sudo` 这个程序。

  举例：假如你进入根目录下 `sys` 这个目录，这个目录里保存的不是你计算机内的文件，而是各种内核参数。比如你想通过更改 `brightness` 中的参数来改变屏幕的亮度，然后你运行 `sudo echo 500 > brightness`，发现操作不允许，这其实是因为**输入和输出流不是为程序所知道的**（即 `echo` 不知道 `brightness`）， 即你运行 `sudo` 的程序，参数是 `echo` 和 `500`，但是输出流操作还是基于你当前的 shell，而你的用户不是 `root` ，所以不能写入。

  应该用 `sudo su` 来切换用户变成 `root`，从而开头提示符从 `$ ` 变成了 `#`。

  还有一种方法：`echo 1600 | sudo tee brightness`，其中 `tee`：将输入内容写入一个文件，同时也会在输出流中将输入流输出。

- `xdg-open <file>` 以一个适当的打开方式打开文件（在 Linux 中）



课后练习：

- `!` 要用单引号输入。

- `chmod` 命令：https://zhuanlan.zhihu.com/p/102148141

- 加载器：加载程序的步骤包括，读取可执行文件，将可执行文件的内容写入存储器中，之后开展其他所需的准备工作，准备让可执行文件运行。当加载完成之后，操作系统会将控制权交给加载的代码，让它开始运行。

- 有关 “Shebang”：（https://en.wikipedia.org/wiki/Shebang_(Unix)）

  如以指令 `\#!/bin/sh` 开头的文件在执行时会实际调用*/bin/sh*程序。（`sh` 是程序，`semester` 是参数）

  由于`#`符号在许多脚本语言中都是注释标识符，Shebang的内容会被这些脚本解释器自动忽略。 在`#`字符不是注释标识符的语言中，例如Scheme，解释器也可能忽略以`#!`开头的首行内容，以提供与Shebang的兼容性。

  在练习中，我们发现可以用 `sh` 程序来运行 `curl` 命令，那么当我们 `chmod` 后，正是因为第一句 Shebang 的内容，使得直接运行程序时计算机知道我们要去运行 `sh` 程序。



## L2: Shell Tools and Scripting

- Something about the `$` : get the value that the variable represents.

```shell
> foo=bar
> echo $foo
bar
> echo "value is $foo"
value is bar
> echo 'value is $foo'
value is $foo
# single quotes won't replace the variable.
```

- A simple script:

```shell
mcd(){
	mkdir -p "$1"
	cd "$1"
}
# one should write this script in the /usr/bin or

# $1 is like the "argv" in other languages, which represents the first argument of the command.
# the same as $1, $0 will represents the name of the script.
# $# is the same as "argc", which represents the number of the arguments.
# $$ is the process ID(known as pid), you can see it as a nickname of the current process. A "process" is a function which is running now, such as terminal, chrome, etc.
# $@ is the list of all arguments.

> source mcd.sh
# this command will execute this script in the shell and load it.
```

- Standard error stream:

```shell
> echo "Hello"
> echo $?
# the second line output the error code of the last command, "0" means everingthing went fine.
# note that "true" will always have a "0" error code, while "false" will always have a "1" error code.

> false || echo "Oops fail"
Oops fail.
# "||" means if the first command didn't work, then it will execute the second one.
> false ; echo "this will always print"
this will always print
```

- How to restore the output of a command:

```shell
> foo=$(pwd)
> echo $foo
~/workspace
# "$(function)" means it's a variable, it will get it's value when being executed.
```

- Something about the "process substitution", which works out similarly as the "command substitution" above.

```shell
> cat <(ls) <(ls ..)
# `<()` will put the output of the command into a temporary file, and the fuction "cat" will print the two files.
# the fuction "cat" get input from the file instead of stdin.
> diff <(ls foo) <(ls bar)
```

​	Btw, let us distinguish the difference between "command" and "function": "command" means the whole line in the shell, while "function" is just a word.

- A specific example of a script:

```shell
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done

# "2>" redirects the stderr.
# the command "grep": find the string in the test and print the line containing the string.
# the stderr of "grep": if finding success, the stderr will be 0, otherwise be 1.
# in the loop, the file starts at $1.
```

- `ls *.sh` : all files with the suffix `.sh`; `ls project?` : `?` will expand to only a single character.
- How to use brace `{}`:

```shell
> convert image.png image.jpg
> convert image.{png,jpg}
# the above two command are the same, which means the brace will expand the suffix.

> touch foo{,1,2,10}
> touch foo foo1 foo2 foo10

> touch project{1,2}/src/test/test{1,2,3}.py
# it will do the Cartesian product, which will generate 6 `.py` files.
```

- Shebang:

```shell
#!/usr/bin/env python
# use the /usr/bin/env to interpret the file with the argument python.
```

- find:

```shell
> find . -name src -type d
# `.`: the current directory; `d`: you want to find a directory.
```



### [Exercises](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions/shell-tools-solution/)

- sort `ls` by recency: `ls -t`

- ```bash
  varName="Aloha"
  echo "varName is not empty: the two constructs behave the same"
  echo ${varName-Hello World}
  echo ${varName:-Hello World}
  echo ""
  unset varName
  echo "varName is unset: they still behave the same way"
  echo ${varName-Hello World}
  echo ${varName:-Hello World}
  echo ""
  varName=""
  echo "varName is empty: this is where the two differs"
  echo ${varName-Hello World}
  echo ${varName:-Hello World}
  echo ""
  # Basically `:-` changes the values of the variable if it's empty or not set, and `:` changes it if it's not set.	
  # use "" to escape the influence of `space`
  ```

- **What is `&&` in Bash?** In Bash — and many other programming languages — `&&` means “AND”. And in command execution context like this, it means items to the left as well as right of `&&` should be run in sequence in this case.

  **What is `&` in Bash?** And a single `&` means that the preceding commands — to the immediate left of the `&` — should simply be run in the background.

- **There is a problem in exercise 3 that why the other codes don't work except the `until` code?**

- ```bash
  $ zip [options] <zipfile> <file1> <file2> <...> # compress files
  $ 7z x <zipfile> # extract zipfile
  ```

- `xargs -d 'char'`: let the input become the arguments of the command and separate the inputs by the char(such as `\n`)



## L3: Vim

-----Just to introduce some basic operation.

### Normal mode

- `/<word>`: to find the word, and press `n` to find the next matching word.
- `u`: undo; 
- `^r`=cancel the undo
- `hjkl`: left, down, up, right
- b: move back the word
- w: move forward the word
- `gg`: move to the first line
- `G`: move to the last line
- `^`: first non-empty character
- `$`: the end of the line
- `0`: the begin of the line
- `^d`: next paragraph
- `^u`: previous paragraph
- `dd`: delete the line
- `r<character>`: change the current character on the cursor

### Command mode(`:`)

`:w`: save

`:q`: quit

`:q!`: quit without saving

`:sp`: split the buffer

### Visual mode(v)

- When you go into the visual mode, your current cursor will set to the beginning cursor, and you can move your cursor to set the ending cursor.
- `y`: copy; `p`: paste
- When pressing the above two bottom, it will go bake to the normal mode.



## L4: Data Wrangling

- `less <file>`: To view the file in a more pure environment.
- `sed <regular expression> < <file>`: Use the regular expression to do data wrangling.

### Basic grammar of Regular Expressions

- `.` means “any single character” except newline

- `*` zero or more of the preceding match(for the last character)

- `+` one or more of the preceding match

- `[abc]` any one character of `a`, `b`, and `c`

- `(RX1|RX2)` either something that matches `RX1` or `RX2`

- `^` the start of the line

- `$` the end of the line

- `(<string>)?`: the string appears zero or one time.

- `\d`: matches digits.

- `\w` for alphanumeric letters and digits(word character),` \w` is equivalent to `[a-zA-Z_0-9]`.

- `\s`: white space.

- `\D`: non-digit character.

- `\S`: any non-whitespace character(A “word character” is a character that can be used to form words.)

- `\b`: matches the boundary between a word and a non-word character.(It matches at a position that is called a “word boundary”. This match is zero-length.)

  There are three different positions that qualify as word boundaries:

  - Before the first character in the string, if the first character is a word character.
  - After the last character in the string, if the last character is a word character.
  - Between two characters in the string, where one is a word character and the other is not a word character.

  So, you can capture an entire word by `\w+\b`.

  I think it is used to capture words.

```shell
> echo "aba" | sed -E 's/[ab]//'
ba
```

Using the regular expressions, it will just match the pattern once and apply the replacement once per line.

To do as many matches as possible: `sed 's/[ab]//g'`

- `-E` to use the more modern version of `sed`. In this version, brackets with `\` will represent the origin brackets, while brackets without `\` will represent their grammar meanings.

- `()` represents a capture group, and `\1` means the first capture group, etc.

Here is an example:

```shell
# the pattern of each line in ssh.log:
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Clovers 46.97.239.16 port 55920 [preauth]


> sed -E 's/^.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' ssh.log
# add `^` and `$` to prevent some unpredictable problems.

# the output:
Clovers
```

- A handy tool to debug the regular expressions: https://regex101.com/r/qqbZqh/2
- regex exercises: https://regexone.com/lesson/

However, if the user name is the entire line of the log, then it will cause problems because the beginning `.*` is greedy matching, you can add a `?` after `.*` to make it non-greedy matching. ( The moment we get the word `Disconnected from`, we start matching. )

- In regular situations, regular expressions will not make cross-line matching; the way that `sed` works is doing matching for each line.



### Other functions to do Data Wrangling

​	----`sed` is a terrible programming language except for searching and replacing.

- `wc -l <file>`: `wc` is the word counting function, and the option `-l` is to let it count the number of lines of the file.

- `<file> | sort | uniq`: sort the lines of the file and make the lines unique.

  - `uniq -c`: count the number of appearance of the same line.

  - `sort -n`: sort by number; `sort -k`: select a white space separated column to do sorting

    `sort -nk1,1`: sort start at the first column and end at the first column.

    In this case, we don't need the option `-nk1,1` , we can simply sort by the whole line because the number happens to be at the first.

  - `tail -n5`: to print last 5 lines; `head -n5`: to print the first 5 lines.

- `awk`: a column-based stream processor.

  - `awk '$1 == 1 && $2 ~ /^c.*e$/ { print $0 }'`: `$1` represents the first column, `$0` represents the whole line. It will print every line whose user name starts with `c` and ends with `e`.

  - `{}` is the command that you want to execute when the condition in front of it matched.

    ```shell
    > awk'BEGIN { rows = 0 }
      $1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
      END { print rows }'
    ```

    In the above case, `BEGIN` represents the function starts processing, when the functions starts, the variable `rows` will be set to `0`. 

    The usage of the command is to count the line whose user name starts with `c` and ends with `e`.

  - `awk '$1 == 1 && $2 ~ /^c.*e$/ { print $0 } | paste -sd,`: write the output in one line and use `,` to separate each word.

- `bc`: a simple command-line calculator.

  `bc -l <file>`: If the file is a simple expression, and by `bc` function, it will help to work out the answer.

  ```shell
  awk '$1 > 1 {print $1}' | paste -sd+ | bc -l
  ```

- `xargs`: takes the lines of the input and turns them into arguments.

  ```shell
  rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
  ```

  `xargs` will let the stdin be the argument of the function `rustup toolchain uninstall`

### Exercises

1. [regex exercises](https://regexone.com/lesson)

2. ```shell
   $ cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | wc -l
   ```

   - 大小写转换：`tr "[:upper:]" "[:lower:]"`(the same as `tr A-Z a-z`)

   - `^([^a]*a){3}.*[^'s]$`：查找一个以 a 结尾的字符串三次

   - `grep -v "\'s$"`: 匹配结尾为’s 的结果，然后取反。 借助 `grep -v`.

   - 这些单词中，出现频率前三的末尾两个字母是什么？ `sed`的 `y`命令，或者 `tr` 程序也许可以帮你解决大小写的问题。

     ```shell
     $ cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq -c | sort | tail -n3
     # 53 as
     # 64 ns
     # 102 an
     ```

   - 共存在多少种词尾两字母组合？

     ```shell
     $ cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq | wc -l
     ```

   - 还有一个很有挑战性的问题：哪个组合从未出现过？ 为了得到没出现的组合，首先我们要生成一个包含全部组合的列表，然后再使用上面得到的出现的组合，比较二者不同即可。

     ```shell
     #!/bin/bash
     for i in {a..z};do
      for j in {a..z};do
         echo  "$i$j"
      done
     done
     # all.sh
     
     ./all.sh > all.txt
     cat /usr/share/dict/words | tr "[:upper:]" "[:lower:]" | grep -E "^([^a]*a){3}.*$" | grep -v "'s$" | sed -E "s/.*([a-z]{2})$/\1/" | sort | uniq > occurance.txt
     diff --unchanged-group-format='' <(cat occurance.txt) <(cat all.txt) | wc -l
     ```

     `--unchanged-group-format=''`用于将两个文件中相同的内容设置为空字符串，剩下的内容就是差异的部分。

3. To do in-place substitution it is quite tempting to do something like `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. However this is a bad idea, because in this command, it will clear `input.txt` first.

   use: `sed -i.bak s/REGEX/SUBSTITUTION/ input.txt`, and the original file will be changed, and it will generate a `.bac` file to backup previous data.

4. skipped

5. skipped

6. Too hard! Copy the solution first.

   在网上找一个类似 [这个](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm) 或者[这个](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1)的数据集。或者从[这里](https://www.springboard.com/blog/free-public-data-sets-data-science-project/)找一些。使用 `curl` 获取数据集并提取其中两列数据，如果您想要获取的是HTML数据，那么[`pup`](https://github.com/EricChiang/pup)可能会更有帮助。对于JSON类型的数据，可以试试[`jq`](https://stedolan.github.io/jq/)。请使用一条指令来找出其中一列的最大值和最小值，用另外一条指令计算两列之间差的总和。

   ```shell
   ~$ curl 'https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm#wikipedians' \
       |sed -n "/table1/,/<\/table>/p" \
       |grep "<tr" | sed "1,12d"|head -n -3 \
       |sed -E 's/(<[^>]*>)+/ /g' \
       |sed 's/ &nbsp;/ -/g' \
       |sed 's/&nbsp;//g' > data
      
   ~$ cat data # 处理后的数据为Jan2001截至Oct2018的
   Oct2018 2642056 12641 70805 10498 48.9M - 6101 - - - - 10.3M - - - - - - 42.6M 
   Sep2018 2629415 11171 66574 10004 48.7M - 6116 - - - - 10.1M - - - - - - 42.4M 
   Aug2018 2618244 12058 68688 10640 48.5M - 6839 - - - - 10.2M - - - - - - 42.1M 
   Jul2018 2606186 12026 68037 10305 48.3M - 6987 - - - - 9.5M - - - - - - 41.9M 
   ...
   Jan2001 7 7 9 - 31 12 1 8.6 1352 29% 10% 267 301kB 3.0k 15 - - 2 163 
   ```

   命令说明（建议先查看网站的源码格式，更加容易理解下面的操作）

   - `|sed -n "/table1/,/<\/table>/p"`：观察网站的源代码，可以发现第一个表格的id为`table1`，该命令将截取匹配到的`table1`以及下一个`</table>`标签行之间的内容

   - `|grep "<tr"`：html表格中，含有数据的行是以`<tr`开头，匹配这样的行

   - `|sed "1,12d"`：去掉前12行（包含表格的表头）

   - `|head -n -3`：去掉最后3行（包含非数据的内容）（注意：部分操作系统可能不支持该用法，最笨拙的替换实现方式是：`|sed "$d"|sed "$d"|sed "$d"`，即执行3次删除最后一行的操作）

   - `|sed -E 's/(<[^>]*>)+/ /g'`：使用正则匹配，将所有相邻的多个html标签（格式行如`< tag >`）替换为空格

   - `|sed 's/  / -/g`：原表格中部分没有数据的单元格是以` `填充的，将其替换为` -`，避免在对数据操作时发生窜列的情况

   - `|sed 's/ //g`：原表格中部分单元格内的空格也是用` `表示的，将其全部删除（不影响数据处理）

     ```shell
     ~$ awk '{print $1,$4,$5}' data | sort --key=2n | head -n 1
     Jan2001 9 -
     # 从data中读取第一列（时间，用来定位后续结果）及第三、四列，并以第二行的数据以数字大小进行排序，然后显示最大值的结果；下一个命令显示最小值的结果
     
     ~$ awk '{print $1,$4,$5}' data | sort --key=2n | tail -n 1
     Mar2007 91388 11506
     
     ~$ awk '{print $1,$4,$5}' data | awk '{print $2-$3}' | awk '{s+=$1} END {print s}'
     10153001
     # 使用第二列的数据减去第三列的数据后，将结果加总
     ```



## L5: Command-line Environment

### Job Control

- use `man signal` to view the different signal that the computer send to do with the process.

- `nohup <command>` means you want to run a command to immune hangups.

  - `&` means you want the run the process even it is hung up, you can do other things when the process is running.

  - ` bg %<num>` means you want to continue specific process on the background.
  - `fg %<num>` means you want to put the process on the background to the front ground.

  ```shell
  $ sleep 1000
  ^Z
  [1]  + 18653 suspended  sleep 1000
  
  $ nohup sleep 2000 &
  [2] 18745
  appending output to nohup.out
  
  $ jobs
  [1]  + suspended  sleep 1000
  [2]  - running    nohup sleep 2000
  
  $ bg %1
  [1]  - 18653 continued  sleep 1000
  
  $ jobs
  [1]  - running    sleep 1000
  [2]  + running    nohup sleep 2000
  
  $ kill -STOP %1
  [1]  + 18653 suspended (signal)  sleep 1000
  
  $ jobs
  [1]  + suspended (signal)  sleep 1000
  [2]  - running    nohup sleep 2000
  
  $ kill -SIGHUP %1
  [1]  + 18653 hangup     sleep 1000
  
  $ jobs
  [2]  + running    nohup sleep 2000
  
  $ kill -SIGHUP %2
  
  $ jobs
  [2]  + running    nohup sleep 2000
  
  $ kill %2
  [2]  + 18745 terminated  nohup sleep 2000
  
  $ jobs
  ```

### Terminal Multiplexer(tmux)

​	------ use a single terminal to do multiple process

- A specific introduction of tmux: https://blog.csdn.net/qq_62839589/article/details/132067867

- The basic structure of tmux:

  |--session

    |--windows

  ​    |--panes

  - ​	"windows" is like the "tabs" in chrome.

- When we write `tmux` in the terminal, then it will start another process separated from the original process.

### Dotfiles

- `alias`: make a long command to be a short one

```shell
> alias alias_name="command_to_alias arg1 arg2"
# there must be no space around `=`

> alias alias_name
alias_name="command_to_alias arg1 arg2"
```

Note that aliases do not persist shell sessions by default. To make an alias persistent you need to include it in shell startup files, like `.bashrc` or `.zshrc`.

- edit the `.zshrc` or `.bashrc` to initialize the settings.

- symlinks: tell the system that `~/dotfiles/.zshrc` is actually `~/.zshrc` it wants, and the folder can be easily backed up in github.

  ```shell
  ln -s path/file symlink
  # symlink is ".zshrc"
  ```

### Remote Machines

- The main command for working with remote machines is `ssh`.
- `ssh` is like a safe shell, which is gonna take responsibility for reaching whoever we want.
- `ssh <username@ip> <command>`: to do remote commands on another computer.
- How to avoid typing the password every time we use `ssh` ? ------- `ssh keys` (public key encryption)
  - give the server the public key.
  - the system will use the private key to prove your identity.
  - generate:`ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519`
  - If you have ever configured pushing to GitHub using SSH keys, then you have probably done the steps outlined [here](https://help.github.com/articles/connecting-to-github-with-ssh/) and have a valid key pair already. To check if you have a passphrase and validate it you can run `ssh-keygen -y -f /path/to/key`.
- `scp <file> <username@ip:path>`: copy the file to the remote machine.
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) improves upon `scp` by detecting identical files in local and remote, and preventing copying them again.
- you can edit `.ssh/config` to avoid input a long command every time.



## L6: Version Control(Git)

- The basic structure of Git:

```pseudocode
type blob=array<byte>  # blob refers to file
type tree=map<string, tree | blob>  # map from name to things(folder)
type commit=struct{ # like a snapshot
	parents: array<commit>
	author: string
	message: string
	snapshot: tree  # actual contents
}
type object=blob | tree | commit
objects=map<string,object>  
# simplify the object and store them by address(use hash functions)
# you can recognize that all the object are actually pointers, and "objects" is actually the whole memorable space.(C#)
```

- The hash function `SHA-1`: a 160-bit hash 
  - hash the object to 40 chars of `hex`
  - construct `map<string, string>`, map the address to a human-readable name, you can recognize the name as a `pointer` variable.
- The nodes of the tree can't be deleted but can be added.
- `ls -a`: see the hidden files and folders with the prefix `.`

### Some commands:

```shell
> git init: to create `.git` in the folder
> git status: what is the status of the current repository
> git add <files> / git add .
> git commit -m "commit-message"
> git log --all --graph --decorate: visualizes history as a DAG
# I will alias it as "gitlog"
> git checkout address: enter different nodes (change pointer `HEAD`)
> git checkout <file>: throw away the changes in the working directory and set it back as the version of `HEAD` 
> git diff <file>: find differences with the last snapshot (known as `HEAD`)
> git diff pointer/address <file>: find differences with specific snapshot
> git diff address1 address2 <file>: find differences between two snapshot
```

- `git add` means you add specific files to the staging area, and `git commit` will commit the changes to the final area, while `git push` is let Github know your changes. 
- `master` usually points to the main branch or the up-to-date version of your project.
- `HEAD` points to what you are looking right now.

### Branching and Merging

```shell
> git branch: list all the branches that are presented in the local directory
> git branch <name>: creates a branch (its parent is `HEAD`)
> git checkout -b <name>: creates a branch and switches to it
# same as git branch <name>; git checkout <name>
> git merge <revision>: merges into current branch
> git merge --abort: undo the last "git merge"
> git mergetool: use a fancy tool to help resolve merge conflicts
> git rebase: rebase set of patches onto a new base
> git branch -vv: tell all the branches
```

- Git not only knows which snapshot we're currently looking at, but also aware of what branches associated with the snapshot.
- When I use `git commit`, the current branch will be used to create a new snapshot, while another branches(pointers) will remain unchanged.
- `master` branch is actually the main workflow, and the most things you do is to fork from the `master` branch and merge different branches back to `master` branch.
- When merging, if the `master` branch is a strictly older thing, then it will display "Fast-Forward", and directly merge the branches.
- `Merge Conflict`: vim the conflict file, polish the file manually, and use `git merge --continue`.

### Git Remotes

```shell
> git remote: list remotes
> git remote add <name> <url>: add a remote and name it.
> git push <remote> <local branch>:<remote branch>: send objects to remote, and update remote reference
> git branch --set-upstream-to=<remote>/<remote branch>: set up correspondence between local and remote branch
# then I can simply use `git push`(In github, it has already be done)

> git fetch: retrieve objects/references from a remote
> git pull: same as git fetch; git merge
# git is very smart, it will always keep the latest version, do you don't have to worry that this command will cover your local changes.
> git clone: download repository from remote
# git clone will clone the whole history from remote, and use `git clone --shallow` can just clone the latest snapshot
```

### Other Tools

```shell
> git add -p <file>: interact with the terminal to upload the changes
> git diff --cached: show the changes in the staged area(compared with last snapshot)
> git blame: who edit each line of the file
> git show: get information of specific commit
> git stash: temporarily remove modifications to working directory
# git stash pop: undo
> git bisect: binary search history (e.g. for regressions)
```

- `.gitignore:` specify intentionally untracked files to ignore



## L7: Debugging and Profiling

### Debugging

### Profiling

#### timing

- *Real* - Wall clock elapsed time from start to finish of the program, including the time taken by other processes and time taken while blocked (e.g. waiting for I/O or network)
- *User* - Amount of time spent in the CPU running user code
- *Sys* - Amount of time spent in the CPU running kernel code

#### Profilers

- `python -m cProfile grep.py`
- `...... | tac`: print reversely
- add a decoration in `.py` (`@profile`), and run command: `kernprof -l -v a.py`

#### Memory

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```shell
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```



## L8: Metaprogramming

### Makefile

### Dependency management

- [*semantic versioning*](https://semver.org/)
- If a new release does not change the API, increase the patch version.(API: application programming interface, an interface that you can use the function without knowing inner working theory)
- If you *add* to your API in a backwards-compatible way, increase the minor version.
- If you change the API in a non-backwards-compatible way, increase the major version.

### Continuous integration systems

Continuous integration, or CI, is an umbrella term for “stuff that runs whenever your code changes”.

Pages is a CI action that runs the Jekyll blog software on every push to `master` and makes the built site available on a particular GitHub domain. 

### Testing

Some terms about testing:

- Test suite: a collective term for all the tests
- Unit test: a “micro-test” that tests a specific feature in isolation
- Integration test: a “macro-test” that runs a larger part of the system to check that different feature or components work *together*.
- Regression test: a test that implements a particular pattern that *previously* caused a bug to ensure that the bug does not resurface.
- Mocking: to replace a function, module, or type with a fake implementation to avoid testing unrelated functionality. For example, you might “mock the network” or “mock the disk”.



## L9: Security and Cryptography

### Entrophy

- Entropy is measured in *bits*, and when selecting uniformly at random from a set of possible outcomes, the entropy is equal to `log_2(# of possibilities)`.
- How many bits of entropy is enough? It depends on your threat model. For online guessing, as the XKCD comic points out, ~40 bits of entropy is pretty good. To be resistant to offline guessing, a stronger password would be necessary (e.g. 80 bits, or more).

### Hash functions

A hash function has the following properties:

- Deterministic: the same input always generates the same output.
- Non-invertible: it is hard to find an input `m` such that `hash(m) = h` for some desired output `h`.
- Target collision resistant: given an input `m_1`, it’s hard to find a different input `m_2` such that `hash(m_1) = hash(m_2)`.
- Collision resistant: it’s hard to find two inputs `m_1` and `m_2` such that `hash(m_1) = hash(m_2)` (note that this is a strictly stronger property than target collision resistance).



#### Applications

- Git: In Git, each object will use `SHA-1` function to get their address, and these address will point to the files, but you can not simply restore the object by the address.

  Instead of managing objects, managing addresses will be much easier.

- A short summary of the contents of a file. Software can often be downloaded from (potentially less trustworthy) mirrors, e.g. Linux ISOs, and it would be nice to not have to trust them. The official sites usually post hashes alongside the download links (that point to third-party mirrors), so that the hash can be checked after downloading a file.

- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme). Suppose you want to commit to a particular value, but reveal the value itself later. For example, I want to do a fair coin toss “in my head”, without a trusted shared coin that two parties can see. I could choose a value `r = random()`, and then share `h = sha256(r)`. Then, you could call heads or tails (we’ll agree that even `r` means heads, and odd `r` means tails). After you call, I can reveal my value `r`, and you can confirm that I haven’t cheated by checking `sha256(r)` matches the hash I shared earlier.



### Key derivation functions

A related concept to cryptographic hashes, [key derivation functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs) are used for a number of applications, including producing fixed-length output for use as keys in other cryptographic algorithms. Usually, KDFs are deliberately slow, in order to slow down offline brute-force attacks.

#### Applications

- Producing keys from passphrases for use in other cryptographic algorithms (e.g. symmetric cryptography, see below).
- Storing login credentials. Storing plaintext passwords is bad; the right approach is to generate and store a random [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` for each user, store `KDF(password + salt)`, and verify login attempts by re-computing the KDF given the entered password and the stored salt.

- Why use salt values: 1) avoid rainbow list(store a list of ciphertext generated by common password); 2) many users use same password in different website.



### Symmetric cryptography

```pseudocode
keygen() -> key  (this function is randomized, or use the password as an argument to generate the key--KDF function)
# the key should never be known

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```



### Asymmetric cryptography

```pseudocode
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)
```

- A symmetric cryptosystem is like a door lock: anyone with the key can lock and unlock it. Asymmetric encryption is like a padlock with a key. You could give the unlocked lock to someone (the public key), they could put a message in a box and then put the lock on, and after that, only you could open the lock because you kept the key (the private key).

```pseudocode
sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

- hard to forge a signature without private key.
- everyone can verify(using public key), but can't forge.

#### Applications

- [PGP email encryption](https://en.wikipedia.org/wiki/Pretty_Good_Privacy). People can have their public keys posted online (e.g. in a PGP keyserver, or on [Keybase](https://keybase.io/)). Anyone can send them encrypted email.
- Private messaging. Apps like [Signal](https://signal.org/) and [Keybase](https://keybase.io/) use asymmetric keys to establish private communication channels.
- Signing software. Git can have GPG-signed commits and tags. With a posted public key, anyone can verify the authenticity of downloaded software.
- Symmetric cryptography is much quicker than Asymmetric cryptography, so you can use Symmetric cryptography to encrypt the whole file, use public key to encrypt the key of Symmetric cryptography, and send it with the ciphertext to another.



## L10: potpourri

### Keyboard remapping

- Open a new terminal or browser window.
- Inserting some specific text, e.g. your long email address or your MIT ID number.
- Sleeping the computer or the displays.
- Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap) or [Autokey](https://github.com/autokey/autokey)
- Windows - Builtin in Control Panel, [AutoHotkey](https://www.autohotkey.com/) or [SharpKeys](https://www.randyrants.com/category/sharpkeys/)

### Backups

First, a copy of the data in the same disk is not a backup, because the disk is the single point of failure for all the data. Similarly, an external drive in your home is also a weak backup solution since it could be lost in a fire/robbery/&c. Instead, having an off-site backup is a recommended practice.

Synchronization solutions are not backups. For instance, Dropbox/GDrive are convenient solutions, but when data is erased or corrupted they propagate the change. For the same reason, disk mirroring solutions like RAID are not backups. They don’t help if data gets deleted, corrupted or encrypted by ransomware.

Some core features of good backups solutions are versioning, deduplication and security. Versioning backups ensure that you can access your history of changes and efficiently recover files. Efficient backup solutions use data deduplication to only store incremental changes and reduce the storage overhead. Regarding security, you should ask yourself what someone would need to know/have in order to read your data and, more importantly, to delete all your data and associated backups. Lastly, blindly trusting backups is a terrible idea and you should verify regularly that you can use them to recover data.

Backups go beyond local files in your computer. Given the significant growth of web applications, large amounts of your data are only stored in the cloud. For instance, your webmail, social media photos, music playlists in streaming services or online docs are gone if you lose access to the corresponding accounts. Having an offline copy of this information is the way to go, and you can find online tools that people have built to fetch the data and save it.

For a more detailed explanation, see 2019’s lecture notes on [Backups](https://missing.csail.mit.edu/2019/backups).

### Common command-line flags/patterns

- You can usually use `--version` or `-V` to have the program print its own version (handy for reporting bugs!).
- Almost all tools have a `--verbose` or `-v` flag to produce more verbose output. You can usually include the flag multiple times (`-vvv`) to get *more* verbose output, which can be handy for debugging. Similarly, many tools have a `--quiet` flag for making it only print something on error.
- The special argument `--` makes a program *stop* processing flags and options (things starting with `-`) in what follows, letting you pass things that look like flags without them being interpreted as such: `rm -- -r` or `ssh machine --for-ssh -- foo --for-foo`.

### Notebook programming

[Notebook programming environments](https://en.wikipedia.org/wiki/Notebook_interface) can be really handy for doing certain types of interactive or exploratory development. Perhaps the most popular notebook programming environment today is [Jupyter](https://jupyter.org/), for Python (and several other languages). [Wolfram Mathematica](https://www.wolfram.com/mathematica/) is another notebook programming environment that’s great for doing math-oriented programming.

