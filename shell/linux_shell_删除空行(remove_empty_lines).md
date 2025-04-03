
##  命令行
```bash
grep -v '^$' file
sed '/^$/d'  file 或 sed -n '/./p' file
awk '/./ {print}' file 或 awk '{if($0!=" ") print}'
tr -s "n"
```

##  vim交互

```bash
%s/^n//g
```

