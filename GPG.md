# GPG

## 生成
gpg --full-generate-key

## 列出本地存储的所有GPG密钥信息
gpg --list-keys
gpg --list-keys --keyid-format=short
gpg --list-keys --keyid-format=long

- pub其后的是该密钥的公钥特征，包括了密钥的参数（加密算法是rsa，长度为2048，生成于2019-08-04，用途是Signing和Certificating，一年之后过期）以及密钥的ID。
- uid其后的是生成密钥时所输入的个人信息。
- sub其后的则是该密钥的子密钥特征，格式和公钥部分大致相同（E表示用途是Encrypting）。


# 关联GPG公钥与Github账户
gpg --armor --export 93F04D48749C0243

- 在Github web界面个人,设置, 侧边栏的SSH and GPG keys中，新增一个GPG key，内容即是上述命令的输出结果。
- 再次提醒，GPG密钥中个人信息的邮箱部分，必须使用在Github中验证过的邮箱，否则添加GPG key会提示未经验证。

## 利用GPG私钥对Git commit进行签名
首先，需要让Git知道签名所用的GPG密钥ID：
git config --global user.signingkey 93F04D48749C0243

在每次commit的时候，加上-S参数，表示这次提交需要用GPG密钥进行签名：
git commit -S -m "..."

在计算机上的任何本地存储库中默认对所有提交进行签名，请运行
```
git config --global commit.gpgsign true
git config --global tag.gpgSign true
```
## 信任Github的GPG密钥
在Github网页端进行的操作，比如创建仓库。这些commit并没有用我们之前生成的密钥进行签名，而是由Github代为签名了。这样的结果就是，我们本地无法确认这些签名的真实性。
为了解决这个问题，我们需要导入并信任Github所用的GPG密钥。
导入
```
curl https://github.com/web-flow.gpg | gpg --import
# curl's output is omitted
gpg: key 4AEE18F83AFDEB23: public key "GitHub (web-flow commit signing) <noreply@github.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```
## 信任（用自己的密钥为其签名验证，需要输入密码）：
gpg --sign-key 4AEE18F83AFDEB23

再尝试查看本地仓库的commit签名信息，则会发现所有的commit签名都已得到验证
git log --show-signature



## 编辑信任
```
$ gpg --edit-key 998088DC253D39817158F9A293F04D48749C0243
trust
5
yes
quit
```
## VSCode设置，

转到首选项 > 设置，然后搜索git.enableCommitSigning. 打开此设置


## 更改密码
```
$ gpg --edit-key 998088DC253D39817158F9A293F04D48749C0243
password
输入旧密码
输入新密码
确认新密码
quit
```
## 列出私钥
gpg -K 或 --list-secret-keys : 查看私钥，参数后面没有指定私钥，则输出所有私钥

## 列出公钥
gpg -k 或 --list-public-keys : 查看公钥，参数后面没有指定公钥，则输出所有公钥

## 导出公钥
```
gpg -a -o public-file.key --export keyId 
导出公钥keyId 到 文件 public-file.key中；
    其中：
    -a 为 --armor 的简写，表示密钥以ASCII的形式输出，默认以二进制的形式输出；
    -o 为 --output 的简写，指定写入的文件；
```
## 导出私钥
```
 gpg -a -o private-file.key --export-secret-keys keyId 
 导出私钥 keyId 到文件 private-file.key中，导出的时候需要输入密钥密码；
```

## 导入公钥/私钥
    gpg --import public-file.key / private-file.key 
    导入公钥或私钥，其中，导入私钥需要输入保护私钥的密码；
## 删除公钥
    gpg --delete-keys keyId : 删除公钥；
## 删除私钥
    gpg --delete-secret-keys : 删除私钥；