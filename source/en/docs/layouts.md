---
title: Screen layouts
description: Displaying the appearance of user interface elements in an application is important
extends: _layouts.documentation.en
section: main
---

The display of the appearance of the user interface elements in the application is of great importance, makes the application
is easier to use and helps users intuitively display screen elements to complete their tasks.

The separation of logic and presentation is one of the principles of development with ORCHID.
One of the elements of the presentation are "Layouts" (layouts) that can be displayed in different variations, if you try to explain briefly, it turns out that this is the `view` on steroids.

## Approach through layouts

For the formation of the page, we in most cases we use the same type of elements, for example, we will present a block that displays the name, signature and avatar of the profile:

```php
<div class="d-sm-flex flex-row flex-wrap text-center text-sm-left align-items-center">
	<span class="thumb-sm avatar m-r-xs">
        <img src="/avatar/maria.jpg" class="bg-light" alt="Maria">
    </span>
    <div class="ml-sm-3 ml-md-0 ml-xl-3 mt-2 mt-sm-0 mt-md-2 mt-xl-0">
        <h6 class="mb-0">Maria</h6>
        <p class="text-muted mb-1">maria@exaple.com</p>
    </div>
</div>
```

A simple display of a block with a profile can appear on dozens of pages and, if they are copied, it can take a long time to maintain their appearance; therefore, various reuse options are worked out. This is called the component approach, regardless of the delivery method and level of responsibility, practiced in both `Blade` and `React/Vue/Angular`.

It is from these components that the layers of the platform are made up, the only difference is that it is necessary to operate with classes, creating which you explicitly determine that the adopted parameter `avatar` will be inserted in the `<img>` tag without having to edit the source code each time.

## Table

Table layout is used to display minimum information for viewing and sampling.

```php
php artisan orchid:table PatientListLayout
```

Example:
```php
namespace App\Layouts\Clinic\Patient;

use Orchid\Screen\TD;
use Orchid\Screen\Layouts\Table;

use Orchid\Platform\Http\Filters\SearchFilter;
use App\Http\Filters\LastNamePatient;

class PatientListLayout extends Table
{

    /**
     * @var string
     */
    public $data = 'patients';

    /**
     * @return array
     */
    public function fields() : array
    {
        return [
            TD::set('last_name','Last name')
                ->align('center')
                ->width('100px')
                ->render(function ($patient) {
                    return '<a href="' . route('platform.clinic.patient.edit',
                            $patient->id) . '">' . $patient->last_name . '</a>';
                }),

            TD::name('first_name')
                ->title('First Name')
                ->sort()
                ->link('platform.clinic.patient.edit', 'id'),
                
            TD::set('phone','Phone')
                ->loadModalAsync('oneAsyncModal', 'savePhone', 'id', 'phone'),
                
            TD::set('email','Email'),
                
            TD::set('created_at','Date of publication'),
               
        ];
    }
}
```

### Allow method

- `align()` horizontal text alignment, takes the values: 'left', 'center', 'right'

- `link($route,$key)` adds a link to the cell, for example, to edit this entry.

- `locale()` displaying column data according to the current locale language.

- `loadModalAsync($modal, $method, $options, $text)` adds a modal window to each column cell. Where $ modal attributes are the name of the modal window, $ method is the method (function) that sends data through the POST request, $ options additional attributes, for example, id or slug, $ text is the displayed value of the cell.

- `name($key)` sets the name of the key from the array of values to display in the table.

- `set($key, $name)` the main method sets the name of the key from the array and the display name.

- `render(function ($item) { return $item->id})` possibility of cell generation according to function. The data of the current line is transferred to $item.

