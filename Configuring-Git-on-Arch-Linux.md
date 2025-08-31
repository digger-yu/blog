# 1. 安装git 

Git默认使用less作为分页器来显示长输出，当找不到less时会报错
```
sudo pacman -S git less jq
```

# 1. 配置git config

```
git config --list
git config --global user.name "digger yu"
git config --global user.email digger-yu@outlook.com
git config --global user.signingkey 93F04D48749C0243
git config --global log.showSignature true
git config --global commit.gpgsign true
git config --local commit.gpgsign true
git config --global color.ui true
git config --global alias.tbmain '!git fetch --all && git reset --hard origin/main && git pull origin main'
git config --global alias.tbmaster '!git fetch --all && git reset --hard origin/master && git pull origin master'
git config --global alias.debug '!GIT_CURL_VERBOSE=1 GIT_TRACE=1 git'
```

# 2. 导入并信任gpg key
```
gpg --list-keys
gpg --import public-file.key
gpg --import private-file.key

gpg --edit-key 93F04D48749C0243
trust
5
yes

#导入github web-flow 并信任
curl https://github.com/web-flow.gpg | gpg --import

gpg --edit-key 968479A1AFF927E37D1A566BB5690EEEBB952194
trust
4
yes

#验证签名
echo "test" | gpg --clearsign
```
其中有一个是过期的密钥，原因请参考：  
https://github.blog/news-insights/company-news/rotating-credentials-for-github-com-and-new-ghes-patches/
# 3. 删除过期的github密钥，给github的密钥签名

```
gpg --delete-keys 5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23
gpg --sign-key B5690EEEBB952194

gpg --list-keys
```
# 4. add ssh-key
```
copy your public and private key to ~/.ssh     
如果没有则用命令ssh-keygen 创建   
将公钥复制到.ssh   
```
https://github.com/settings/ssh/new

# 4.1 .ssh/config
vi ~/.ssh/config
```
Host github.com
HostName ssh.github.com 
User git
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
GlobalKnownHostsFile ~/.ssh/known_hosts
``` 
配置文件可视情况，增加可选压缩模式，加密算法等
```
# 使用 OpenSSH 的安全延迟压缩模式
Compression yes
CompressionLevel 6  # 可选压缩级别（1-9）
完整配置示例
  Compression no                # 完全禁用（避免CRIME攻击）
  Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
  HostKeyAlgorithms ecdsa-sha2-nistp256,ssh-ed25519
```
# 4.2 .ssh/known_host
将以下 ssh 密钥条目添加到 ~/.ssh/known_hosts 文件中，以避免手动验证 GitHub 主机：

```
curl -L https://api.github.com/meta | jq -r '.ssh_keys | .[]' | sed -e 's/^/github.com /' >> ~/.ssh/known_hosts
```
无jq方案
```
#可以先用以下命令去重（执行前备份）：
#sort -u ~/.ssh/known_hosts -o ~/.ssh/known_hosts
#清理旧的 GitHub 密钥
#sed -i.bak '/^\(github\.com\|gist\.github\.com\)/d' ~/.ssh/known_hosts
# 添加新的 GitHub 密钥
curl -sSL https://api.github.com/meta | \
  awk '
    BEGIN {in_ssh=0}
    /"ssh_keys": \[/{in_ssh=1; next}
    in_ssh && /\]/{in_ssh=0}
    in_ssh && /"/ {
        gsub(/^[[:space:]]+"|",?$/, "")
        print "github.com", $0
        print "gist.github.com", $0
    }
  ' >> ~/.ssh/known_hosts
```
https://docs.github.com/zh/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints

设置权限
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/known_hosts
```
test  -v参数显示详细过程
```
ssh -T git@github.com -v
```
# 4.3 .bashrc
导入gpg密钥之后，如果在ssh命令行操作git，需要输入密码，默认是弹出图形界面对话框
```
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
source ~/.bashrc
```
# 4.4 可选采用无密码的子密钥方案
```
gpg --expert --edit-key YOUR_KEY_ID
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 4 # 选择 RSA (sign only)
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0  # 0= 永不过期
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y

# 获取新子密钥ID
list
passwd 子密钥
gpg> save

#配置 Git 使用新密钥
gpg --list-keys --keyid-format LONG

# 设置 Git 使用新子密钥
git config --global user.signingkey NEW_SUBKEY_ID
```
# 5. 可选添加 Personal access tokens
GitHub 目前支持两种类型的 personal access token：fine-grained personal access token 和 personal access tokens (classic)。   
GitHub 建议尽可能使用 fine-grained personal access token 而不是 personal access tokens (classic)。   
注意    
Fine-grained personal access token 虽然更安全且更可控，但无法完成 personal access token (classic) 所能完成的所有任务。     
请参阅下面关于 Fine-grained personal access tokens 限制的部分，以了解更多信息。     
https://docs.github.com/zh/authentication/keeping-your-account-and-data-secure/ [managing-your-personal-access-tokens#fine-grained-personal-access-tokens-limitations](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
```
git调试模式
 GIT_CURL_VERBOSE=1 GIT_TRACE=1 git clone https://github.com/xxx.git
problem :== Info: Couldn't find host github.com in the .netrc file; using defaults

vi ~/.gitconfig
[credential]
    helper = store

vi ~/.netrc
machine github.com
    login your-user-name
    password your-personal-access-token
    
登录GitHub：在浏览器中打开GitHub，然后登录您的账户。
创建Personal Access Token：
在页面右上角，点击您的头像，然后点击Settings（设置）。
在左侧边栏中，点击Developer settings（开发人员设置）。
在左侧边栏中，找到并单击Personal access tokens（个人访问令牌）。
单击Generate new token（生成新令牌）按钮。
为您的令牌指定一个描述性名称，以便于识别。
选择到期日期。您可以选择默认值或使用日历选择器设置令牌的过期日期。
选择您要授予此令牌的范围或权限。根据您的需求选择适当的选项，例如repo（存储库）
```
## 5.1 避免明文存储，优先选择SSH 密钥认证​

```
避免明文存储风险​​
.netrc以​​明文存储密码​​，若系统被入侵，PAT 可能泄露。
```
| **助手类型**         | **存储方式**                     | **安全性** | **持久性**       | **依赖环境**                          | **适用场景**                     | **配置命令**                                      |
|----------------------|----------------------------------|------------|------------------|---------------------------------------|----------------------------------|--------------------------------------------------|
| **`cache`**          | 内存缓存                         | 中         | 临时（默认15分钟）| 无                                    | 短期高频操作（如开发调试）       | `git config --global credential.helper 'cache --timeout=3600'` |
| **`store`**          | 明文存储（`~/.git-credentials`） | 低         | 永久             | 无                                    | 个人设备且低安全需求             | `git config --global credential.helper store`                 |
| **`libsecret`**      | 系统密钥环（GNOME Keyring/KWallet） | 高         | 永久             | `libsecret` + `gnome-keyring`（需安装） | 桌面环境长期使用（推荐）         | `git config --global credential.helper libsecret`            |
| **`manager-core`**   | 加密存储（跨平台）               | 高         | 永久             | 需安装 [Git Credential Manager][GCM]   | 多平台同步或企业环境             | `git config --global credential.helper manager-core`         |
| **SSH 密钥**         | 非对称加密认证                   | 极高       | 永久             | `openssh`                             | 生产环境、高安全要求             | 无需配置助手，需生成密钥并关联远程仓库：`git remote set-url origin git@github.com:user/repo.git` |

[GCM]: https://github.com/GitCredentialManager/git-credential-manager
