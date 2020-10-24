# Command-line options


sed 는 명령 라인에서 스크립트 스트링이나 `-e`, `-f` 옵션을 사용하지 않으면 인수로 사용된 파일명 중에서 
첫번째 파일을 명령 스크립트 파일로 그리고 나머지 파일들을 데이터 파일로 취급합니다.
아무런 데이터 파일도 인수로 주지 않을 경우는 stdin 에서 입력을 받습니다.



#### -n, --quiet, --silent

sed 의 디폴트 출력 모드인 자동 출력 모드를 disable 합니다.

#### -e script, --expression=script

스크립트가 길어질 경우 가독성을 위해 `-e` 옵션을 이용해 분리해 작성할 수 있습니다.
shell 에서 명령문을 작성할때 파이프로 sed 명령이 중복되어 연결되어 있다면
`-e` 옵션을 이용해 한번의 명령 실행으로 바꿀수 있습니다.
또한 insert, append, execute 같은 명령을 사용할 때 필요한 옵션입니다.

#### -f script-file, --file=script-file

실행할 스크립트 파일을 지정합니다.

####  -E, -r, --regexp-extended

Extended Regular Expressions 을 사용합니다.


#### -u, --unbuffered

입, 출력에 버퍼를 사용하지 않습니다.
`tail -f` 같은 명령과 함께 사용할때 필요한 옵션 입니다.


####  -s, --separate

sed 는 기본적으로 인수로 전달되는 데이터 파일들을 모두 합쳐서 하나의 스트림으로 취급합니다. 
그러므로 10 개의 데이터 파일이 인수로 전달되어도 `1p` 명령은 한번만 출력됩니다.
하지만 `-s` 옵션을 사용하면 각각 분리된 스트림으로 취급하여 `1p` 명령은 10 번 출력됩니다.


####  -z, --null-data

라인 구분자로 newline 대신에 NUL 문자를 사용합니다.
특정 명령에서 라인 구분자를 NUL 문자로 하여 출력할 경우에 사용되는데
이외에도 파일 전체를 한번에 pattern space 로 읽어들이는 용도로 사용할 수 있습니다.


####  -l *N*, --line-length=*N*

`l` 명령은 pattern space 의 내용을 분석할 때 사용하는 명령인데요.
기본적으로 escape sequence 형태로 출력하므로 개행을 하지 않습니다.
이 옵션을 이용하면 line-wrap length 를 지정할 수 있습니다.
default 는 70 이고 length 값을 0 으로 설정하면 no wrap 입니다.



####  -i[*SUFFIX*], --in-place[*=SUFFIX*]


이 옵션은 sed 를 이용해 파일 내용을 직접 수정할 때 사용합니다.  
특히 디렉토리를 recursive 하게 검색해서 매칭 되는 파일들을 모두 수정할 때 편리합니다.  

처리되는 순서는 다음과 같습니다.

1. 먼저 임시 파일을 만들어서 처리중에 결과를 저장합니다.

2. 처리가 완료되면 임시 파일을 원본 파일명으로 변경하고 원본 파일은 삭제합니다.

3. 이때 SUFFIX 값이 주어졌으면 원본 파일을 삭제하지 않고 SUFFIX 를 붙여서 보관합니다.

그러므로 용량이 큰 파일일 경우 저장공간이 처리하고자 하는 파일의 2배가 필요하고
처리가 완료된 파일은 새로운 inode 를 갖는 파일이 됩니다.

```bash
$ ls
data1  data2  data3

# SUFFIX 는 -i 옵션에 공백 없이 붙인다.
$ sed -i.orig 's/XXX/YYY/' * 

$ ls
data1  data1.orig  data2  data2.orig  data3  data3.orig
-------------------------------------------------------

# ./dir 디렉토리 이하 모든 파일을 검색해서 'hello' 를 포함하고 있을 경우 foo 를 bar 로 변경
$ grep -rl 'hello' ./dir | xargs sed -i.orig 's/foo/bar/g'
```

>옵션에 SUFFIX 를 사용할 때는 공백 없이 붙여야 합니다.  
>만약에 `-i`, `-r` 옵션 사용을 `-ir` 와같이 한다면 `r` 이 SUFFIX 가 되므로 주의해야 합니다.



#### --follow-symlinks

`-i` 옵션을 사용하여 직접 파일을 저장할 경우 대상 파일이 symbolic link 이면 
처리되는 방식에 차이가 있습니다.
다음은 `foo` 가 파일 `file.txt` 의 symbolic link 일 경우 --follow-symlinks 옵션을
사용 안 했을 때와 했을 때의 차이를 비교한 것입니다.

```bash
$ cat file.txt 
1111111111

$ ls -l        
total 4    # foo 는 file.txt 의 symbolic link
-rw-rw-r-- 1 mug896 mug896 11 2020-03-21 10:40 file.txt
lrwxrwxrwx 1 mug896 mug896  8 2020-03-21 10:42 foo -> file.txt
..............................................................

$ sed -i 's/1/22/g' foo 

$ ls -l
total 8    # foo symbolic link 가 일반 파일로 변경된다.
-rw-rw-r-- 1 mug896 mug896 11 2020-03-21 10:40 file.txt
-rw-rw-r-- 1 mug896 mug896 21 2020-03-21 10:44 foo

$ cat foo
22222222222222222222
$ cat file.txt
1111111111

# 다음은 --follow-symlinks 옵션을 사용했을 경우
$ sed --follow-symlinks -i 's/1/22/g' foo

$ ls -l
total 4    # foo symbolic link 는 그대로이고 연결된 file.txt 가 변경된다.
-rw-rw-r-- 1 mug896 mug896 21 2020-03-21 10:45 file.txt
lrwxrwxrwx 1 mug896 mug896  8 2020-03-21 10:45 foo -> file.txt

$ cat file.txt
22222222222222222222

############### 다음은 -i 옵션에 suffix 를 사용했을 경우 #############

$ sed -i.bak 's/1/22/g' foo

$ ls -l     # sed 명령의 처리 결과가 foo 파일로 저장되고 foo.bak 은 symbolic link 가 된다.
-rw-rw-r--  1 mug896 mug896     11 2018-06-21 23:20 file.txt
-rw-rw-r--  1 mug896 mug896     21 2018-07-14 15:36 foo
lrwxrwxrwx  1 mug896 mug896      8 2018-07-14 15:35 foo.bak -> file.txt

# 다음은 --follow-symlinks 옵션을 사용했을 경우
$ sed -i.bak --follow-symlinks 's/1/22/g' foo

$ ls -l     # foo symbolic link 는 그대로이고 연결된 file.txt 가 변경된다.
-rw-rw-r--  1 mug896 mug896     21 2018-07-14 16:06 file.txt
-rw-rw-r--  1 mug896 mug896     11 2018-07-14 15:44 file.txt.bak
lrwxrwxrwx  1 mug896 mug896      8 2018-07-14 16:04 foo -> file.txt

$ cat file.txt
22222222222222222222
$ cat file.txt.bak
1111111111
```



#### -b, --binary

MS-windows 에서는 unix 와 다르게 개행문자로 `\r\n` 를 사용하는데요.
windows 에서 sed 명령이 실행될 때는 내부적으로 `\r\n` 를 `\n` 로 변환하여 읽어들이고
처리 후에는 출력시에 다시 `\n` 를 `\r\n` 로 변환하여 출력합니다.
--binary 옵션은 이렇게 입,출력 때 적용되는 변환을 금지할 때 사용됩니다.
