インストール
============

## エクステンションをインストールする

エクステンションをインストールするためには、Composer を使います。次のコマンドを実行します。

```
composer require --prefer-dist yiisoft/yii-auth-client "~2.3.0"
```

または、あなたの composer.json の `require` セクションに

```json
"yiisoft/yii-auth-client": "~3.0.0"
```

を追加します。

## アプリケーションを構成する

エクステンションがインストールされた後に、認証クライアントコレクションのアプリケーションコンポーネントをセットアップする必要があります。

```php
return [
    'components' => [
        'authClientCollection' => [
            '__class' => Yiisoft\Yii\AuthClient\Collection::class,
            'clients' => [
                'google' => [
                    '__class' => Yiisoft\Yii\AuthClient\Clients\Google::class,
                    'clientId' => 'google_client_id',
                    'clientSecret' => 'google_client_secret',
                ],
                'facebook' => [
                    '__class' => Yiisoft\Yii\AuthClient\Clients\Facebook::class,
                    'clientId' => 'facebook_client_id',
                    'clientSecret' => 'facebook_client_secret',
                ],
                // etc.
            ],
        ]
        // ...
    ],
    // ...
];
```

特別な設定なしに使用できる次のクライアントが提供されています。

- [[\Yiisoft\Yii\AuthClient\Clients\Facebook|Facebook]]
- [[Yiisoft\Yii\AuthClient\Clients\GitHub|GitHub]]
- Google ([[Yiisoft\Yii\AuthClient\Clients\Google|OAuth]] または [[Yiisoft\Yii\AuthClient\Clients\GoogleHybrid|OAuth Hybrid]] で)
- [[Yiisoft\Yii\AuthClient\Clients\LinkedIn|LinkedIn]]
- [[Yiisoft\Yii\AuthClient\Clients\Live|Microsoft Live]]
- [[Yiisoft\Yii\AuthClient\Clients\Twitter|Twitter]]
- [[Yiisoft\Yii\AuthClient\Clients\VKontakte|VKontakte]]
- [[Yiisoft\Yii\AuthClient\Clients\Yandex|Yandex]]

それぞれのクライアントの構成は少しずつ異なります。
OAuth では、使おうとしているサービスからクライアント ID と秘密キーを取得することが必要です。OpenID では、たいていの場合、何も設定しなくても動作します。

## 認証データを保存する

外部サービスによって認証されたユーザを認識するために、最初の認証のときに提供された ID を保存し、以後の認証のときにはそれをチェックする必要があります。
ログインのオプションを外部サービスに限定するのは良いアイデアではありません。
外部サービスによる認証が失敗して、ユーザがログインする方法がなくなるかもしれないからです。
そんなことはせずに、外部認証と昔ながらのパスワードによるログインの両方を提供する方が適切です。

ユーザの情報をデータベースに保存しようとする場合、スキーマは次のようなものになります。

```php
class m??????_??????_auth extends \Yiisoft\Db\Migration
{
    public function up()
    {
        $this->createTable('user', [
            'id' => $this->primaryKey(),
            'username' => $this->string()->notNull(),
            'auth_key' => $this->string()->notNull(),
            'password_hash' => $this->string()->notNull(),
            'password_reset_token' => $this->string()->notNull(),
            'email' => $this->string()->notNull(),
            'status' => $this->smallInteger()->notNull()->defaultValue(10),
            'created_at' => $this->integer()->notNull(),
            'updated_at' => $this->integer()->notNull(),
        ]);

        $this->createTable('auth', [
            'id' => $this->primaryKey(),
            'user_id' => $this->integer()->notNull(),
            'source' => $this->string()->notNull(),
            'source_id' => $this->string()->notNull(),
        ]);

        $this->addForeignKey('fk-auth-user_id-user-id', 'auth', 'user_id', 'user', 'id', 'CASCADE', 'CASCADE');
    }

    public function down()
    {
        $this->dropTable('auth');
        $this->dropTable('user');
    }
}
```

上記の SQL における `user` は、アドバンストプロジェクトテンプレートでユーザ情報を保存するために使われている標準的なテーブルです。
全てのユーザはそれぞれ複数の外部サービスを使って認証できますので、全ての `user` レコードはそれぞれ複数の `auth` レコードと関連を持ち得ます。
`auth` テーブルにおいて `source` は使用される認証プロバイダの名前であり、
`source_id` はログイン成功後に外部サービスから提供される一意のユーザ識別子です。

上記で作成されたテーブルを使って `Auth` モデルを生成することが出来ます。これ以上の修正は必要ありません。

