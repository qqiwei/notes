#### gdb common commands

[ref](https://www.bilibili.com/video/BV1vQ4y1N7Pv?from=search&seid=18277767980430820717) 

```
# to build
g++ certain_file.cpp -g -o certain_file.out

gdb certain_file.out
(gdb) file certain_file
(gdb) run
(gdb) quit
(gdb) break func_name
(gdb) break line_num
(gdb) continue
(gdb) next
(gdb) step
(gdb) list
(gdb) print variable_name
(gdb) print array_name
(gdb) print func_name
(gdb) info break
(gdb) info register
(gdb) delete break_pointer_num
(gdb) 
```