- `sort()` Adds the ability to sort by this column ([See HTTP filtering and sorting](https://orchid.software/en/docs/filters)) in the header.

- `width()` explicitly sets the width of the column `width('100px')`

### Column Extension

Working with the same type of data, it is often required to process them in the same way, in order not to duplicate the code in the layers, it is possible to extend the `TD` class using own methods, for this you need to register the closure function in the service provider.

Registration example:

```php
// AppServiceProvider.php
TD::macro('bool', function () {

    $column = $this->column;

    $this->render(function ($datum) use ($column) {
        return view('bool',[
            'bool' => $datum->$column
        ]);
    });

    return $this;
});
```

Template example:
```php
// bool.blade.php

@if($bool)
    <i class="icon-check text-success"></i>
@else
    <i class="icon-close text-danger"></i>
@endif
```

Example of use:
```php
public function grid(): array
{
    return [
        TD::set('status')->bool(),
    ];
}
```


## Rows

Line layout is the minimum set that is most commonly used.
His goal is to combine all the required fields.

To create, execute the command:

```php
php artisan orchid:rows PatientFirstRows
```

Example:
```php
namespace App\Layouts\Clinic\Patient;

use App\Http\Widgets\AppointmentTypes;
use Orchid\Screen\Field;
use Orchid\Platform\Layouts\Rows;

class Appointment extends Rows
{

    /**
     * @return array
     *
     * @throws \Orchid\Press\EntityTypeException
     */
    public function fields(): array
    {
        return [

            DateTimer::make()
                ->name('appointment_time')
                ->required()
                ->title('Time'),

            Relationship::make()
                ->name('appointment_type')
                ->required()
                ->title('Appointment type')
                ->handler(AppointmentTypes::class),

            TextArea::make()
                ->name('doctor_notes')
                ->rows(10)
                ->required()
                ->title('Doctor notes')
                ->help('What did the patient complain about?'),

        ];
    }
}
```

The strings support short writing without creating a separate class,
for example, when you want to show one or two fields

```php
public function layout(): array
{
    return [
        Layout::rows([
           Input::make('example')
                ->type('text')
                ->title('Example')
        ]),
    ];
}
```


## Modal windows

```php
public function layout(): array
{
    return [
        Layout::modal('exampleModal', [
	    Layout::rows([]),
        ]),
    ];
}
```

Modal windows have properties, size and button names that can be changed:

```php
Layout::modal('exampleModals', [
    Layout::rows([]),
])
    ->title('Title window')
    ->size(Modal::SIZE_LG)
    ->applyButton(self::APPLY_BUTTON)
    ->closeButton(self::CLOSE_BUTTON),
```

## Charts

Graphic layout is a convenient way to graphically display the dynamics of values, but it requires some
data processing example data from `query`

```php
public function query($patient = null) : array
{
    $charts = [
        [
            'title'  => "Some Data",
            'values' => [25, 40, 30, 35, 8, 52, 17, -4],
        ],
        [
            'title'  => "Another Set",
            'values' => [25, 50, -10, 15, 18, 32, 27, 14],
        ],
        [
            'title'  => "Yet Another",
            'values' => [15, 20, -3, -15, 58, 12, -17, 37],
        ],
    ];
    
    return [
        'charts' => $charts,
    ];
}
```

To create, run the command:
```php
php artisan orchid:chart ChartsLayout
```

Example:
```php
namespace App\Layouts\Clinic\Patient;

use Orchid\Platform\Layouts\Chart;

class ChartsLayout extends Chart
{

    /**
     * @var string
     */
    public $title = 'DemoCharts';

    /**
     * @var int
     */
    public $height = 150;

    /**
     * Available options:
     * 'bar', 'line', 
     * 'pie', 'percentage'
     *
     * @var string
     */
    public $type = 'scatter';

    /**
     * @var array
     */
    public $labels = [
        "12am-3am",
        "3am-6am",
        "6am-9am",
        "9am-12pm",
        "12pm-3pm",
        "3pm-6pm",
        "6pm-9pm",
        "9pm-12am",
    ];

    /**
     * @var string
     */
    public $data = 'charts';
}
```


## Filter Set

To group filters, their demand and use, there is a separate layer `Selection`,
in which they are listed.

To create, execute the command:

```php
php artisan orchid:selection MySelection
```

Пример класса:
```php
namespace App\Orchid\Layouts;

use Orchid\Platform\Filters\Filter;
use Orchid\Press\Http\Filters\CreatedFilter;
use Orchid\Press\Http\Filters\SearchFilter;
use Orchid\Screen\Layouts\Selection;

class MySelection extends Selection
{
    /**
     * @return Filter[]
     */
    public function filters(): array
    {
        return [
          SearchFilter::class,
          CreatedFilter::class
        ];
    }
}
```


## Tabs

Tabs support short syntax by calling a static method,
which does not require the creation of a separate class:

```php
public function layout(): array
{
    return [
        Layout::tabs([
            'Example Tab Table' => TableExample::class,
            'Example Tab Rows'  => RowExample::class,
        ]),
    ];
}
```

The name of the tabs will match the keys of the array

## Columns

Similar to tabs:

```php
public function layout(): array
{
    return [
        Layout::columns([
           TableExample::class,
           RowExample::class,
        ]),
    ];
}
```


## Drop-down list


```php
public function layout(): array
{
    return [
        Layout::collapse([
            Input::make('name')
                ->type('text')
                ->title('Name Articles')
        ])->label('More'),
    ];
}
```

## Accordion

```php
public function layout(): array
{
    return [
        Layout::accordion([
            'Personal Information' => [
                Layout::rows([
                    Input::make('user.name')
                        ->type('text')
                        ->required()
                        ->title('Name')
                        ->placeholder('Name'),

                    Input::make('user.email')
                        ->type('email')
                        ->required()
                        ->title('Email')
                        ->placeholder('Email'),
                ]),
            ],
            'Billing Address'      => [
                Layout::rows([
                    Input::make('address')
                        ->type('text')
                        ->required()
                ]),
            ],
        ]),
    ];
}
```

## Custom Template


It is expected situation when you need to display your own template,
for this:


```php
public function layout(): array
{
    return [
        Layout::view('myTemplate'),
    ];
}
```

All data from the `query` method will be passed to your template.


## Wrapper

The “Wrap” can serve as an intermediary between the “User Pattern” and standard layers, with which
available indicate exactly where the other layers should be displayed.

```php
public function layout(): array
{
    return [
        Layout::wrapper('myTemplate', [
            'foo' => [
                RowLayout::class,
                RowLayout::class,
            ],
            'bar' => RowLayout::class,
        ]),
    ];
}
```

In the `myTemplate` template, the variables` foo` and `bar` will be transferred, which contain the already compiled` View`, which can be displayed:

```html
<div class="row">
    <div class="col-7 border-right">
        @foreach($foo as $row)
            {!! $row !!}
        @endforeach
    </div>
    <div class="col-5 no-gutter">
        {!! $bar !!}
    </div>
</div>
```

Variables from `query` are also available in the template.


## Layer Expansion

The `Layouts` class is grouping several different ones; in order to add a new feature to it, it is enough to specify it in the service provider as:

```php
use Orchid\Screen\Layouts\Base;
use Orchid\Screen\Repository;
use Orchid\Screen\Layout;

Layout::macro('hello', function (string $name) {
    return new class($name) extends Base
    {
        /**
         * @ string
         */
        public $name;

        /**
         * Hello constructor.
         *
         * @param string $name
         */
        public function __construct(string $name)
        {
            $this->name = $name;
        }

        /**
         * @param Repository $repository
         *
         * @return mixed
         */
        public function build(Repository $repository)
        {
            return view('hello')->with('name', $this->name);
        }

    };
});
```

Then on the screen the call will look like:

```php
public function layout(): array
{
    return [
        Layout::hello('Alexandr Chernyaev')
    ];
}
```
