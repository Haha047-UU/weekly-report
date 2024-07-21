# How about getting information about the GitHub repositories
## Use the access_token
面对身份认证的问题时，第一个要注意的是：如何将访问令牌从提供程序转换到 Next Auth V5 会话的？
当你的项目需要提供商的访问令牌来进行身份验证时，却很难访问这些提供商的访问令牌了。因为在 V5 中缺乏对自动刷新令牌的支持，这意味着它实际上仍然只适用于最初的 "登录并获取用户名 "步骤，而不是任何确保用户已通过身份验证的持续检查。在本次学习中，我学习的是 Next-Auth (现被称为 Auth.js )。通过Auth来实现身份验证，要使用到 GitHub 提供的访问令牌，需要在 Next-Auth 中配置。
而我学习的解决方案是使用 Typescript 的 App Dir 在 Next.js 应用程序中实现这一点。
在开发过程中，使用的是基于提交 4e02b3441 中的 next-auth-example。access_token 在默认情况下被隐藏或省略,我们要在回调中将其添加到会话类型中，为了使其正常工作而不被 Typescript 跳出，我们必须扩展该类型。在项目的基础部分创建 types/next-auth.d.ts，以容纳我们的类型扩展。代码如下：
```
//./types/next-auth.d.ts
import NextAuth, { DefaultSession } from "next-auth"
import { DefaultJWT } from "@auth/core/jwt";

declare module "next-auth" {

  // Extend session to hold the access_token
  interface Session extends DefaultSession {
    access_token: string;
  }

  // Extend token to hold the access_token before it gets put into session
  interface JWT {
    access_token: string & DefaultJWT;
  }
}
```
现在已经添加了类型扩展文件，必须告诉Typescript 要查看它，在你的 tsconfig.json 中的 compilerOptions.include 关键字中添加你刚刚创建的文件的路径。部分代码如下：
```
//./tsconfig.json
"include": [
    "process.d.ts",
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    "types/next-auth.d.ts"
  ],
```
我们要在登录过程中获取 access_token。这意味着要使用回调。为此，请修改 auth.ts 中的配置。我已经为此修改了 auth.ts，以使用我自己的提供商，并设置了 JWT 策略，下面是整个文件的样子。我添加了 session.strategy = JWT，因为它适合我寻找的令牌持久化模型，但与这里的解决方案没有直接关系：
```
//./auth.ts
import NextAuth from "next-auth"
import "next-auth/jwt"

import GitHub from "next-auth/providers/github"
import { createStorage } from "unstorage"
import memoryDriver from "unstorage/drivers/memory"
import vercelKVDriver from "unstorage/drivers/vercel-kv"
import { UnstorageAdapter } from "@auth/unstorage-adapter"
import type { NextAuthConfig } from "next-auth"

const storage = createStorage({
  driver: process.env.VERCEL
    ? vercelKVDriver({
      url: process.env.KV_REST_API_URL,
      token: process.env.KV_REST_API_TOKEN,
      env: false,
    })
    : memoryDriver(),
})

export const config = {
  theme: {
    logo: "https://next-auth.js.org/img/logo/logo-sm.png",
  },
  adapter: UnstorageAdapter(storage),
  providers: [
    GitHub
  ],
  session: {
    strategy: "jwt",
  },
  debug: true,
  basePath: "/api/auth",
  callbacks: {
    session({ session, token }) {
      if (token.access_token) {
        session.access_token = token.access_token as string// Put the provider's access token in the session so that we can access it client-side and server-side with `auth()`
      }
      return session
    },
    jwt({ token, account, profile }) {
      if (account) {
        token.access_token = account.access_token // Store the provider's access token in the token so that we can put it in the session in the session callback above
      }

      return token
    },
    authorized({ request, auth }) {
      const { pathname } = request.nextUrl
      if (pathname === "/middleware-example") return !!auth
      return true
    },
  },
  experimental: {
    enableWebAuthn: true,
  }
} satisfies NextAuthConfig

export const { handlers, auth, signIn, signOut } = NextAuth(config)

```
页面正在使用auth(),现在访问令牌就像在页面上执行 {session?.access_token} 一样简单。

## Use octokit.js to call the Github REST API
在学习集成 octokit 之前，要弄清楚环境变量配置的，是让 GitHub 认可我创建的 Oauth App，让提供商把用户的 session 相关的 token 可以给到我，再用 token 去做相关操作。相关步骤为：
1.封装 api 调用，变为函数。
2.封装 react component，调用 api。
3.封装页面，嵌入 component。
以下是部分封装 api 调用的代码（用来展现 octokit 的作用）：
```
import { Octokit } from "@octokit/rest";//从@octokit/rest 包中导入 Octokit 类。
const octokit = new Octokit({ auth: process.env.access_token });
//上行代码创建 Octokit 实例，该对象包含一个 auth 属性。auth 属性的值设置为环境变量，使用访问令牌，您可以进行身份验证以访问GitHub API，执行需要权限的操作，比如读取或修改仓库数据。
```

以下代码的作用是：根据提供的用户名获取用户的仓库列表，其中 octokit.rest.repos.listForUser 是 octokit.js 库中，用于获取用户的仓库列表。
```
export async function getUserRepos(username: string) { 
  const response = await octokit.rest.repos.listForUser({
    username,
    type: 'all',
    sort: 'updated',
    per_page: 100
  });
  return response.data;
}
```

以下代码的作用是：根据提供的仓库所有者和名称获取仓库的默认分支名称，其中 octokit.rest.repos.get 是用于获取 GitHub 上指定仓库的详细信息。

```
export async function getRepoDefaultBranch(owner: string, repo: string) {
  const response = await octokit.rest.repos.get({
    owner,
    repo
  });
  return response.data.default_branch;
}
```

以下代码的作用是：获取一个 Git 仓库的目录树结构并将其构建为一个树形结构，sha：提交的哈希值，其中 octokit.rest.git.getTree 用于获取 GitHub 仓库中特定提交或分支上的 Git 树（tree）的详细信息。这个函数可以列出树中的所有条目，包括文件和子目录。
```
export async function getRepoTree(owner: string, repo: string, sha: string) {
  const response = await octokit.rest.git.getTree({
    owner,
    repo,
    tree_sha: sha,
    recursive: 'true' //表示应该递归地获取所有子目录和文件。
  });

  return buildTree(response.data.tree);
}
```