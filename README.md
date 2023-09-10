# Workshop - PHP Laravel - Intro
Para visualizar o projeto navegue pelas branchs que representam cada etapa do desenvolvimento

# Requisitos do projeto
- PHP 8 ou superior

## Etapas

- [Etapa 1 - Configuração do projeto](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa1-configuracao)
- [Etapa 2 - Preparando o BD](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa2-preparando-o-bd)

## Passo a passo

### Etapa 1 - Configuração do projeto

- Criar o projeto
```bash
composer create-project laravel/laravel workshop-php-laravel-intro
```

- Para facilitar vamos utilizar o Sail como ambiente de desenvolvimento (ele vai criar um container docker com tudo que precisamos para rodar o projeto)
```bash
cd workshop-php-laravel-intro
composer require laravel/sail --dev
php artisan sail:install
```

- Para subir o container
```bash
./vendor/bin/sail up
```

- Para acessar no navegador vamos usar - http://localhost

- Em alguns casos precisamos dar permissão para os arquivos da pasta storage
```bash
sudo chmod 777 -R storage
```

Em caso de dúvidas o link da documentação oficial: [https://laravel.com/docs/10.x/installation](https://laravel.com/docs/10.x/installation)

### Etapa 2 - Preparando o BD

- Vamos criar uma entidade Task com a migration para representar as tarefas
```bash
php artisan make:model Task -m
```

- Vamos ajustar a migration (database/migrations/create_tasks_table.php) para criar a tabela tasks
```php
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title'); // Título da tarefa
            $table->boolean('completed')->default(false); // Se a tarefa foi concluída
            $table->timestamps(); // Data de criação e atualização
        });
    }
```

- Vamos rodar a migration no Docker para criar a tabela no banco de dados
```bash
./vendor/bin/sail artisan migrate
```

- O Laravel já cria também os recursos para utilização de usuários, por enquanto não vamos utilizar, mas já está criado

- Precisamos criar também um factory para gerar os dados de teste das tarefas
```bash
php artisan make:factory TaskFactory
```

- Vamos ajustar o seeder (database/seeders/DatabaseSeeder.php) para popular a tabela com dados de teste
```php
    public function run(): void
    {
        \App\Models\Task::factory(10)->create();
    }
```

- Vamos rodar o seeder para popular a tabela com dados de teste
```bash
./vendor/bin/sail artisan db:seed
```

### Etapa 3 - Criando as rotas

- Vamos criar um resource controller para gerenciar as tarefas (o resource controller já cria as rotas para os métodos padrões)
```bash
php artisan make:controller -r TaskController
```

- Configuramos o controler para utilizar o model Task
```php
    use App\Models\Task;
```

- Vamos remover as funções create e edit do controller (app/Http/Controllers/TaskController.php) pois não vamos utilizar

- Em seguida devemos ajustar o controller (app/Http/Controllers/TaskController.php) para utilizar o model Task e retornar todas as tarefas
```php
    public function index()
    {
        return Task::all();
    }
```

- Ajustamos para armazenar uma nova tarefa
```php
    public function store(Request $request)
    {
        $task = Task::create($request->all());
        return response()->json($task, 201);
    }
```

- Vamos ajustar também para exibir uma tarefa específica
```php
    public function show(string $id)
    {
        $task = Task::find($id);
        if ($task) {
            return $task;
        }

        return response()->json(['message' => 'Tarefa não encontrada',], 404);
    }
```

- Ajustamos para atualizar uma tarefa específica
```php
    public function update(Request $request, string $id)
    {
        $task = Task::find($id);
        if ($task) {
            $task->fill($request->all());
            $task->save();
            return $task;
        }

        return response()->json(['message' => 'Tarefa não encontrada',], 404);
    }
```

- E finalmente, para excluir uma tarefa específica
```php
    public function destroy(string $id)
    {
        $task = Task::find($id);
        if ($task) {
            $task->delete();
            return response()->json(['message' => 'Tarefa removida com sucesso',], 200);
        }

        return response()->json(['message' => 'Tarefa não encontrada',], 404);
    }
```

- Com o controller pronto, vamos ajustar as rotas (routes/api.php) para utilizar o controller
```php
use App\Http\Controllers\TaskController;

Route::apiResource('tasks', TaskController::class);
```

- Para conseguir atribuir os valores para os campos da tarefa, precisamos ajustar o model (app/Models/Task.php) para permitir a atribuição em massa
```php
    protected $fillable = [
        'title',
        'completed',
    ];
```

- Você pode visualizar as rotas criadas com o comando
```bash
./vendor/bin/sail artisan route:list
```

- Na raiz do repositório você pode encontrar um arquivo .json para importar as rotas no Postman
