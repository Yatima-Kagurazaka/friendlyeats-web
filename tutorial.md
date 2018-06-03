# Cloud Firestore Web Codelab 和訳+改変

## はじめに

このコードラボでは，レストランの評価アプリを通じて，以下のようなことを学びます；
- Cloud Firestoreでのデータ読み書き
- Cloud Firestoreでのリアルタイム更新
- Firebase Authenticationとセキュリティルールを利用した，Cloud Firestoreでのデータ保護
- 複合的なCloud Firestoreクエリ

Qiita

## セットアップ

先ほど作ったプロジェクトを紐付けましょう．  
  
```bash
firebase use --add
```
入力しないと先へ進めないところでは，適当に`staging`など入れておけばOKです．  
問題なければ右下の[進む]を．  
.  
.  
  
※ちなみに…  
```
Error: Authentication Error: Your credentials are no longer valid. Please run firebase login --reauth
```  
と出た場合は，   
```bash
firebase login --reauth --no-localhost
```  
と  
`--no-localhost`  
を加えてお試しください．

## Firebase Authenticationの有効化

匿名ログインを有効にします．  
下のコマンドを打ち，表示されたリンクに飛びましょう．  
（もしリンクがおかしい場合はコピペを）  
```bash
firebase open auth
```
（あるいは[ウェブコンソール](https://console.firebase.google.com)からプロジェクト選択後，Authentication > [ログイン方法](https://console.firebase.google.com/u/0/project/_/authentication/providers)でも行けます）  
  
そして「匿名」をクリックし，有効にして保存しましょう．  
![Anonymous](https://codelabs.developers.google.com/codelabs/firestore-web/img/17fd5c19e8b40eae.png)

（Firebase Authenticationの詳細については[こちら](https://firebase.google.com/products/auth/)）
  
## ローカルサーバ

さて，
```bash
firebase serve
```
と入れてみましょう．  
そして表示された`http://localhost:5000`をクリック．  
少し待つと…いかがでしょうか？表示されましたか？
  
![hennnano](https://codelabs.developers.google.com/codelabs/firestore-web/img/5e02c29efb9eebd3.png)  
  
変なキャラクターが表示されなければ，一度ブラウザを更新してみてください．

## Firestoreへの書き込み

Firestoreでは，データは「ドキュメント」と「コレクション」（＋「サブコレクション」）に分かれます（[詳細](https://firebase.google.com/docs/firestore/data-model)）．  
今回はレストラン一覧のアプリを作りましょう．  
  
`restaurants` というコレクションを作ります．  
`walkthrough editor-open-file friendlyeats-web/scripts/FriendlyEats.Data.js "scripts/FriendlyEats.Data.js"`を開いて，関数 `FriendlyEats.prototype.addRestaurant` を見つけ(Ctrl + F)，下記のように書き換えましょう．
```
FriendlyEats.prototype.addRestaurant = function(data) {
  var collection = firebase.firestore().collection('restaurants');
  return collection.add(data);
};
```
これは `restaurants` コレクションに新しいドキュメントを加えるものです．  
まず `restaurants` への参照をし，その後 `add` （加える）という手順です．  
（ちなみにドキュメントはJavaScriptでのObjectであるべきです．[詳細](https://www.google.co.jp/search?q=js+object)）
  
出来たら，アプリ内の「Add Mock Data」を押しましょう．  
しかしアプリ内では，まだレストランは表示されません．  
データの読み込みを実装する必要があります．  
  
（データの保存はされています！[ウェブコンソール](https://console.firebase.google.com)から，database > [データ](https://console.firebase.google.com/u/0/project/_/database/firestore/data)を見てみてください．）
## Firestoreからの読み込み - 1

まず，デフォルトのフィルタされていないレストラン一覧を取得してみましょう．  
  
`walkthrough editor-open-file friendlyeats-web/scripts/FriendlyEats.Data.js "scripts/FriendlyEats.Data.js"`で `FriendlyEats.prototype.getAllRestaurants()` メソッドを見つけ，下記のように書き換えましょう．
```
FriendlyEats.prototype.getAllRestaurants = function(render) {
 var query = firebase.firestore()
   .collection('restaurants')
   .orderBy('avgRating', 'desc')
   .limit(50);
 this.getDocumentsInQuery(query, render);
};
 ```
 ここでは，`restaurants` というトップレベルのコレクションをレーティング平均（今はまだすべて0）で並べ替え，上限50に制限する，という指定をしています（クエリ）．  
 そして `getDocumentsInQuery()` という，読み込み・描画のためのメソッドが呼ばれます．  
   
 それを実装していきましょう．
 
##  Firestoreからの読み込み - 2

`FriendlyEats.prototype.getDocumentsInQuery()` メソッドを下記のように書き換えます．

```
FriendlyEats.prototype.getDocumentsInQuery = function(query, render) {
 query.onSnapshot(function(snapshot) {
   if (!snapshot.size) return render();

   snapshot.docChanges.forEach(function(change) {
     if (change.type === 'added') {
       render(change.doc);
     }
   });
 });
};
```

この `query.onSnapshot` は，クエリ結果に変更があるたび内部（コールバック）を実行します．  
最初に，`restaurants` コレクション全体とともにコールバックが呼ばれます．  
そして中の個々のドキュメントが関数 `render` に渡されます．
    
ちなみに `change.type` は `removed` や `changed` となる場合もあるため，`added` に限定しています． 

##  Firestoreからの読み込み - 3

さて，アプリを更新してみましょう．  
レストラン一覧が表示されているはずです！  
  
![restaurants](https://codelabs.developers.google.com/codelabs/firestore-web/img/89e58de6c41cee60.png)
  
更に，データに変更があるとリアルタイムで更新されます．
試しに[ウェブコンソール](https://console.firebase.google.com)からデータを追加してみてください（プロジェクトを選択後，database > [データ](https://console.firebase.google.com/u/0/project/_/database/firestore/data)）．  
アプリもすぐに更新されるでしょう！  
  
ちなみに `query.onSnapshot` の代わりに `query.get` を使うことで，リアルタイム更新ではなく1回だけ取得することもできます．

## Firestoreからの1回のみの読み込み

1回のみの取得も試してみましょう．  
特定のレストランのページで使われる `FriendlyEats.prototype.getRestaurant()` メソッド（`walkthrough editor-open-file friendlyeats-web/scripts/FriendlyEats.Data.js "scripts/FriendlyEats.Data.js"`）に利用してみます．  

```
FriendlyEats.prototype.getRestaurant = function(id) {
  return firebase.firestore().collection('restaurants').doc(id).get();
};
```
これで個々のページが見られるようになったはずです．  
レビュー追加機能はまた後ほど．

## 並び替え，フィルタ - 1

応用的なクエリを使ってフィルタを作ってみましょう．  
以下は寿司屋を取得するクエリです．  
```
var filteredQuery = query.where('category', '==', 'Sushi')
```
`where()` メソッドで，コレクション内の特定条件のドキュメントのみ指定できます．  
ここでは，`category` が `Sushi` であるもののみとしています．  
  
今回のアプリでは，複数のフィルタを同時に使えるようにしましょう．  
例えば「サンフランシスコのピザ屋」や「ロサンゼルスのシーフード屋を人気順で」など．  
  
## 並び替え，フィルタ - 2

`walkthrough editor-open-file friendlyeats-web/scripts/FriendlyEats.Data.js "scripts/FriendlyEats.Data.js"` で `FriendlyEats.prototype.getFilteredRestaurants()` メソッドを見てみましょう．  
このメソッドでは，複数の基準でレストランをフィルタするクエリを作成します．  
メソッドを次のように書き換えます．

```
FriendlyEats.prototype.getFilteredRestaurants = function(filters, render) {
var query = firebase.firestore().collection('restaurants');

if (filters.category !== 'Any') {
  query = query.where('category', '==', filters.category);
}

if (filters.city !== 'Any') {
  query = query.where('city', '==', filters.city);
}

if (filters.price !== 'Any') {
  query = query.where('price', '==', filters.price.length);
}

if (filters.sort === 'Rating') {
  query = query.orderBy('avgRating', 'desc');
} else if (filters.sort === 'Reviews') {
  query = query.orderBy('numRatings', 'desc');
}

this.getDocumentsInQuery(query, render);
};
```

これは，複数の `where` フィルタと1つの `orderBy` 句を追加して，ユーザーの入力に基づいて複合クエリを作成します．  
今回のクエリでは，ユーザーの要件に合ったレストランだけが返されます．  
  
## 並び替え，フィルタ - 3
  
アプリ内で，価格・都市・カテゴリ別にフィルタできることを確認します．  
試している間に，ブラウザの開発用ツールのログに次のようなエラーが表示されるでしょう．  
```
The query requires an index. You can create it here: https://console.firebase.google.com/project/.../database/firestore/indexes?create_index=...
```
これは，Cloud Firestoreがほとんどの複合クエリに対してインデックスを必要とするためです．  
クエリにインデックスを張ることで，Firestoreを迅速・大規模に保ちます．  
  
エラーメッセージからリンクを開くと，Firebaseコンソールのインデックス作成UIが自動的に開き，正しいパラメータが入力されます．  
（Firestoreのインデックスの詳細については[こちら](https://firebase.google.com/docs/firestore/query-data/indexing)）

## インデックスのデプロイ

アプリ内の全パターンを調べて全リンクをたどるのは大変なので，`firebase` コマンドを使用して多数のインデックスを一度にデプロイしましょう．  
以下のような内容の， `walkthrough editor-open-file friendlyeats-web/firestore.indexes.json "firestore.indexes.json"` というファイルがあるかと思います．
```
{
 "indexes": [
   {
     "collectionId": "restaurants",
     "fields": [
       { "fieldPath": "city", "mode": "ASCENDING" },
       { "fieldPath": "avgRating", "mode": "DESCENDING" }
     ]
   },
   ...
 ]
}
```
このファイルには既に，すべての可能なフィルタの組み合わせに必要な，すべてのインデックスが記載されています．

これらのインデックスをこのコマンドでデプロイしましょう；
```bash
firebase deploy --only firestore:indexes
```

数分後，インデックスが有効になり，警告メッセージが表示されなくなります．


## トランザクション - 1

このセクションでは，ユーザーがレストランにレビューを追加する機能を追加します．  
  
これまでのデータ書き込みは，実行されるかされないかであり中途半端な状態がなく（アトミック），比較的単純でした．  
エラーが発生した場合は，自動的に再試行するか，ユーザーに再試行するように促せば済みました．

レストランに評価を追加するには，複数の読み書きを調整する必要があります．  
まずレビュー自体を送信しなければなりません．  
そしてレストランの評価と平均評価を更新する必要があります．  
これらのいずれかのみが失敗した場合，データベースのある部分と別の部分が不整合となります．

幸運なことに，Cloud Firestoreにはトランザクションがあります． 
単一のアトミックな操作で複数の読み書きを実行し，データの一貫性を保ちます．

## トランザクション - 2

`FriendlyEats.prototype.addRating()` メソッドを次のように書き換えます．

```
FriendlyEats.prototype.addRating = function(restaurantID, rating) {
 var collection = firebase.firestore().collection('restaurants');
 var document = collection.doc(restaurantID);

 return document.collection('ratings').add(rating).then(function() {
   return firebase.firestore().runTransaction(function(transaction) {
     return transaction.get(document).then(function(doc) {
       var data = doc.data();

       var newAverage =
         (data.numRatings * data.avgRating + rating.rating) /
         (data.numRatings + 1);

       return transaction.update(document, {
         numRatings: data.numRatings + 1,
         avgRating: newAverage
       });
     });
   });
 });
};
```

ここではまず，レビューをそのレストランの `ratings` サブコレクションに追加します．  
この書き込みが成功すると，レストラン自体の `averageRating` と `ratingCount` の数値を更新するトランザクションを開始します．  
  
注：サーバー上でトランザクションが失敗すると，コールバックも繰り返し実行されます．  
アプリの状態を変更するロジックを，トランザクションのコールバック内に配置しないでください．  


## セキュリティルール

データの読み書きを一望してきましたが，実戦投入するにはセキュリティルールも設定する必要があるでしょう． 
   
今のところ誰でも読み書きできるようになっていますが，認証されたユーザーだけに制限してみましょう．  
（実際はより細かく制限する必要がある場合が多いです）  

[ウェブコンソール](https://console.firebase.google.com)を開き，Database > [ルール](https://console.firebase.google.com/project/_/database/firestore/rules)を選択します．  
次のように書き換えてみてください．
```
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // Only authenticated users can read or write data
      allow read, write: if request.auth != null;
    }
  }
}
```
「公開」を押せば反映されます．  
あるいは，コマンドでも可能です．  
```bash
firebase deploy --only firestore:rules
```
このコマンドは `walkthrough editor-open-file friendlyeats-web/firestore.rules "firestore.rules"` の内容をデプロイします（ここでは準備されています）．  
  
セキュリティルールの詳細は[こちら](https://firebase.google.com/docs/firestore/security/get-started)．  
今回のこのアプリ向けのルールをより細かく設定した[例](https://github.com/firebase/quickstart-js/blob/master/firestore/firestore.rules)もご覧ください．
## お疲れさまでした！

このコードラボでは，Firestoreでの基本的・発展的な読み書き，セキュリティルールによるデータ保護について学びました．  
詳しくは以下などご参照ください．
- [Cloud Firestoreの紹介](https://firebase.google.com/docs/firestore/)
- [データ構造の選択](https://firebase.google.com/docs/firestore/manage-data/structure-data)
- [Cloud Firestore ウェブクライアントのサンプル](https://firebase.google.com/docs/firestore/client/samples-web)

