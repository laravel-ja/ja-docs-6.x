# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel6.0](#laravel-6.0)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravelとファーストパーティパッケージは、[セマンティックバージョニング](https://semver.org)に従っています。メジャーなフレームのリリースは、２月と８月の半年ごとにリリースされます。マイナーとパッチリリースはより細かく毎週リリースされます。マイナーとパッチリリースは、**決して**ブレーキングチェンジを含みません。

皆さんのアプリケーションやパッケージからLaravelフレームワークかコンポーネントを参照する場合は、Laravelのメジャーリリースはブレーキングチェンジを含まないわけですから、`^6.0`のようにバージョンを常に指定してください。しかし、メジャーリリースへ１日以内でアップデートできるように、私たちは常に努力しています。

<a name="support-policy"></a>
## サポートポリシー

Laravel6.0のようなLTSリリースでは、バグフィックスは２年間、セキュリティフィックスは３年間提供します。これらのリリースは長期間に渡るサポートとメンテナンスを提供します。 一般的なリリースでは、バグフィックスは６ヶ月、セキュリティフィックスは１年です。Lumenのようなその他の追加ライブラリでは、最新リリースのみでバグフィックスを受け付けています。

| バージョン | リリース | バグフィックス期限 | セキュリティフィックス期限 |
| --- | --- | --- | --- |
| 5.5 (LTS) | ２０１７年８月３０日 | ２０１９年８月３０日 | ２０２０年８月３０日 |
| 5.6 | ２０１８年２月７日 | ２０１８年８月７日 | ２０１９年２月７日 |
| 5.7 | ２０１８年９月４日 | ２０１９年３月４日 | ２０１９年９月４日 |
| 5.8 | ２０１９年２月２６日 | ２０１９年８月２６日 | ２０２０年２月２６日 |
| 6.0 (LTS) | ２０１９年９月３日 | ２０２１年９月３日 | ２０２２年９月３日 |

<a name="laravel-6.0"></a>
## Laravel6.0

