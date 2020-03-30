# アップグレードガイド

- [5.8から6.0へのアップグレード](#upgrade-6.0)

<a name="high-impact-changes"></a>
## 重要度の高い変更

<div class="content-list" markdown="1">
- [認可リソースと`viewAny`](#authorized-resources)
- [文字列と配列のヘルパ](#helpers)
</div>

<a name="medium-impact-changes"></a>
## 重要度が中程度の変更

<div class="content-list" markdown="1">
- [Carbon 1.xのサポート停止](#carbon-support)
- [Redisデフォルトクライアント](#redis-default-client)
- [データベースの`Capsule::table`メソッド](#capsule-table)
- [EloquentのArrayableと`toArray`](#eloquent-to-array)
- [Eloquentの`BelongsTo::update`メソッド](#belongs-to-update)
- [Eloquentの主キータイプ](#eloquent-primary-key-type)
- [多言語化の`Lang::trans` and `Lang::transChoice`メソッド](#trans-and-trans-choice)
- [多言語化の`Lang::getFromJson`メソッド](#get-from-json)
- [キュー再試行制限](#queue-retry-limit)
- [メール確認再送信Route](#email-verification-route)
- [メール確認ルートの変更](#email-verification-route-change)
- [`Input`ファサード](#the-input-facade)
</div>

<a name="upgrade-6.0"></a>
## 5.8から6.0へのアップグレード

#### アップグレード見積もり時間：１時間

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

### 動作要件：PHP7.2

**影響の可能性： 中程度**

PHP7.1は２０１９年１２月に積極的にメンテナンスされなくなっています。そのため、Laravel6.0はPHP7.2以上を動作要件とします。

<a name="updating-dependencies"></a>
### 依存パッケージのアップデート

`composer.json`ファイル中の`laravel/framework`指定を`^6.0`に更新してください。

次に、アプリケーションで使用しているサードパーティパッケージを調べ、Laravel6をサポートする適切なバージョンを使用しているか確認してください。

### 認可

<a name="authorized-resources"></a>
#### 認可リソースと`viewAny`

**影響の可能性： 高い**

`authorizeResource`メソッドを用いてコントラーへ付加している認可ポリシーは、`viewAny`メソッドを定義する必要があります。コントローラの`index`メソッドへユーザーがアクセスする時に呼び出されます。定義しない場合は非認可扱いとなり、コントローラの`index`メソッドへの呼び出しは拒否されます。

#### 認可レスポンス

**影響の可能性： 低い**

`Illuminate\Auth\Access\Response`クラスのコンストラクタの使用法を変更しました。適宜更新してください。認可レスポンスを自力で構築しておらず、ポリシーの`allow`と`deny`インスタンスメソッドだけを使用しているなら、変更の必要はありません。

    /**
     * 新しいレスポンスの生成
     *
     * @param  bool  $allowed
     * @param  string  $message
     * @param  mixed  $code
     * @return void
     */
    public function __construct($allowed, $message = '', $code = null)

#### 「Deny」レスポンスを返す

**影響の可能性： 低い**

以前のリリースのLaravelでは即時に例外が投げられるため、ポリシーメソッドから`deny`メソッドの値を返す必要はありませんでした。しかしながら現在はLaravelのドキュメント通りに、ポリシーから`deny`メソッドの値を返す必要があります。

    public function update(User $user, Post $post)
    {
        if (! $user->role->isEditor()) {
            return $this->deny("You must be an editor to edit this post.")
        }

        return $user->id === $post->user_id;
    }

<a name="auth-access-gate-contract"></a>
#### `Illuminate\Contracts\Auth\Access\Gate`契約

**影響の可能性： 低い**

`Illuminate\Contracts\Auth\Access\Gate`契約は、新しく`inspect`メソッドを追加しました。自力でこのインターフェイスを実装している場合は、このメソッドを実装へ加えてください。

### Carbon

<a name="carbon-support"></a>
#### Carbon 1.xのサポート停止

**影響の可能性： 中程度**

Carbon 1.xのメンテナンス切れが近づいて来たため、[サポートを停止しました](https://github.com/laravel/framework/pull/28683)。Carbon2.0を使用するようにアプリケーションをアップグレードしてください。

### 設定

#### `AWS_REGION`環境変数

**影響の可能性： 状況による**

[Laravel Vapor](https://vapor.laravel.com)を利用する計画がある場合は、`config`ディレクトリ中のファイルのすべての`AWS_REGION`出現箇所を`AWS_DEFAULT_REGION`へ更新してください。さらに、この環境変数の名前を`.env`ファイルに含めてください。

<a name="redis-default-client"></a>
#### Redisデフォルトクライアント

**影響の可能性： 中程度**

デフォルトのRedisクライアントを`predis`から`phpredis`へ変更しました。`predis`を使い続けるには、`config/database.php`設定ファイルの`redis.client`設定オプションを`predis`に指定してください。

<a name="dynamodb-cache-store"></a>
#### DynamoDBキャッシュ保存

**影響の可能性： 条件による**

[Laravel Vapor](https://vapor.laravel.com)の使用を計画している場合は、`config/cache.php`ファイルに`dynamodb`保存のオプションを含むように変更してください。

    <?php
    return [
        ...
        'stores' => [
            ...
            'dynamodb' => [
                'driver' => 'dynamodb',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
                'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
                'endpoint' => env('DYNAMODB_ENDPOINT'),
            ],
        ],
        ...
    ];

<a name="sqs-environment-variables"></a>
#### SQS環境変数

**影響の可能性： 条件による**

[Laravel Vapor](https://vapor.laravel.com)の使用を計画している場合は、`config/queue.php`ファイルに`sqs`接続の環境変数オプションを含むように変更してください。

    <?php
    return [
        ...
        'connections' => [
            ...
            'sqs' => [
                'driver' => 'sqs',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
                'queue' => env('SQS_QUEUE', 'your-queue-name'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
            ],
        ],
        ...
    ];

### データベース

<a name="capsule-table"></a>
#### `Capsule::table`メソッド

**影響の可能性： 中程度**

> {note} この変更は`illuminate/database`を依存パッケージとして使用している、Laravel以外のアプリケーションに適用されます。

`Illuminate\Database\Capsule\Manager`クラスの`table`メソッド使用方法が変更され、
第２引数にテーブルのエイリアスを受け取るようにしました。Laravelアプリケーション外で、`illuminate/database`を使用している場合は、このメソッドの呼び出しを適切に更新してください。

    /**
     * fluentクエリビルダの取得
     *
     * @param  \Closure|\Illuminate\Database\Query\Builder|string  $table
     * @param  string|null  $as
     * @param  string|null  $connection
     * @return \Illuminate\Database\Query\Builder
     */
    public static function table($table, $as = null, $connection = null)

#### `cursor`メソッド

**影響の可能性： 低い**

`cursor`メソッドが`Generator`インスタンスの代わりに、`Illuminate\Support\LazyCollection`インスタンスを返すようにしました。`LazyCollection`もジェネレータ同様に繰り返し処理できます。

    $users = App\User::cursor();

    foreach ($users as $user) {
        //
    }

<a name="eloquent"></a>
### Eloquent

<a name="belongs-to-update"></a>
#### `BelongsTo::update`メソッド

**影響の可能性： 中程度**

一貫性を取るため、`BelongsTo`リレーションの`update`メソッドは、アドホックな更新クエリとして機能するようにしました。つまり、複数代入の保護やEloquentイベントの発行を行わなくなりました。これにより、他のタイプのリレーションの`update`メソッドと一貫性が取れました。

`BelongsTo`リレーションを通じて所属しているモデルを更新し、複数代入更新の保護とイベントを受け取りたい場合は、モデル自身の`update`メソッドを呼び出してください。

    // アドホッククエリ、複数代入保護やイベントはない
    $post->user()->update(['foo' => 'bar']);

    // モデルの更新、複数代入保護とイベントは提供される
    $post->user->update(['foo' => 'bar']);

<a name="eloquent-to-array"></a>
#### Arrayableと`toArray`

**影響の可能性： 中程度**

Eloquentモデルの`toArray`メソッドは、`Illuminate\Contracts\Support\Arrayable`を実装している属性を全部配列へキャストするようにしました。

<a name="eloquent-primary-key-type"></a>
#### 主キータイプの宣言

**影響の可能性： 中程度**

Laravel6.0では、整数キータイプに対する[パフォーマンスの最適化](https://github.com/laravel/framework/pull/28153)を行いました。モデルの主キーに文字列を使用してる場合は、モデル上の`$keyType`プロパティを使い、キータイプを宣言する必要があります。

    /**
     * 主キーの「タイプ」
     *
     * @var string
     */
    protected $keyType = 'string';

### メール確認

<a name="email-verification-route"></a>
#### 再送信確認ルートのHTTPメソッド

**影響の可能性： 中程度**

CSRF攻撃の可能性を防ぐため、Laravelの組み込みメール確認を利用する場合に、ルータに登録される`email/resend`ルートは、`GET`から`POST`へメソッドを変更しました。そのため、このルートに対して正しいリクエストタイプを送るように、フロントエンドを更新する必要があります。たとえば、組み込みのメール確認テンプレートのスカフォールドを使用している場合は、次のようになります。

    {{ __('Before proceeding, please check your email for a verification link.') }}
    {{ __('If you did not receive the email') }},

    <form class="d-inline" method="POST" action="{{ route('verification.resend') }}">
        @csrf

        <button type="submit" class="btn btn-link p-0 m-0 align-baseline">
            {{ __('click here to request another') }}
        </button>.
    </form>

<a name="mustverifyemail-contract"></a>
#### `MustVerifyEmail`契約

**影響の可能性： 低い**

`Illuminate\Contracts\Auth\MustVerifyEmail`契約へ新たに、`getEmailForVerification`メソッドを追加しました。この契約を自分で実装している場合は、このメソッドを実装してください。このメソッドはオブジェクトに関連したメールアドレスを返します。`App\User`モデルで`Illuminate\Auth\MustVerifyEmail`トレイトを使用している場合は、このトレイトの実装でメソッドを実装しているため、必要な変更はありません。

<a name="email-verification-route-change"></a>
#### メール確認ルートの変更

**影響の可能性： 中程度**

メール確認ルートは`/email/verify/{id}`から`/email/verify/{id}/{hash}`へ変更しました。Laravel6.xへアップグレードする前のバージョンから送信されたメール確認メールは無効になり、４０４ページが表示されます。望むのであれば、古い認証のURLパスに合うルートを定義し、ユーザーへメールアドレスを再ベリファイするように情報メッセージを表示しましょう。

<a name="helpers"></a>
### ヘルパ

#### 文字列と配列のヘルパパッケージ

**影響の可能性： 高い**

すべての`str_`と`array_`ヘルパは新しい`laravel/helpers` Composerパッケージに移され、フレームワークから削除しました。望みであればこれらのヘルパの呼び出しすべてで、`Illuminate\Support\Str`と`Illuminate\Support\Arr`クラスを使ってください。もしくは、アプリケーションへ新たに`laravel/helpers`パッケージを追加すれば、こうしたヘルパを今までどおり利用できます。

    composer require laravel/helpers

### 多言語化

<a name="trans-and-trans-choice"></a>
#### `Lang::trans`と`Lang::transChoice`メソッド

**影響の可能性： 中程度**

トランスレータの`Lang::trans`と`Lang::transChoice`メソッドは、`Lang::get`と`Lang::choice`へリネームしました。

さらに、`Illuminate\Contracts\Translation\Translator`契約を自分で実装している場合は、その実装の`trans`と`transChoice`を`get`と`choice`へ更新してください。

<a name="get-from-json"></a>
#### `Lang::getFromJson`メソッド

**影響の可能性： 中程度**

`Lang::get`と`Lang::getFromJson`メソッドはまとめました。`Lang::getFromJson`メソッドの呼び出しは、`Lang::get`を呼び出すように更新してください。

> {note} `Lang::transChoice`、`Lang::trans`、`Lang::getFromJson`の削除に関連するBladeエラーを防ぐために、`php artisan view:clear` Artisanコマンドを実行すべきでしょう。

### メール

#### MandrillとSparkPostドライバの削除

**影響の可能性： 低い**

`mandrill`と`sparkpost`メールドライバは削除しました。続けてどちらかのドライバを使用したい場合は、そうしたドライバを提供するコミュニティによりメンテナンスされているパッケージを選び、採用することを推奨します。

### 通知

#### Nexmoルーティングの削除

**影響の可能性： 低い**

しばらく残していましたが、フレームワークのコアからNexmo通知チャンネルを削除しました。Nexmo通知を利用していた方は、[ドキュメントで説明している通りに](/docs/{{version}}/notifications#routing-sms-notifications)、Notifiableエンティティに自分で実装してください。

### パスワードリセット

#### パスワードの検証

**影響の可能性： 低い**

`PasswordBroker`はパスワードの制限やバリデーションを行わなくしました。パスワードのバリデーションはすでに`ResetPasswordController`クラスで行っており、ブローカのバリデーションは冗長でカスタマイズ不可能でした。`PasswordBroker`（もしくは`Password`ファサード）を組み込まれている`ResetPasswordController`以外で使用している場合は、ブローカへ渡す前にすべてのパスワードをバリデートする必要があります。

### キュー

<a name="queue-retry-limit"></a>
#### キュー試行回数制限

**影響の可能性： 中程度**

以前のリリースのLaravelは、`php artisan queue:work`コマンドは無制限にジョブを再試行していました。Laravel6.0からこのコマンドは、デフォルトで１回試行します。強制的に無制限回試行したい場合は、`--tries`オプションに`0`を指定してください。

    php artisan queue:work --tries=0

さらに付け加え、アプリケーションのデータベースへ確実に`failed_jobs`テーブルを持たせてください。このテーブルのマイグレーションは、`queue:failed-table` Artisanコマンドで生成できます。

    php artisan queue:failed-table

### リクエスト

<a name="the-input-facade"></a>
#### `Input`ファサード

**影響の可能性： 中程度**

`Input`ファサードは基本的に`Request`ファサードの重複でしたが、削除しました。`Input::get`メソッドを使用していた場合、`Request::input`メソッドを使ってください。他の`Input`ファサードの呼び出しも、シンプルに`Request`ファサードの使用へ更新できます。

### スケジュール

#### `between`メソッド

**影響の可能性： 低い**

以前のリリースのLaravelでは、スケジューラの`between`メソッドは日付の境界に関して混乱をもたらす動作をしていました。例をご覧ください。

    $schedule->command('list')->between('23:00', '4:00');

ほとんどのユーザーは、このメソッドは23:00から4:00までの間の毎分、`list`コマンドを実行すると期待するでしょう。しかしながら、以前のLaravelのリリースでは、スケジューラは時間を入れ替えて、4:00から23:00までの毎分`list`コマンドを実行していました。Laravel6.0では期待通りに動作します。

### ストレージ

<a name="rackspace-storage-driver"></a>
#### Rackspaceストレージドライバの削除

**影響の可能性： 低い**

`rackspace`ストレージドライバを削除しました。Rackspaceをストレージプロバイダとして使い続けたい場合は、このドライバを提供するコミュニティがメンテナンスしているパッケージを採用することを推奨します。

### URL生成

#### ルートURLジェネレータと追加のパラメータ

以前のリリースのLaravelでは、`route`ヘルパや`URL::route`メソッドへ連想配列のパラメータを渡した場合、パラメータの値にルートパスと一致するキーがない場合であってもルートへのURLを生成する場合、まれにパラメータをURI値として使用していました。Laravel6.0からは、こうした値は代わりにクエリ文字列として付け加えます。例として、以下のルートを考えてください。

    Route::get('/profile/{location}', function ($location = null) {
        //
    })->name('profile');

    // Laravel5.8: http://example.com/profile/active
    echo route('profile', ['status' => 'active']);

    // Laravel6.0: http://example.com/profile?status=active
    echo route('profile', ['status' => 'active']);

`action`ヘルパと`URL::action`メソッドもこの変更の影響を受けます。

    Route::get('/profile/{id}', 'ProfileController@show');

    // Laravel5.8: http://example.com/profile/1
    echo action('ProfileController@show', ['profile' => 1]);

    // Laravel6.0: http://example.com/profile?profile=1
    echo action('ProfileController@show', ['profile' => 1]);

### バリデーション

#### FormRequestの`validationData`メソッド

**影響の可能性： 低い**

リクエストの`validationData`メソッドを`protected`から`public`へ変更しました。皆さんの実装でこのメソッドをオーバーライドしている場合は、`public`の可視性へ変更してください。

<a name="miscellaneous"></a>
### その他

`laravel/laravel`の[GitHubリポジトリ](https://github.com/laravel/laravel)で、変更を確認することを推奨します。これらの変更は必須でありませんが、皆さんのアプリケーションではファイルの同期を保つほうが良いでしょう。変更のいくつかは、このアップグレードガイドで取り扱っていますが、設定ファイルやコメントなどの変更は取り扱っていません。変更は簡単に[GitHubの比較ツール](https://github.com/laravel/laravel/compare/5.8...6.x)で閲覧でき、みなさんにとって重要な変更を選択できます。
