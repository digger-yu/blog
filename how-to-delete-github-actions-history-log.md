#1. 查看gh版本
```
gh --version
```
[cli.github.com](https://cli.github.com)

#2. 登录
```
gh auth login

Where do you use GitHub? GitHub.com
What is your preferred protocol for Git operations on this host? HTTPS
Authenticate Git with your GitHub credentials? Yes
How would you like to authenticate GitHub CLI? Login with a web browser

First copy your one-time code: 
```

#3. 删除全部（不管状态）
copy and run in git bash 
```
gh run list --repo digger-yu/lei --limit 500 --json databaseId \
  --jq '.[].databaseId' |
while IFS= read -r rid; do
  [ -z "$rid" ] && continue
  echo "Deleting run #$rid ..."
  gh run delete "$rid" --repo digger-yu/lei
done

echo "✅ Done! 刷新：https://github.com/digger-yu/xxxx/actions"
```

#4
```
# 删全部（前 200 条，不够就再跑一次）
gh run list --limit 200 --json databaseId --jq '.[].databaseId' \
  | xargs -I{} gh run delete {}

# 只删 failed 的
gh run list --status failure --limit 200 --json databaseId --jq '.[].databaseId' \
  | xargs -I{} gh run delete {}
```