Laravel6.0LTSは、Laravel5.8で行われた向上に加え、以降の変更で構成されています。セマンティックバージョニングの導入、[Laravel Vapor](https://vapor.laravel.com)とのコンパチビリティ、認可レスポンスの向上、ジョブミドルウェア、レージーコレクション、サブクエリの向上、フロントエンドスカフォールドを`laravel/ui` Composerパッケージへ移行、ならびに多くのバグフィックスとユーザービリティの向上です。

### セマンティックバージョニング

Laravelフレームワーク(`laravel/framework`)パッケージが、[セマンティックバージョニング](https://semver.org/)規約に準拠しました。すでにこのバージョニング規約に従っている他のLaravelファーストパーティパッケージと統一されました。Laravelのリリースサイクルには変化はありません。

### Laravel Vaporコンパチビリティ

**Laravel Vaporは[Taylor Otwell](https://github.com/taylorotwell)が構築しています。**

Laravel6.0は、Laravelのためのオートスケールされるサーバレス環境のプラットフォームである[Laravel Vapor](https://vapor.laravel.com)とのコンパチビリティを提供しました。AWS Lambda上のLaravelアプリケーション管理の複雑さを抽象化し、同時にアプリケーションをSQSキュー、データベース、Redisクラスタ、ネットワーク、CloudFront CDN、その他と接続します。

### Ignitionによる例外表示の向上

Laravel6.0は、Freek Van der HertenとMarcel Pociotによる、例外詳細ページのオープンソースである[Ignition](https://github.com/facade/ignition)を採用しています。Ignitionは、Bladeエラーファイルと行番号の処理、一般的な問題に対する実行可能な解決策、コード編集、例外の共有、UXの向上などたくさんの利便性を以前のリリースから提供しています。

### 認可レスポンスの向上

**認可レスポンスの向上は、[Gary Green](https://github.com/garygreen)により行われました。**

以前のリリースのLaravelは、エンドユーザーに対しカスタム認可メッセージを取得し、表示するのが困難でした。これにより、エンドユーザーになぜ特定のリクエストが拒否されるのかをしっかりと説明するのも困難でした。Laravel6.0では認可レスポンスメッセージと新しい`Gate::inspect`メソッドを使うことでより簡単にできるようになりました。例として、以下のポリシーメソッドをご覧ください。

    /**
     * このユーザーが指定したフライトを見ることができるか判定
     *
     * @param  \App\User  $user
     * @param  \App\Flight  $flight
     * @return mixed
     */
    public function view(User $user, Flight $flight)
    {
        return $this->deny('Explanation of denial.');
    }

認可ポリシーのレスポンスとメッセージは`Gate::inspect`メソッドで簡単に取得できます。

    $response = Gate::inspect('view', $flight);

    if ($response->allowed()) {
        // ユーザーはこのフライトを見る許可がある
    }

    if ($response->denied()) {
        echo $response->message();
    }

さらに、ルートやコントローラで`$this->authorize`か`Gate::authorize`のようなヘルパメソッドを使用すると、このカスタムメッセージは自動的にフロントエンドに返されます。

### ジョブミドルウェア

**ジョブミドルウェアは、[Taylor Otwell](https://github.com/taylorotwell)により実装されました。**

ジョブミドルウェアはキュー済みジョブの実行周りのカスタムロジックをラップできるようにし、ジョブ自身の定形コードを減らします。たとえば、以前のLaravelリリースでは、ジョブの`handle`メソッドのロジックで、レート制限コールバックをラップしていました。

    /**
     * ジョブの実行
     *
     * @return void
     */
    public function handle()
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // ジョブの処理…
        }, function () {
            // ロックが取得できない…

            return $this->release(5);
        });
    }

Laravel6.0ではこのロジックをジョブミドルウェアへ移すことができ、レート制限の責任を`handle`メソッドは全く負わなくて済みます。

    <?php

    namespace App\Jobs\Middleware;

    use Illuminate\Support\Facades\Redis;

    class RateLimited
    {
        /**
         * キュージョブの実行
         *
         * @param  mixed  $job
         * @param  callable  $next
         * @return mixed
         */
        public function handle($job, $next)
        {
            Redis::throttle('key')
                    ->block(0)->allow(1)->every(5)
                    ->then(function () use ($job, $next) {
                        // ロック取得…

                        $next($job);
                    }, function () use ($job) {
                        // ロックが取得できない…

                        $job->release(5);
                    });
        }
    }

ミドルウェアを作成したら、ジョブの`middleware`メソッドから返すことで指定できます。

    use App\Jobs\Middleware\RateLimited;

    /**
     * このジョブが通過する必要のあるミドルウェアの取得
     *
     * @return array
     */
    public function middleware()
    {
        return [new RateLimited];
    }

### レージーコレクション

**レージーコレクションは、[Joseph Silber](https://github.com/JosephSilber)により改善されました。**

すでに多くの開発者がLaravelの強力な[コレクションメソッド](https://laravel.com/docs/collections)を楽しんで利用しています。すでに強力な`Collection`クラスを補足するために、`LazyCollection`クラスはPHPの[PHPジェネレータ](https://www.php.net/manual/ja/language.generators.overview.php)を活用しています。巨大なデータセットをメモリ使用を抑えて利用する目的のためです。

たとえば、アプリケーションで数ギガバイトのログを処理する必要があり、ログを解析するためにLaravelのコレクションメソッドを活用するとしましょう。ファイル全体をメモリに一度に読み込む代わりに、レイジーコレクションなら毎回ファイルの小さな部分だけをメモリに保持するだけで済みます。

    use App\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })
    ->chunk(4)
    ->map(function ($lines) {
        return LogEntry::fromLines($lines);
    })
    ->each(function (LogEntry $logEntry) {
        // ログエントリーの処理…
    });

もしくは、10,000個のEloquentモデルを繰り返し処理する必要があると想像してください。今までのLaravelコレクションでは、一度に10,000個のEloquentモデルすべてをメモリーにロードする必要がありました。

    $users = App\User::all()->filter(function ($user) {
        return $user->id > 500;
    });

しかし、Laravel6.0からクエリビルダの`cursor`メソッドは、`LazyCollection`インスタンスを返します。これによりデータベースに対し１つのクエリを実行するだけでなく、一度に１つのEloquentモデルをメモリにロードするだけで済みます。この例では、各ユーザーを個別に繰り返し処理するまで`filter`コールバックは実行されず、大幅にメモリ使用量を減らせます。

    $users = App\User::cursor()->filter(function ($user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

### Eloquentサブクエリの向上

**Eloquentサブクエリの向上は、[Jonathan Reinink](https://github.com/reinink)により行われました。**

Laravel6.0はデータベースサブクエリのサポートにも、多くの新しい改善と向上を導入しました。例として、フライト（`flights`）と目的地（`destinations`）テーブルを想像してください。`flights`テーブルは、フライトの目的地への到着時間を意味する`arrived_at`カラムを持っています。

Laravel6.0の新しいサブクエリSELECTの機能を使えば、全`destinations`と、一番早く目的地へ到着するのフライト名を１回のクエリで取得できます。

    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();

さらに、クエリビルダの`orderBy`関数も新しいサブクエリ機能ををサポートしています。この機能を使い、ラストフライトが目的地へいつ到着するかに基づいて全目的地をソートしてみましょう。今回も、これによりデータベースに対し１回のクエリしか実行されません。

    return Destination::orderByDesc(
        Flight::select('arrived_at')
            ->whereColumn('destination_id', 'destinations.id')
            ->orderBy('arrived_at', 'desc')
            ->limit(1)
    )->get();

### Laravel UI

以前のバージョンのLaravelでは通常フロントエンドを提供していましたが、`laravel/ui` Composerパッケージとして分離されました。これにより、フレームワーク本体より独立し、ファーストパーティのUIスカフォールドとして開発できるようになり、バージョニングも分けることができました。この変更の結果、デフォルトのフレームワークスカフォールドにはBootstrapやVueのコードが無くなり、`make:auth`コマンドも同様にフレームワークから分離しました。

以前のLaravelリリースで提供していたVue／Bootstrapのスカフォールドに戻すには、`laravel/ui`パッケージをインストールし、`ui` Artisanコマンドを使用してフロントエンドスカフォールドをインストールします。

    composer require laravel/ui --dev

    php artisan ui vue --auth
