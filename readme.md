# cheatsheet-parallel

## options

```
-j N              : number of jobs
--jobs 0          : run as many jobs as possible
--no-run-if-empty : do not run if input is empty
--colsep '\t'     : column separator
--xargs           : fit on a single line
--xargs -s 10000  : limit command length
-N 3              : number of arguments
--dryrun          : show command
-q                : quiet
-k, --keep-order  : keep order
--shuf            : shuffle job order
--eta             : show estimated time
--progress        : show progress
--bar             : show progress bar
```

## run command

### no commands means arguments are commands

```
parallel ::: 'echo "test"'
echo 'echo "test"' | parallel
```

### run function

```
myfunc() {
  echo "$1"
}
export -f myfunc
parallel myfunc ::: 1 2 3
```

### quote command

```
parallel -q perl -e 'print "@ARGV\n"' ::: a b c
parallel perl -e \''print "@ARGV\n"'\' ::: a b c
```
Arguments passed to commands through parallel are expanded by the shell twice:
once in the invocation of parallel, and once when parallel runs your command. -q prevents the second shell expansion.
This example need -1 because of the space in 'print "@ARGV\n"'


### delay

```
parallel --delay 2.5 echo Starting {}\;date ::: 1 2 3
```

## parameters

```sh
parallel echo ::: a b c                       # input
parallel echo ::: a b c ::: 1 2 3             # multiple inputs
parallel echo :::: input.txt                  # input from file
seq 3 | parallel echo                         # input from stdin
seq 3 | parallel -a - echo ::: a b c          # input from stdin and parameter
echo input.txt | parallel echo :::: -         # input from stdin as file
parallel --link echo ::: a b c ::: 1 2 3      # link input
parallel echo ::: a b c :::+ 1 2 3 ::: x y z  # link a b c with 1 2 3
parallel --tag echo foo-{} ::: A B C          # prefixed with argument
parallel --tagstring {}-bar echo {} ::: A B C # prefixed with argument
```

## replace

```sh
parallel echo {} ::: a b c                                                      # default replace
parallel echo {.} ::: A/B.C                                                     # remove extension: A/B
parallel echo {/} ::: A/B.C                                                     # remove path: B.C
parallel echo {//} ::: A/B.C                                                    # only path: A
parallel echo {/.} ::: A/B.C                                                    # remove extension and path: B
parallel --plus echo {+.} ::: A/B.C                                             # extension: C
parallel --plus echo {/ABC/DEF} ::: /ABC/test.txt                               # change path
parallel --plus echo {%_demo} ::: demo_test_demo_demo                           # remove suffix
parallel --plus echo {#demo_} ::: demo_test_demo_demo                           # remove prefix
parallel --plus echo {/_demo/} ::: test_demo_demo                               # remove first
parallel echo {#} ::: a b c                                                     # job number
parallel -j 2 echo {%} ::: a b c                                                # slot number
parallel -I ,, echo ,, ::: a b c                                                # replace string
parallel echo {1} and {2} ::: A B ::: C D                                       # positional replace
parallel echo /={1/} //={1//} /.={1/.} .={1.} ::: A/B.C D/E.F                   # replace with positional
parallel echo 1={1} 2={2} 3={3} -1={-1} -2={-2} -3={-3} ::: A B ::: C D ::: E F # position replace from behind
parallel --trim lr echo pre-{}-post ::: ' A '                                   # trim
parallel echo Job {#} of {= '$_=total_jobs()' =} ::: {1..5}                     # total jobs
parallel echo {} shell quoted is {= '$_=Q($_)' =} ::: '*/!#$'                   # shell quoted
parallel echo {= 'if($_==3) { skip() }' =} ::: {1..5}                           # skip
parallel echo {= 'if($arg[1]==$arg[2]) { skip() }' =} ::: {1..3} ::: {1..3}     # argument skip
echo 'test' | parallel 'echo {} >&3' 3> >(cat)                                  # redirect
```

## one-liner

##### rename files with numbers

```sh
ls | parallel --plus -j 4 -k mv {} {#}.{+.}
```

##### convert webp to png

```sh
parallel convert {} {.}.png ::: *.webp
```

##### count files in directories

```sh
ls | parallel 'echo -n {}" "; ls {}|wc -l'
ls | parallel '(echo -n {}" "; ls {}|wc -l) >{}.dir' # save to file
```

##### check files

```sh
cat file_list | parallel 'if [ ! -e {} ] ; then echo {}; fi'
```

##### mv files to first letter directory a-file -> a/a-file

```sh
parallel 'mkdir -p {=s/(.).*/$1/=}; mv {} {=s/(.).*/$1/=}' ::: *
```

##### remove extension

```sh
parallel --plus echo '{%.gz}' ::: foo.tar.gz
```

##### remove two extensions: foo.tar.gz -> foo

```sh
parallel echo '{= s:\.[^.]+$::;s:\.[^.]+$::; =}' ::: foo.tar.gz
```

#### find missing 00:00, 00:05, 00:10 ... 23:55

```sh
parallel [ -f {1}:{2} ] "||" echo {1}:{2} does not exist ::: {00..23} ::: {00..55..5}
```

##### csv

```
cat <<'EOF' | parallel --colsep ';' --header : 'echo {=Date s:([0-9]+)/([0-9]+)/([0-9]+):$3/$2/$1:;=} {Name} {Location}'
Date;Name;Location
3/8/1978;"Beeblebrox; Zaphod";"Betelgeuse V"
10/12/1979;"Dent; Arthur";Earth
1/5/1981;Slartibartfast;Magrathea
EOF
```

##### function tester

```
tester() {
  if (eval "$@") >&/dev/null; then
    perl -e 'printf "\033[30;102m[ OK ]\033[0m @ARGV\n"' "$@"
  else
    perl -e 'printf "\033[30;101m[FAIL]\033[0m @ARGV\n"' "$@"
  fi
}
export -f tester
parallel tester my_program ::: arg1 arg2
parallel tester exit ::: 1 0 2 0
```

##### log rotate

log, log.1, log.2, log.3, log.4, log.5, log.6, log.7, log.8, log.9
-> log.1, log.2, log.3, log.4, log.5, log.6, log.7, log.8, log.9, log.10

```
seq 9 -1 1 | parallel -j1 mv log.{} log.'{= $_++ =}'
mv log log.1
```
