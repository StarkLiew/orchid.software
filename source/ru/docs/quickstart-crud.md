---
title: Управление данными
description: Laravel CRUD
extends: _layouts.documentation.ru
section: main
---

Это руководство представляет минимальный набор для создания и редактирования моделей с помощью платформы. В текущем примере мы создадим страницы администрирования для блога.

На этом этапе желательно, что бы вы уже познакомились с основами концепции [экранов](https://orchid.software/ru/docs/screens) и просмотрели "[Быстрый старт для начинающих](https://orchid.software/ru/docs/quickstart)".

Для начала нам необходимо создать новую таблицу для этого выполним команду:

```php
php artisan make:migration create_posts_table
```

В директории `database/migrations` будет создан новый файл миграции, добавим в него описание требуемых колонок.

```php
// `****_**_**_create_posts_table.php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePostsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('title');
            $table->string('description');
            $table->text('body');
            $table->bigInteger('author');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

Для применения новой схемы к подключенной базе данных выполним:

```php
php artisan migrate
```

После успешного создания таблицы, добавим новую `Eloquent` модель, для этого выполним:

```php
php artisan make:model Post
```

В директории `app` будет создан новый файл `Post.php` опишем поля как доступные для заполнения:

```php
// app/Post.php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Orchid\Screen\AsSource;

class Post extends Model
{
    use AsSource;

    /**
     * @var array
     */
    protected $fillable = [
        'title',
        'description',
        'body',
        'author'
    ];
}
```

> **Примечание.** Модель имеет трейд `AsSource`, для удобного обращения через dot нотацию.

Теперь мы готовы к настоящему использованию платформы. 

[В предыдущем пособии](https://orchid.software/ru/docs/quickstart) мы уже создавали наш первый экран для отправки email сообщений, но теперь нам необходимо как отображать записи, так и редактировать их, по этому добавим два новых экрана на каждое действие, поочередно выполнив команды:

```php
php artisan orchid:screen PostEditScreen
php artisan orchid:screen PostListScreen
```

> **Примечание.** Создание двух и более экранов для обеспечения CRUD не обязательно, но используется для удобства.

Зарегистрируем новые экраны в маршрутном листе приложения:

```php
// routes/platform.php

use App\Orchid\Screens\PostEditScreen;
use App\Orchid\Screens\PostListScreen;

$this->router->screen('post/{post?}', PostEditScreen::class)
    ->name('platform.post.edit');
    
$this->router->screen('posts', PostListScreen::class)
    ->name('platform.post.list');
```

Теперь экраны доступны для просмотра по адресу `/dashboard/post` и `/dashboard/posts`.
Полученные экраны на текущий момент не имеют, ни данных, ни действий, отредактируем `PostEditScreen` для просмотра добавив название, описание и требуемые поля:

```php
namespace App\Orchid\Screens;

use App\Post;
use App\User;
use Illuminate\Http\Request;
use Orchid\Screen\Fields\Input;
use Orchid\Screen\Fields\Quill;
use Orchid\Screen\Fields\Relation;
use Orchid\Screen\Fields\TextArea;
use Orchid\Screen\Fields\Upload;
use Orchid\Screen\Layout;
use Orchid\Screen\Link;
use Orchid\Screen\Screen;
use Orchid\Support\Facades\Alert;

class PostEditScreen extends Screen
{
    /**
     * Display header name.
     *
     * @var string
     */
    public $name = 'Creating a new post';

    /**
     * Display header description.
     *
     * @var string
     */
    public $description = 'Blog posts';

    /**
     * @var bool
     */
    public $exists = false;

    /**
     * Query data.
     *
     * @param Post $post
     *
     * @return array
     */
    public function query(Post $post): array
    {
        $this->exists = $post->exists;

        if($this->exists){
            $this->name = 'Edit post';
        }

        return [
            'post' => $post
        ];
    }

    /**
     * Button commands.
     *
     * @return Link[]
     */
    public function commandBar(): array
    {
        return [
            Link::name('Create post')
                ->icon('icon-pencil')
                ->method('createOrUpdate')
                ->canSee(!$this->exists),

            Link::name('Update')
                ->icon('icon-note')
                ->method('createOrUpdate')
                ->canSee($this->exists),

            Link::name('Remove')
                ->icon('icon-trash')
                ->method('remove')
                ->canSee($this->exists),
        ];
    }

    /**
     * Views.
     *
     * @return Layout[]
     */
    public function layout(): array
    {
        return [
            Layout::rows([
                Input::make('post.title')
                    ->title('Title')
                    ->placeholder('Attractive but mysterious title'),

                TextArea::make('post.description')
                    ->title('Description')
                    ->rows(3)
                    ->maxlength(200)
                    ->placeholder('Brief description for preview'),

                Relation::make('post.author')
                    ->title('Author')
                    ->fromModel(User::class, 'name'),

                Quill::make('post.body')
                    ->title('Main text'),

            ])->with(75)
        ];
    }

    /**
     * @param Post    $post
     * @param Request $request
     *
     * @return \Illuminate\Http\RedirectResponse
     */
    public function createOrUpdate(Post $post, Request $request)
    {
        $post->fill($request->get('post'))->save();

        Alert::info('You have successfully created an post.');

        return redirect()->route('platform.post.list');
    }

    /**
     * @param Post $post
     *
     * @return \Illuminate\Http\RedirectResponse
     * @throws \Exception
     */
    public function remove(Post $post)
    {
        $post->delete()
            ? Alert::info('You have successfully deleted the post.')
            : Alert::warning('An error has occurred')
        ;

        return redirect()->route('platform.post.list');
    }
}
```

Теперь мы можем создавать, редактировать и удалять записи. Но не просматривать в виде списка, изменим это!

Во всех предыдущих определениях слоёв использовалась только короткая запись вида `Layout:rows()`, но для отображения сложной или объемной информации желательно создавать отдельные классы.

Добавим новый класс `Layout`, выполнив команду:

```php
php artisan orchid:table PostListLayout
```

Укажем какие данные мы ходим видеть:

```php
namespace App\Orchid\Layouts;

use App\Post;
use Orchid\Screen\TD;
use Orchid\Screen\Layouts\Table;

class PostListLayout extends Table
{
    /**
     * Data source.
     *
     * @var string
     */
    public $data = 'posts';

    /**
     * @return TD[]
     */
    public function fields(): array
    {
        return [
            TD::set('title', 'Title')
                ->render(function (Post $post) {
                    // Please use view('path')
                    $route = route('platform.post.edit', $post);
                    $title = e($post->title);

                    return "<a href='{$route}'>{$title}</a>";
                }),
            
            TD::set('created_at', 'Created'),
            TD::set('updated_at', 'Last edit'),
        ];
    }
}
```

Свойсвто `data` указывает ключ, что будет источником для нашей таблице на экране.

> **Примечание.** Формирование `HTML` непосредственно в классе необходимо только для примера и является плохой практикой. Пожалуйста используйте `Blade` шаблоны.

Определив слой таблицы, вернемся к экрану просмотра, изменим его :

```php
namespace App\Orchid\Screens;

use App\Orchid\Layouts\PostListLayout;
use App\Post;
use Orchid\Screen\Link;
use Orchid\Screen\Screen;

class PostListScreen extends Screen
{
    /**
     * Display header name.
     *
     * @var string
     */
    public $name = 'Blog post';

    /**
     * Display header description.
     *
     * @var string
     */
    public $description = 'All blog posts';

    /**
     * Query data.
     *
     * @return array
     */
    public function query(): array
    {
        return [
            'posts' => Post::paginate()
        ];
    }

    /**
     * Button commands.
     *
     * @return Link[]
     */
    public function commandBar(): array
    {
        return [
            Link::name('Create new')
                ->icon('icon-pencil')
                ->link(route('platform.post.edit'))
        ];
    }

    /**
     * Views.
     *
     * @return Layout[]
     */
    public function layout(): array
    {
        return [
            PostListLayout::class
        ];
    }
}
```

Теперь мы набор для управления(создавать, просматривать, редактировать и удалять записи) блогом, не забудьте позаботиться об удобстве навигации из предыдущего материала добавив пункты меню и хлебные крошки.


