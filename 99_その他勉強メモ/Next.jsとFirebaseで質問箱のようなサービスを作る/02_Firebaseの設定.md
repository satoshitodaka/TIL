# Firebaseの設定
## Next.jsに設定を組み込む
- firebase のサイト上で設定が完了したのち、アプリにFirebaseのライブラリをインストールする。
```
$ yarn add firebase
```
- 設定ファイル`lib/firebase.ts`を作成する。
```ts
// 最初のimportでは、ベースとなるパッケージを読み込む。
import { initializeApp, getApps } from 'firebase/app'
import { getAnalytics } from 'firebase/analytics'

// 以下の書き方をすると、コードは読み込むが値はインポートされない
import 'firebase/analytics'
import 'firebase/auth'
import 'firebase/firestore'

// Firebaseのコードを転記するとエラーが発生する。Firebaseはブラウザ起動時に利用するが、
// Next.jsはサーバーサイドでも稼働するのでWindow オブジェクトの有無でブラウザでの実行を判断する。
// また、このファイルが複数回読み込まれる場合があり、その度にアプリの初期化されると困るため、getApps().lengthでアプリの有無を条件に加える。
if (typeof window !== 'undefined' && getApps().length === 0) {
  // firebaseConfigには、Firebaseのコンソールに表示された内容を記述する。
  // そのまま使うと固定してしまうので、別途設定する環境変数を利用するようにする。
  const firebaseConfig = {
    apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
    authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
    projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
    storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
    messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
    appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
    measurementId: process.env.NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID,
  }

  initializeApp(firebaseConfig)
  getAnalytics()
}
```
> [app package - firebase](https://firebase.google.com/docs/reference/js/app)
> [Google アナリティクスを使ってみる - firebase](https://firebase.google.com/docs/analytics/get-started?hl=ja&platform=web)

- .env.localにFirebaseの環境変数を設定する。環境変数の値はFirebaseのコンソールを参照する。
```
NEXT_PUBLIC_FIREBASE_API_KEY=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_PROJECT_ID=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_APP_ID=XXXXXXXXX
NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=XXXXXXXXX
```
- 作成した`lib/firebase.ts`を読み込むため、全ページを作成するためのコンポートとなる`pages/_app.tsx`で読み込む。
```tsx
import '../lib/firebase'
```
## その他参考
- [import - mdn web doc](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)
- [module(import/export)ってなんなん? - Zenn](https://zenn.dev/kanachan/articles/ad28de7389bcd0)