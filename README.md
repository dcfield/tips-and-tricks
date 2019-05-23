# tips-and-tricks
Random tips + tricks

## Git tricks
- Count number of author commits
```
git log --author="_Your_Name_Here_" --pretty=tformat: --numstat \
| gawk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s removed lines: %s total lines: %s\n", add, subs, loc }' -
```

- List of git commit counts by author
```
git shortlog -s
```

- Count git commits from author
```
git rev-list HEAD --author="Shing Lyu" --count 
```

- Count git commits from author since date
```
git rev-list HEAD --author="Shing Lu" --count --since="23/05/2019"
```
