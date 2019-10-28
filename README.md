# Laravel Simple CRUD Trait

Um exemplo simples de crud genérico.

# Instalação

Crie um arquivo em `app/Http/Controllers/Helpers/SimpleCrud.php` com o seguinte conteúdo:

```
<?php

namespace App\Http\Controllers\Helpers;

use Illuminate\Http\Request;

trait SimpleCrud
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $this->beforeAction(__FUNCTION__);

        $results = $this->model->paginate();
        return view($this->view . '.index', compact('results'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        $this->beforeAction(__FUNCTION__);

        return view($this->view . '.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $this->beforeAction(__FUNCTION__);

        $request->validate($this->model->rules());
        $result = $this->model->create($request->all());
        return redirect()->route($this->route . '.edit', $result->id);
    }

    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        $this->beforeAction(__FUNCTION__);

        $result = $this->model->findOrFail($id);
        return view($this->view . '.show', compact('result'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function edit($id)
    {
        $this->beforeAction(__FUNCTION__);

        $result = $this->model->findOrFail($id);
        return view($this->view . '.edit', compact('result'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        $this->beforeAction(__FUNCTION__);

        $request->validate($this->model->rules());

        $result = $this->model->findOrFail($id);
        $result->update($request->all());

        return redirect()->back();
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy($id)
    {
        $this->beforeAction(__FUNCTION__);

        $result = $this->model->findOrFail($id);
        $result->delete();

        return redirect()->back();
    }

    protected function beforeAction($action)
    {
        // 
    }
}

```

## Modo de uso

Na classe desejada, configure conforme o exemplo:

```
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Helpers\SimpleCrud;
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class UsersController extends Controller
{
    use SimpleCrud;

    protected $model;

    /**
     * Define o caminho do diretório de views (em resources/views/{$view})
     */
    protected $view = 'users';
    
    /**
     * Define o inicio do nome da rota (ver a seguir)
     */
    protected $route = 'users';

    /**
     * Injeta a model automaticamente (recurso padrão do Laravel)
     */
    public function __construct(User $model)
    {
        $this->model = $model;
    }

    /**
     * Executa uma ação qualquer ANTES de rodar o crud
     * 
     * Recebe o nome da action atual
     */
    protected function beforeAction($action)
    {
        if ($action !== 'index') {
            $this->authorize('isAdmin', Auth::user());
        }
    }
}

```

Na sequência crie os arquivos de view com a seguinte estrutura:

- resources
   - views
       - { valor da variável $view }
           - index.blade.php
           - create.blade.php
           - edit.blade.php
           - show.blade.php

As rotas precisam ser nomeadas:

```
Route::get('/users', 'UsersController@index')->name('users.index');
Route::get('/users', 'UsersController@create')->name('users.create');
Route::post('/users', 'UsersController@store')->name('users.store');
Route::get('/users/{id}', 'UsersController@show')->name('users.show');
Route::get('/users/{id}/edit', 'UsersController@edit')->name('users.edit');
Route::post('/users/{id}/edit', 'UsersController@update')->name('users.update');
Route::post('/users/{id}/destroy', 'UsersController@destroy')->name('users.destroy');
```

O segredo aqui é no parâmetro `name()`, que segue o padrão `{ valor da variavel $route }.{nome da action}`:

```
... ->name('users.index');
```

Simples e fácil de manter.
