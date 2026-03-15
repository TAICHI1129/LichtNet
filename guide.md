# Lichtnetログイン

## 外部サイト管理者向けご案内

Lichtnetログインを利用すると、ユーザーは **Lichtnetアカウントであなたのサイトにログイン**できます。

ユーザーは新しいパスワードを作る必要がなく、既存のLichtnetアカウントをそのまま使用できます。

---

# 1 ログインの流れ

```id="m3n0sg"
ユーザー
↓
あなたのサイト
↓
「Lichtnetでログイン」クリック
↓
Lichtnetログインページ
↓
ユーザー認証
↓
署名付きトークン発行
↓
あなたのサイト(callback.php)へ戻る
↓
トークン検証
↓
ログイン完了
```

---

# 2 ログインボタンの設置

あなたのサイトに以下のリンクを設置してください。

```html id="km31zz"
<a href="https://lichtnet.example/login.php?from=https://yoursite.com/callback.php">
Lichtnetでログイン
</a>
```

### パラメータ

| パラメータ | 説明            |
| ----- | ------------- |
| from  | ログイン成功後に戻るURL |

---

# 3 callback.php の作成

Lichtnetはログイン成功後、以下の形式であなたのサイトに戻ります。

```id="sn06b4"
https://yoursite.com/callback.php?token=xxxxxxxx
```

あなたのサイトでは `callback.php` を作成し、トークンを検証してください。

---

# 4 トークン検証例

```php id="1psen5"
<?php

define("SECRET_KEY","LICHTNET_SUPER_SECRET_KEY");

$token=$_GET["token"] ?? "";

if(!$token){
exit("token missing");
}

list($payload_b64,$signature)=explode(".",$token);

$expected=hash_hmac("sha256",$payload_b64,SECRET_KEY);

if(!hash_equals($expected,$signature)){
exit("invalid token");
}

$payload=json_decode(base64_decode($payload_b64),true);

if(time() > $payload["exp"]){
exit("token expired");
}

$email=$payload["email"];

session_start();
$_SESSION["user"]=$email;

echo "ログイン成功 ".$email;
```

---

# 5 トークン仕様

Lichtnetトークンは以下の構造です。

```id="w25ejr"
base64(payload).signature
```

payload

```json id="v2tn6m"
{
 "email":"user@example.com",
 "iat":発行時間,
 "exp":有効期限
}
```

### フィールド説明

| 項目    | 説明           |
| ----- | ------------ |
| email | ユーザーのメールアドレス |
| iat   | トークン発行時刻     |
| exp   | トークン有効期限     |

---

# 6 セキュリティ注意

安全に利用するため、以下を推奨します。

* HTTPSを使用してください
* トークンは1回のみ使用してください
* callbackページを公開しないでください

---

# 7 動作例

```id="rw4jzc"
mysite.com
↓
Lichtnetでログイン
↓
lichtnet/login.php
↓
ログイン成功
↓
mysite.com/callback.php?token=xxxx
↓
ログイン完了
```
