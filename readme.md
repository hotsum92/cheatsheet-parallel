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
parallel --plus echo {/ABC/DEF} ::: /ABC/test.txt                               # change path
parallel echo {#} ::: a b c                                                     # job number
parallel -j 2 echo {%} ::: a b c                                                # slot number
parallel -I ,, echo ,, ::: a b c                                                # replace string
parallel echo {1} and {2} ::: A B ::: C D                                       # positional replace
parallel echo /={1/} //={1//} /.={1/.} .={1.} ::: A/B.C D/E.F                   # replace with positional
parallel echo 1={1} 2={2} 3={3} -1={-1} -2={-2} -3={-3} ::: A B ::: C D ::: E F # position replace from behind
parallel --trim lr echo pre-{}-post ::: ' A '                                   # trim
```
