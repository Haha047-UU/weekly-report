# 实现登录
## 1.在github上创建OAuth
登录GitHub，点击settings,再点击Developer settings，使用OAuth Apps，new OAuth Apps。
填入Application name:example  Homepage URL:http://localhost:3000 
Authorization callback URL:http://localhost:3000/api/auth/callback/github
再点击Register application

可获得Client ID，点击Generate a new client secret,创建一个Client secrets
## 2.项目配置
在项目中安装auth.js
```
npm install next-auth
```
生成加密密钥
```
npx auth secret
```
获得了一组密钥
AUTH_SECRET=xxxxxxxx

创建.env.local文件,文件内容如下：
```
AUTH_SECRET=xxxxxxxx

AUTH_AUTH0_ID=
AUTH_AUTH0_SECRET=
AUTH_AUTH0_ISSUER=

AUTH_FACEBOOK_ID=
AUTH_FACEBOOK_SECRET=

AUTH_GITHUB_ID=github上创建的client ID
AUTH_GITHUB_SECRET=github上获得的Client secrets

AUTH_GOOGLE_ID=
AUTH_GOOGLE_SECRET=

AUTH_TWITTER_ID=
AUTH_TWITTER_SECRET=
```
创建auth.js,里面的内容如下：
```
import NextAuth from "next-auth"
import GitHub from "next-auth/providers/github"
 
export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [GitHub],
})
```
之后
```
npm run dev
```
查看项目用github登录是否可以使用