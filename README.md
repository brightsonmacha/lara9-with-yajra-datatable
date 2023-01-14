<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

## About

Demostrating the usage for Yajra Datatable for display hundreds of records from database with serve side processing.

## Stack

- [Laravel V9.X](https://laravel.com/docs).
- [Yajra Datatable](https://yajrabox.com/docs/laravel-datatables/10.0/quick-starter).

- Front end [Laravel UI, Laravel Auth](https://github.com/laravel/ui).


## Steps

- Using Composer global Laravel Installer, create new Laravel project.
```
laravel new datatables
```
- Bootstrap Laravel UI.
```
composer require laravel/ui --dev
php artisan ui bootstrap --auth
```
- Install Yajra Datatable.
```
composer require yajra/laravel-datatables:^9.0
```


- Install Laravel DataTables Vite
Next, we will install Laravel DataTables Vite to simplify our frontend setup.
```
npm i laravel-datatables-vite --save-dev
``` 

This will install the following packages:
 `Bootstrap Icons`, `DataTables with Buttons and Select plugins for Bootstrap 5`, `Laravel DataTables custom scripts`

- Configure scripts and css needed for application.
<br>
```
resources/js/app.js
import './bootstrap';
import 'laravel-datatables-vite';
```

```
resources/sass/app.scss
// Fonts
@import url('https://fonts.bunny.net/css?family=Nunito');
 
// Variables
@import 'variables';
 
// Bootstrap
@import 'bootstrap/scss/bootstrap';
 
// DataTables
@import 'bootstrap-icons/font/bootstrap-icons.css';
@import "datatables.net-bs5/css/dataTables.bootstrap5.min.css";
@import "datatables.net-buttons-bs5/css/buttons.bootstrap5.min.css";
@import 'datatables.net-select-bs5/css/select.bootstrap5.css';
```

Start the Vite development server to automatically recompile our JS, CSS and refresh the browser when we make changes to Blade templates:

```
npm run dev
```

- Setup a Users DataTable. In project terminal directory run the following command:
```
php artisan datatables:make Users
```
Next, we will configure our UsersDataTable and add the columns that we want to display.

```
namespace App\DataTables;

use App\Models\User;
use Illuminate\Database\Eloquent\Builder as QueryBuilder;
use Yajra\DataTables\EloquentDataTable;
use Yajra\DataTables\Html\Builder as HtmlBuilder;
use Yajra\DataTables\Html\Button;
use Yajra\DataTables\Html\Column;
use Yajra\DataTables\Html\Editor\Editor;
use Yajra\DataTables\Html\Editor\Fields;
use Yajra\DataTables\Services\DataTable;

class UsersDataTable extends DataTable
{
    public function dataTable(QueryBuilder $query): EloquentDataTable
    {
        return (new EloquentDataTable($query))
            ->addColumn('action', 'users.action')
            ->setRowId('id');
    }

    public function query(User $model): QueryBuilder
    {
        return $model->newQuery();
    }

    
    public function html(): HtmlBuilder
    {
        return $this->builder()
                    ->setTableId('users-table')
                    ->columns($this->getColumns())
                    ->minifiedAjax()
                    //->dom('Bfrtip')
                    ->orderBy(1)
                    ->selectStyleSingle()
                    ->buttons([
                        Button::make('excel'),
                        Button::make('csv'),
                        Button::make('pdf'),
                        Button::make('print'),
                        Button::make('reset'),
                        Button::make('reload')
                    ]);
    }


    public function getColumns(): array
    {
        return [
            Column::computed('action')
                  ->exportable(false)
                  ->printable(false)
                  ->width(60)
                  ->addClass('text-center'),
            Column::make('id'),
            Column::make('name'),
            Column::make('email'),
            Column::make('created_at'),
            Column::make('updated_at'),
        ];
    }


    protected function filename(): string
    {
        return 'Users_' . date('YmdHis');
    }
}

```


- Setup a Users Controller, View & Route
```
php artisan make:controller UsersController
```

```
app/Http/Controllers/UsersController.php
namespace App\Http\Controllers;
 
use App\DataTables\UsersDataTable;
 
class UsersController extends Controller
{
    public function index(UsersDataTable $dataTable)
    {
        return $dataTable->render('users.index');
    }
}

```


```
resources/views/users/index.blade.php
@extends('layouts.app')
 
@section('content')
    <div class="container">
        <div class="card">
            <div class="card-header">Manage Users</div>
            <div class="card-body">
                {{ $dataTable->table() }}
            </div>
        </div>
    </div>
@endsection
 
@push('scripts')
    {{ $dataTable->scripts(attributes: ['type' => 'module']) }}
@endpush

```

```
routes/web.php
use App\Http\Controllers\UsersController;
 
Route::get('/users', [UsersController::class, 'index'])->name('users.index');

```

- Update the default app layout. To be able to load our custom scripts, we need to add `@stack('scripts')` before the end of body tag in our app.blade.php layout.

```
resources/views/layouts/app.blade.php
....
        <main class="py-4">
            @yield('content')
        </main>
    </div>
    @stack('scripts')
</body>
</html>
```


- Migrate and Seed Test Data
```
php artisan migrate
```
Seed data to your database using UserFactory, edit as below

```
<?php

namespace Database\Seeders;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        \App\Models\User::factory(12000)->create();
    }
}

```

Then run
```
php artisan db:seed
```

Our application should now be ready to run.

```
php artisan serve --=port=8085
```
Once you have started the Artisan development server, your application will be accessible in your web browser at `http://localhost:8085`

Now visit to see the results. <br>
`http://localhost:8085/users`
