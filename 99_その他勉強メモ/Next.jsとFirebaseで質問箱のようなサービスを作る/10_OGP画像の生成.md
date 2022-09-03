# OGP画像の生成
- 画像処理のため、node-canvasというライブラリをインストールする。
```
$ yarn add canvas
```
## 画像を作成する
- `http://localhost:3000/api/answers/回答ID/ogp`と呼び出すAPIを作成するため、`pages/api/answers/[id]/ogp.ts`にAPIの処理を記述する。
```ts
import { NextApiRequest, NextApiResponse } from 'next'
import { createCanvas } from 'canvas'

export default async (req: NextApiRequest, res: NextApiResponse) => {
  // TwitterのOGPサイズに合わせて縦横のサイズを定義する
  const width = 600
  const height = 315
  const canvas = createCanvas(width, height)
  const context = canvas.getContext('2d')

  context.fillStyle = '#888888'
  context.fillRect(0, 0, width, height)

  const buffer = canvas.toBuffer()

  res.writeHead(200, {
    'Content-Type': 'image/png',
    'Content-Length': buffer.length,
  })
  res.end(buffer, 'binary')
}
```
## 文字を描画する
### フォントの準備
- IPAフォントをダウンロードする。
- [文字情報技術促進協議会のHP](https://moji.or.jp/ipafont/ipaex00401/)よりダウンロードし、`fonts/ipaexg.ttf`に配置する。

### 実際に描画する
- フォントを登録するための関数`registerFont`をインポートする。
```ts
import * as path from 'path'
import { createCanvas, registerFont } from 'canvas'
```
## 画像を描画する
## 質問を表示する
- 質問データを読み込む
```ts
import { NextApiRequest, NextApiResponse } from 'next'
import { createCanvas, registerFont, loadImage } from 'canvas'
import * as path from 'path'
import '../../../../lib/firebase_admin'
import { firestore } from 'firebase-admin'
import { Answer } from '../../../../models/Answer'
import { Question } from '../../../../models/Question'

export default async (req: NextApiRequest, res: NextApiResponse) => {
  const id = req.query.id as string

  const answerDoc = await firestore().collection('answers').doc(id).get()
  const answer = answerDoc.data() as Answer
  const questionDoc = await firestore()
    .collection('questions')
    .doc(answer.questionId)
    .get()
  const question = questionDoc.data() as Question

  // 省略
}
```
### テキストを行ごとに分割する
- canvas は自動的に文書を折り返してくれないため、`measureText`メソッドを使って文書の横幅を計算し、適宜行ごとに分割して描画する。
```ts
type SeparatedText = {
  line: string
  remaining: string
}

function createTextLine(context, text: string): SeparatedText {
  const maxWidth = 400

  for (let i = 0; i < text.length; i++) {
    const line = text.substring(0, i + 1)
    if (context.measureText(line).width > maxWidth) {
      return {
        line,
        remaining: text.substring(i + 1),
      }
    }
  }

  return {
    line: text,
    remaining: '',
  }
}

function createTextLines(context, text: string): string[] {
  const lines: string[] = []
  let currentText = text

  while (currentText !== '') {
    const separatedText = createTextLine(context, currentText)
    lines.push(separatedText.line)
    currentText = separatedText.remaining
  }

  return lines
}
```
## 本番用の設定をする
- Vercelにデプロイする際にうまく行かなかったいかなかったので、今回はスキップする。
> [Next.jsでcanvasをいれてVercelにデプロイした際に「error /vercel/path0/node_modules/canvas: Command failed.」が発生する](https://teratail.com/questions/j9q17jb8d30tg3)
