# React Cloudfunctions Firebase Tutorial

作成日：2020/06/07

更新日：2021/04/07

---

## 今回の講座のゴール

- フロントは React，サーバは CloudFunctions（Node.js），DB は Cloud Firestore を用いたアプリケーションが手元のある状態．
- 何やってるかをできる限り自分の言葉でコメントする．
- フロント，サーバともに JavaScript の構成でなにか作れそうな感覚を得る．

---

## 今回作成するアプリケーション

- 前回の todo リストの CRUD 処理を CloudFunctions を用いた REST API として実装する．
- フロント側の動作は前回と同じ（処理は axios による http リクエストに変更）．

---

## 【必須】事前準備

- 前回のライブ講義で作成した todo リストを動く状態にしておく．
- 前回のライブ講義で作成した Firebase のプロジェクト（資料では`react-firebase`）のコンソールページを開いておく．

---

## 今回の内容の前提

前回のライブ講義と Node.js オンライン講座と課題を完了していれば OK！

- `React`を用いた基礎的な実装を行った経験がある．
- `CloudFunctions(Node.js)`と`Express`を用いた開発とデプロイの経験がある．
- REST API（http 通信）の仕組みがなんとなくわかる．

---

## 【参考】これまでの内容

- [React チュートリアル](https://github.com/taroosg/react-tutorial)

- [React + Cloud FireStore](https://github.com/taroosg/react-firebase-tutorial)

- [CloudFunctions チュートリアル](https://github.com/taroosg/cloudfunctions-express-tutorial)

---

## 環境構築（CloudFunctions）

---

### 必要なツールのバージョン確認

- Node.js と npm が必要なので，以下のコマンドで状況を確認する．
- バージョンが表示されれば OK．

```bash
$ node -v
v12.15.0
$ npm -v
6.14.5
```

---

### プロジェクトの作成

- `cloudfunctions-firebase`という名前でプロジェクトを作成する．
- 今回は例としてデスクトップに作成しているが任意の場所で OK．
- 下記コマンドを順番に実行．

---

```bash
$ cd ~/Desktop
$ mkdir cloudfunctions-firebase
$ cd cloudfunctions-firebase
$ firebase init
```

---

- `functions`を選択（スペースキー）して Enter．

```bash
? Which Firebase CLI features do you want to set up for this folder? Press Space
 to select features, then Enter to confirm your choices.
 ◯ Database: Deploy Firebase Realtime Database Rules
 ◯ Firestore: Deploy rules and create indexes for Firestore
❯◉ Functions: Configure and deploy Cloud Functions
 ◯ Hosting: Configure and deploy Firebase Hosting sites
 ◯ Storage: Deploy Cloud Storage security rules
 ◯ Emulators: Set up local emulators for Firebase features
```

---

- `Use an existing project`にカーソルを合わせて Enter．

```bash
? Please select an option: (Use arrow keys)
❯ Use an existing project
  Create a new project
  Add Firebase to an existing Google Cloud Platform project
  Don't set up a default project
```

---

- 前回のライブ講義で使用したプロジェクトにカーソルを合わせて Enter．

```bash
? Select a default Firebase project for this directory:
  hoge-216007 (hoge)
  hogehoge (hogehoge)
  fuga-1d565 (fuga)
❯ react-firebase (react-firebase)
  fugafuga (fugafuga)
  piyo-r359f (piyo)
  piyopiyo (piyopiyo)
```

---

- `JavaScript`にカーソルを合わせて Enter．

```bash
? What language would you like to use to write Cloud Functions? (Use arrow keys)

❯ JavaScript
  TypeScript
```

---

- `n`->`y`の順に入力&Enter して完了！

```bash
? Do you want to use ESLint to catch probable bugs and enforce style? No
✔  Wrote functions/package.json
✔  Wrote functions/index.js
✔  Wrote functions/.gitignore
? Do you want to install dependencies with npm now? Yes
...
✔  Firebase initialization complete!
$
```

---

### 動作確認

- エディタで作成したプロジェクトを開く．
- `functions/index.js`を開き，下記のように編集する．

```js
// index.js
const functions = require("firebase-functions");

// ↓↓↓ コメント外す ↓↓↓
exports.helloWorld = functions.https.onRequest((request, response) => {
  response.send("Hello from Firebase!");
});
```

---

- 編集したらローカルサーバを立ち上げて．．．
- （ローカルサーバの URL はメモしておくと良い．）

```bash
$ firebase serve
...
✔  functions[helloWorld]: http function initialized (http://localhost:5000/react-firebase/us-central1/helloWorld).
```

---

- ターミナルからリクエストを送信！
- `Hello from Firebase!`が返ってくれば OK！

```bash
$ curl http://localhost:5000/react-firebase/us-central1/helloWorld
Hello from Firebase!
```

---

### `Express`と`cors`の導入

- 【重要】functions フォルダに移動しておく．
- 下記コマンドを実行してインストールする．

```bash
$ cd functions
$ npm install express
$ npm install cors
```

---

- インストールが終わったら index.js を編集する．

```js
const functions = require("firebase-functions");
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());

app.get("/hello", (req, res) => {
  res.send("Hello Express!");
});

const api = functions.https.onRequest(app);
module.exports = { api };
```

---

- ローカルサーバを立ち上げて．．．

```bash
$ firebase serve
...
✔  functions[api]: http function initialized (http://localhost:5000/react-firebase/us-central1/api).
```

---

- ターミナルからリクエスト送信！
- `Hello Express!`が返ってくれば OK！

```bash
$ curl http://localhost:5000/react-firebase/us-central1/api/hello
Hello Express!
```

---

## CloudFunctions と CloudFirestore の通信の準備

---

### 必要なファイルのダウンロード

- Firebase のコンソールにアクセスし，今回のプロジェクトのページを表示．
- ⚙ -> `プロジェクトを設定` で設定画面を表示．
- `サービスアカウント` -> `新しい秘密鍵を作成` の順にクリック．
- 適当な場所に json ファイルを保存する．
- コンドール画面は開いたままにしておこう．

---

![コンソール画面01](./images/console01.png)

---

### json ファイルの配置と構成ファイルの作成

- `functions`ディレクトリの中に`model`ディレクトリを作成する．
- `model`ディレクトリの中に ↑ でダウンロードした json ファイルを移動する．
- `model`ディレクトリの中に`firebase.js`ファイルを作成する．

---

- `firebase.js`ファイルに ↑ で開いたコンソール画面から`Admin SDK構成スニペット`をコピペする．
- `var serviceAccount = require("...");`の部分の`require()`内を`jsonファイルのパス`に書き換える．
- 最下行に`module.exports = admin;`を追記する．

---

```js
// firebase.js
var admin = require("firebase-admin");

var serviceAccount = require("./react-firebase-firebase-adminsdk-8hsn0-hogehoge.json");

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://react-firebase.firebaseio.com",
});

module.exports = admin;
```

---

## Functions と Firestore の通信確認（Read の処理）

---

### エンドポイントの作成とデータ取得

- CRUD 処理に対応するエンドポイントを作成する．
- 続いて，Firestore のデータを全件取得する処理を追記し，動作を確認する．

---

- 今回作成するエンドポイントは以下の 4 つ．

| Method | URI          | 処理内容                   |
| ------ | ------------ | -------------------------- |
| GET    | /            | 全ドキュメントの取得       |
| POST   | /            | 新規ドキュメントの登録     |
| PUT    | /:documentId | 指定したドキュメントの更新 |
| DELETE | /:documentId | 指定したドキュメントの削除 |

---

- まず，`index.js`に下記のように CRUD のエンドポイント 4 つを作成する．
- firestore 関連のモジュール読み込みも行う．

```js
// index.js
const functions = require("firebase-functions");
const express = require("express");
const cors = require("cors");
// ↓↓↓ 追記 ↓↓↓
const admin = require("./model/firebase");
const db = admin.firestore();
// ↑↑↑ 追記 ↑↑↑

const app = express();
app.use(cors());

// get all items
app.get("/", async (req, res, next) => {
  res.send("todo get endpoint");
});

// post item
app.post("/", async (req, res, next) => {
  res.send("todo post endpoint");
});

// update item
app.put("/:id", async (req, res, next) => {
  res.send("todo put endpoint");
});

// delete item
app.delete("/:id", async (req, res, next) => {
  res.send("todo delete endpoint");
});

const api = functions.https.onRequest(app);
module.exports = { api };
```

---

- 続いて，Firestore のアイテムを全件取得する処理を追記する．

```js
// index.js
// ...省略
// get all items
app.get("/", async (req, res, next) => {
  try {
    const todoSnapshot = await db
      .collection("todos")
      .orderBy("isDone", "asc")
      .orderBy("limit", "asc")
      .get();
    const todos = todoSnapshot.docs.map((x) => {
      return {
        id: x.id,
        data: x.data(),
      };
    });
    res.send(todos);
  } catch (e) {
    next(e);
  }
});
// ...省略
```

---

- ローカルサーバを立ち上げて．．．

```bash
$ firebase serve
✔  functions[api]: http function initialized (http://localhost:5000/react-firebase/us-central1/api).
```

---

- ターミナルからリクエストを送信！
- Firestore に保存されているデータが JSON になって返ってくれば OK！

```bash
$ curl http://localhost:5000/react-firebase/us-central1/api/
[
  {
    "id": "dY8pOkIUEXTOGC5LbObZ",
    "data": {
      "isDone": false,
      "limit": {
        "_seconds": 1590786000,
        "_nanoseconds": 0
      },
      "todo": "Node.js課題"
    }
  },
  ...
]
```

---

### デプロイ

- 下記コマンドでデプロイする．
- デプロイ先の URL はメモしておくと良い．

```bash
$ firebase deploy
...
✔  functions[api(us-central1)]: Successful create operation.
Function URL (api): https://us-central1-react-firebase.cloudfunctions.net/api

✔  Deploy complete!
```

---

- ターミナルからリクエスト送信して．．．
- Firestore に保存されているデータが JSON になって返ってくれば OK！

```bash
$ curl https://us-central1-react-firebase.cloudfunctions.net/api/
[
  {
    "id": "dY8pOkIUEXTOGC5LbObZ",
    "data": {
      "isDone": false,
      "limit": {
        "_seconds": 1590786000,
        "_nanoseconds": 0
      },
      "todo": "Node.js課題"
    }
  },
  ...
]
```

---

### React で実装したクライアントアプリケーションからのリクエスト処理

- 前回の講座で実装した React のアプリケーションをエディタで開く．
- CloudFunctions にリクエストを送るため，下記のコマンドで`axios`をインストールする．

```bash
$ npm install axios
```

---

- 続いて，`ItemList.jsx`コンポーネントで`axios`を import する．

```js
import axios from "axios";
```

- `getTodosFromFirestore`関数を以下のように編集する．

```js
// ItemList.jsx
const getTodosFromFirestore = async () => {
  const requestUrl = "Node.jsで設定したエンドポイントのURL";
  const todoArray = await axios.get(requestUrl);
  setTodoList(todoArray.data);
  return todoArray.data;
};
```

---

- `Firestore`の値を CloudFunctions から取得した場合，`timestamp`の形式が異なる．
- `{seconds:..., nanoseconds:...}`が`{_seconds:..., _nanoseconds:...}`となっているため`Item.jsx`の表示箇所を修正する．

```js
// Item.jsx（deleteボタンの下部分）
{
  !todo.data.isDone ? (
    <div>
      <p>
        締め切り：{convertFromTimestampToDatetime(todo.data.limit._seconds)}
      </p>
      <p>やること：{todo.data.todo}</p>
    </div>
  ) : (
    <div>
      <p>
        <del>
          締め切り：{convertFromTimestampToDatetime(todo.data.limit._seconds)}
        </del>
      </p>
      <p>
        <del>やること：{todo.data.todo}</del>
      </p>
    </div>
  );
}
```

---

### 動作確認

- 記述したら`npm start`でアプリを立ち上げ，動作確認する．
- これまでと同じ動作（画面に todo の一覧が表示される）ができていれば OK．
- フロントの`React`とサーバの`CloudFunctions`で通信ができたことが確認できた！

---

### `CloudFUnctions`（サーバ側）を使用する利点

- `Firestore`の処理自体は`React`からでもできる．
- サーバ側に処理を分けることで，API キーなどの情報を秘匿できる．
- フロント側もシンプルは http リクエストを行えば良いので実装の負担も軽減される．

---

## Create の処理

- `React`と`CloudFunctions`の通信は実装できたので，Read 以外の処理も追加する．

---

### `React`側の処理

- `InputForm.jsx`で`axios`を読み込む．

```js
import axios from "axios";
```

- `postDataToFirestore`関数を以下のように編集する．
- `axios.post()`は第 1 引数にリクエスト先，第 2 引数に post するデータを設定する．

```js
// InputForm.jsx
const postDataToFirestore = async (collectionName, postData) => {
  const requestUrl = "Node.jsで設定したエンドポイントのURL";
  const addedData = await axios.post(requestUrl, postData);
  return addedData;
};
```

---

### `Node.js`側の処理

- index.js の post 用エンドポイントを以下のように追記．
- 送信されてきたデータが空のときはエラーを返す．
- その他の場合は`Firestore`へ登録処理を行い，ID を登録データを返す．

---

```js
app.post("/", async (req, res, next) => {
  try {
    const postData = req.body;
    if (!postData) {
      throw new Error("Data is blank");
    }
    const ref = await db.collection("todos").add(postData);
    res.send({
      id: ref.id,
      data: postData,
    });
  } catch (e) {
    next(e);
  }
});
```

---

- 追記したらデプロイする．

```bash
$ firebase deploy
```

---

### 動作確認

- React 側からフォームにデータを入力して送信する．
- 一覧に追加されれば OK．

---

## Update の処理

- 続いて Update の処理を実装する．

---

### `React`側の処理

- `Item.jsx`で`axios`を読み込む．

```js
import axios from "axios";
```

---

- `postDataToFirestore`関数を以下のように編集する．
- `axios.put()`は第 1 引数にリクエスト先，第 2 引数に更新するデータを設定する．
- リクエスト先に更新したいドキュメントの ID を付加すると Node.js 側で取得できる．

```js
// Item.jsx
const updateDataOnFirestore = async (collectionName, documentId, isDone) => {
  const newData = { isDone: isDone ? false : true };
  const requestUrl = "Node.jsで設定したエンドポイントのURL";
  const updatedData = await axios.put(`${requestUrl}${documentId}`, newData);
  getTodosFromFirestore();
  return updatedData;
};
```

---

### `Node.js`側の処理

- index.js の put 用エンドポイントを以下のように追記．
- ID または更新データが空の場合はエラーを返す．
- URL に付加されたドキュメント ID を読み取り，更新するデータを指定する．

---

```js
// index.js
app.put("/:id", async (req, res, next) => {
  try {
    const id = req.params.id;
    const newData = req.body;
    if (!id) {
      throw new Error("id is blank");
    }
    if (!newData) {
      throw new Error("data is blank");
    }
    const ref = await db.collection("todos").doc(id).update(newData);
    res.send({
      id: ref.id,
      data: newData,
    });
  } catch (e) {
    next(e);
  }
});
```

---

- 追記したらデプロイする．

```bash
$ firebase deploy
```

---

### 動作確認

- React 側でチェックボックスを操作する．
- 取り消し線やソート順が反映されれば OK！

---

## Delete の処理

- 最後は Delete の処理を実装する．

---

### `React`側の処理

- `postDataToFirestore`関数を以下のように編集する．
- `axios.delete()`は URL にドキュメント ID を設定する．
- Node.js 側では ID さえわかれば良いので，送信データは特になし．

```js
// Item.jsx
const deleteDataOnFirestore = async (collectionName, documentId) => {
  const requestUrl = "Node.jsで設定したエンドポイントのURL";
  const removedData = await axios.delete(`${requestUrl}${documentId}`);
  getTodosFromFirestore();
  return removedData;
};
```

---

### `Node.js`側の処理

- index.js の delete 用エンドポイントを以下のように追記．
- delete ではドキュメント id を指定できれば良いので，URL に付加されたドキュメント ID で削除対象を指定する．

---

```js
// Item.jsx
app.delete("/:id", async (req, res, next) => {
  try {
    const id = req.params.id;
    if (!id) {
      throw new Error("id is blank");
    }
    const ref = await db.collection("todos").doc(id).delete();
    res.send({
      id: ref.id,
    });
  } catch (e) {
    next(e);
  }
});
```

---

- 追記したらデプロイする．

```bash
$ firebase deploy
```

---

### 動作確認

- React 側で削除ボタンを操作する．
- データの削除が反映されれば OK！
- これで CRUD 処理完成！！

---

## まとめ

- Firebase 関連は CloudFunctions に任せる！
- React は http リクエスト送るだけのシンプルな実装！
- React 側に Firebase 関連の情報は置かないで OK！

---

## 今後のチャレンジ

- エラー処理を追加してみる．
- サーバ側の役割（リクエスト&レスポンス，ロジック，DB の CRUD）をレイヤーに分けてみる．
- DB を別のもので実装してみる．
- TypeScript で実装してみる．
- Next.js や NestJS に挑戦してみる．

---

今回はここまで( `･ω･)b🍻
