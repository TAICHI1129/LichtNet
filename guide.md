# Lichtnetログイン導入ガイド（外部サイト管理者様向け）

Lichtnetログインを導入すると、
貴サイトにて **Lichtnetアカウントによるログイン機能** を提供できます。

ユーザーは新たにパスワードを作成する必要がなく、スムーズにログイン可能になります。

---

## 1. ログインの仕組み

以下の流れでログインが行われます。

1. ユーザーが「Lichtnetでログイン」ボタンをクリック
2. Lichtnetのログイン画面へ遷移
3. ユーザー認証（メールアドレス・パスワード）
4. 認証成功後、指定されたURLへ戻る
5. トークンを受け取り、ユーザー情報を取得

---

## 2. ログインボタンの設置方法

以下のリンクを設置してください。

```html
<a href="https://lichtnet.ct.ws/login.php?from=https://yoursite.com/callback.php">
Lichtnetでログイン
</a>
```

### 注意事項

* `from` パラメータには、ログイン後に戻るURL（callback）を指定してください
* このURLは事前にLichtnetへ登録する必要があります
* HTTPSでの利用を推奨します

---

## 3. callback.phpの実装

ログイン成功後、ユーザーは以下のURLに戻ります。

```
https://yoursite.com/callback.php
```

このページでトークンを受け取り、検証を行ってください。

### 実装例

```php
<?php

define("SECRET_KEY","LICHTNET_SECRET_KEY");

$token = $_POST["token"] ?? "";
if(!$token) die("トークンが存在しません");

/* トークン分解 */
$parts = explode(".", $token);
if(count($parts) !== 2){
    die("トークン形式が不正です");
}

list($b64, $sign) = $parts;

/* 署名検証 */
$expected = hash_hmac("sha256", $b64, SECRET_KEY);
if(!hash_equals($expected, $sign)){
    die("不正なトークンです");
}

/* データ取得 */
$data = json_decode(base64_decode($b64), true);
if(!$data){
    die("データの復号に失敗しました");
}

/* 有効期限確認 */
if($data["exp"] < time()){
    die("トークンの有効期限が切れています");
}

/* ログイン処理 */
$email = $data["email"];
$username = $data["username"];

echo "ログイン成功<br>";
echo "email: ".htmlspecialchars($email)."<br>";
echo "username: ".htmlspecialchars($username);
```

---

## 4. 取得できる情報

```json
{
  "email": "user@example.com",
  "username": "user123",
  "iat": 1680000000,
  "exp": 1680000120
}
```

* email：メールアドレス
* username：ユーザー名
* iat：発行時刻
* exp：有効期限

---

## 5. セキュリティに関する注意

* 必ずHTTPSで通信を行ってください
* トークンは1回のみ使用し、再利用しないでください
* callbackページはログイン処理専用としてご利用ください
* トークンは保存せず、セッションへ変換することを推奨します

---

## 6. 動作イメージ

```
貴サイト
  ↓
ログインボタン
  ↓
Lichtnetログイン画面
  ↓
認証成功
  ↓
callback.php（トークン受信）
  ↓
検証
  ↓
ログイン完了
```

---

## 7. よくある問題

* 登録したURLと一致していない
* http / https の違い
* URL末尾のスラッシュ違い
* POSTではなくGETで受け取っている

---

## まとめ

導入に必要な作業は以下の3点です。

1. ログインボタンを設置
2. callbackページを作成
3. トークンの検証を実装

---

本機能により、安全かつシンプルなログイン連携を実現できます。
