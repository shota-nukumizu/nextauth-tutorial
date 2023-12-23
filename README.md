# 開発手順

## (1) インストール

```
npx create-next-app@latest
```

以上のコマンドを入力すると、以下の指示が表示されるので以下のように回答する。

```
What is your project named? my-app
Would you like to use TypeScript? No 
Would you like to use ESLint? No 
Would you like to use Tailwind CSS? No
Would you like to use `src/` directory? Yes
Would you like to use App Router? (recommended) No
Would you like to customize the default import alias (@/*)? No 
What import alias would you like configured? @/* No
```

## (2) NextAuth.jsのインストール

```
npm i next-auth
```

## (3) Next.js側の実装

`src/_app.js`：セッション情報の保持。

```js
import { SessionProvider } from 'next-auth/react';

function MyApp({ Component, pageProps: { session, ...pageProps } }) {
  return (
    <SessionProvider session={session}>
      <Component {...pageProps} />
    </SessionProvider>
  );
}

export default MyApp;
```

`src/pages/api/auth/[...nextauth].js`：Google認証に必要な情報を入れる。

```js
import NextAuth from "next-auth"
import GoogleProvider from "next-auth/providers/google"

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
})
```

`src/index.js`：ログイン画面の表示

```js
import { useSession, signIn, signOut } from "next-auth/react"

export default function Component() {
  const { data: session } = useSession()
  // ログイン済みのときに表示される
  if (session) {
    return (
      <>
        Signed in as {session.user.name} <br />
        <button onClick={() => signOut()}>Sign out</button>
      </>
    )
  }
  // ログインしていない状態、この画面が表示される
  return (
    <>
      Not signed in <br />
      <button onClick={() => signIn()}>Sign in</button>
    </>
  )
}
```

`.env.local`：`OOGLE_CLIENT_ID`と`GOOGLE_CLIENT_SECRET`を保持する。

```
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
NEXTAUTH_URL=http://localhost:3000/
```

## (5) Google Develoer Consoleにアクセス

OAuth 2.0クライアントIDを作成する

* Google Cloud Consoleにアクセスし、プロジェクトを作成する
* 「認証情報」ページに移動し、「認証情報を作成」を選択して、「OAuth クライアント ID」を作成する
* 承認前のURIは`http://localhost:3000/`と入力する
* 承認済みのリダイレクト URI に`http://localhost:3000/api/auth/callback/google`を追加する

## (6) サーバの立ち上げ

```
npm run dev
```

ブラウザで`http://localhost:3000/`へアクセスする。