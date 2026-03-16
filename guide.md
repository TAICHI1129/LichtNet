# Lichtnetログイン外部サイト管理者向け案内

Lichtnetログインを導入すると、あなたのサイトは **Lichtnetアカウントでのログイン** が可能になります。
ユーザーは新しいパスワードを作る必要はありません。

---

## 1 ログインの流れ

```
ユーザー
   ↓
あなたのサイト「Lichtnetでログイン」ボタン
   ↓
lichtnet/login.php
   ↓
ユーザー認証（メール＋パスワード）
   ↓
署名付きトークン発行
   ↓
callback.php にリダイレクト
   ↓
トークン検証
   ↓
ログイン完了
```

---

## 2 ログインボタン設置例

外部サイトにこのリンクを置くだけです。

```html
<a href="https://lichtnet.ct.ws/login.php?from=https://yoursite.com/callback.php">
Lichtnetでログイン
</a>
```

* `from` パラメータにログイン後戻すURLを指定してください。
* 言語切替はユーザー側で可能です（日本語・英語・スペイン語・中国語）。

---

## 3 callback.php の作成

Lichtnetログイン成功後、以下の形式で戻ります。

```
https://yoursite.com/callback.php?token=xxxxxxxx
```

例：

```php
<?php
define("SECRET_KEY","LICHTNET_SECRET_KEY");

$token=$_GET["token"] ?? "";

if(!$token) exit("token missing");

list($payload_b64,$signature)=explode(".",$token);
$expected=hash_hmac("sha256",$payload_b64,SECRET_KEY);

if(!hash_equals($expected,$signature)) exit("invalid token");

$data=json_decode(base64_decode($payload_b64),true);

if(time() > $data["exp"]) exit("token expired");

$email=$data["email"];

session_start();
$_SESSION["user"]=$email;

echo "ログイン成功: ".$email;
```

---

## 4 トークン仕様

トークン構造:

```
base64(payload).signature
```

payloadの中身:

```json
{
  "email": "user@example.com",
  "iat": 1680000000,
  "exp": 1680000120
}
```

* `email`：ユーザーのメールアドレス
* `iat`：発行時刻（秒）
* `exp`：有効期限（秒）

---

## 5 セキュリティ上の注意

* HTTPSで通信してください。
* トークンは **1回のみ使用** すること。
* callbackページは公開されるため、他の用途に使わないこと。
* 将来的には CSRF対策（state）を追加するのが望ましいです。

---

## 6 ユーザーへの表示

Lichtnetログイン画面には **規約とプライバシーポリシーの表示** があり、ユーザーはログイン時に確認可能です。

例（日本語）:

> サービスに情報を送信することがあります。利用規約、プライバシーポリシーを確認してください。

---

## 7 動作例

```
あなたのサイト
  ↓
Lichtnetでログインクリック
  ↓
lichtnet/login.php
  ↓
ユーザー認証成功
  ↓
callback.php?token=xxxxx
  ↓
ログイン完了
```
