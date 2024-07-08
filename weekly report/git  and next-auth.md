# Git and Next Auth
## git 操作
### 克隆远程仓库到本地
```
git clone <URL>
```
### 将现有项目添加到github仓库上
初始化本地Git仓库
```
git init
```
添加远程仓库
```
git remote add origin URL_ADDRESS
```
更新远程仓库的URL
```
git remote set-url origin URL_ADDRESS
```
### 将修改的文件提交到本地仓库
将本地仓库的修改推送到远程仓库
添加项目文件到Git
```
git add .
```
提交修改到暂存区
```
git commit -m "message <your choice>"
```
将本地仓库的修改推送到远程仓库
```
git push -u origin master
```
### 当远程仓库和本地仓库不一致时
拉取远程分支的更改
```
git pull orgin master
```

## next-auth-example改成可以使用的登录认证
从github上git clone next-auth-example
### 修改项目，以及github上的配置
#### 添加以下代码
这段代码的目的是使存储驱动的选择依赖于是否部署在 Vercel 平台。
```
const storage = createStorage({
  driver: process.env.VERCEL
    ? vercelKVDriver({
      url: process.env.KV_REST_API_URL,
      token: process.env.KV_REST_API_TOKEN,
      env: false,
    })
    : memoryDriver(),
})
//providers选择GitHub
```
生成一个随机的密钥
```
npx auth secret
```
创建./app/api/auth/[...nextauth]/route.ts文件，添加以下代码
```
import { handlers } from "@/auth" // Referring to the auth.ts we just created
export const { GET, POST } = handlers
```
在Github上Oauth中配置Callback URL：[origin]/api/auth/callback/github
Add Signin button
```

import { signIn } from "@/auth"
 
export function SignIn() {
  return (
    <form
      action={async () => {
        "use server"
        await signIn("github")
      }}
    >
      <button type="submit">Signin with GitHub</button>
    </form>
  )
} 
```
### 将改好的项目git push到github上，然后在vercel上部署
创建project，添加名字，以及配置Environment variables
```
AUTH_GITHUB_ID="xxxxxxxxx"
AUTH_GITHUB_SECRET="xxxxxxxxx"
AUTH_SECRET="xxxxxxxx"
KV_REST_API_READ_ONLY_TOKEN="xxxxxxxx"
KV_REST_API_TOKEN="xxxxxxxxx"
KV_REST_API_URL="xxxxxxxx"
KV_URL="xxxxxxxx"
NODE_ENV="xxxxxxxx"
```
build and deploy后配置地址，在域名解析和DNS中添加A记录，即可在域名上访问。


