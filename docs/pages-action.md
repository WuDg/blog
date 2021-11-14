# pages-action

## 第一步
1. gitee->settings->公钥管理 启用 ssh-key
2. 同步 github 项目到 gitee

## 第二步
github->project->settings->secrets
* ACCESS_TOKEN：
* GITEE_PASSWORD：gitee密码
* GITEE_RSA_PRIVATE_KEY：本地生成的私钥

github->settings->developer settings->Personal access tokens->generate personal access token

## 第三步
新增文件：.github/workflows/sync.yml
```sync.yml
name: Sync

on:
  push:
    branches: [master, main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Sync to Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
        with:
          source-repo: git@github.com:WuDg/source-parent.git
          destination-repo: git@gitee.com:WuDG/source-parent.git
      
      - name: Build Gitee Pages
        uses: yanglbme/gitee-pages-action@main
        with:
            gitee-username: WuDG
            gitee-password: ${{ secrets.GITEE_PASSWORD }}
            gitee-repo: WuDG/blog

```