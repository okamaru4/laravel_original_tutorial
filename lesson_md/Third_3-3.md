# PHPUnitの実装 2

- 前回は、テストを実行するのに最低限必要な設定とちょっとしたテストを実装しました。
- 今回は、アプリケーション全体にテストが行えるようにしたいと思います。


## `Feature` のテストを仕上げていく

先に画面が問題なく表示されること(問題なくhttp通信がされていることを意味します)をテストする実装の記述を行います。。

対象file `tests/Feature/TodoTest.php`

```php
    /** @test */
    public function storeTest()
    {
        $this->post('todo/', 
            ['title' => "foo"]
        );
        $this->assertDatabaseHas('todos', ['title' => "foo"]);
    }
    
    /** @test */
    public function editTest()
    {
        $response = $this->get('todo/1/edit');
        $response->assertStatus(200);
    }
    
    /** @test */
    public function updateTest()
    {
        $this->put('todo/1',
                   ['title' => 'updateData']
        );
        $this->assertDatabaseHas('todos', ['title' => 'updateData']);
    }
    
    /** @test */
    public function destroyTest()
    { 
        $this->delete('/todo/1');
        $this->assertDatabaseMissing('todos', ['id' => 1]);
    }
```

現状のControllerに対してhttp通信を介するテストは、これで全て行えたと思います。

次に行うのがControllerのメソッド内や機能がちゃんと動いているかどうかのテストとなります。


## 'Unit' へのテストの実装

ひどく曖昧になりそうな部分ではあるが`Feature` へのテストは、概ね通信が問題なく行われるかどうかなどに主を置いています。なので機能的な点に関してを`Unit` で書いていくという流れになります。ただ曖昧な点もありますので御了承下さい。


- イメージ

どこに対してのなんのテストを行うかですがイメージとしては、`Controller` で行なっていることが問題なく行えるかという点にまずは、着目したいです。

今回のアプリでは、テストする項目が少ないので`Controller` にてInstanceの作成が問題なく行えることを確認したいと思いますが実際に`Controller` に対してのテストを行うわけではありません。

`Facade` のやり方と`new` でのやり方どちらでやっても同じ挙動になってもらいたいので比較します。


`tests/Unit/TodoTest.php`
```php
<?php

namespace Tests\Unit;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Todo;

class TodoTest extends TestCase
{
    // 省略 以下追記
    /** @test */
    public function todoModelInstanceTest()
    {
        $this->assertInstanceOf(Todo::class, new Todo());
    }

}
```

インスタンスとして問題なく生成されているかを確認することができます。

実際`Controller` での記載は、異なりますが結果論的には、これで問題ない挙動になっているかどうかの確認が行えるはずです。

テストを実行すると問題なくパスすることが確認できるかと思います。


これでおおよそのテスト自体は、できました。
それぞれのメソッドや実際に`DB` への処理が行われているかどうかなどの処理のテストもかけてます。
がこれは、あくまで最低限での記述になっているため細く追うと至らぬ点もあると思いますが入門なら問題ないと思います。

