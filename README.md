# Workshop - PHP Laravel - Intro
Para visualizar o projeto navegue pelas branchs que representam cada etapa do desenvolvimento

# Requisitos do projeto
- PHP 8 ou superior

## Etapas

- [Etapa 1 - Configuração do projeto](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa1-configuracao)
- [Etapa 2 - Preparando o BD](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa2-preparando-o-bd)
- [Etapa 3 - Criando as rotas](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa3-criando-rotas)
- [Etapa 4 - Deploy](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa4-deploy)

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


### Etapa 4 - Deploy com fly.io

- Vamos criar uma conta no fly.io (https://fly.io)

- Vamos instalar o flyctl (https://fly.io/docs/getting-started/installing-flyctl/)
```bash
curl -L https://fly.io/install.sh | sh
```

- Vamos fazer login no fly.io
```bash
fly auth login
```

- Vamos configurar nosso app usando o comando launch (quando perguntar sobre o deploy escolha Não/N)
```bash
fly launch
```

- Antes do deploy precisamos ajustar o APP_URL no arquivo fly.toml para o endereço do nosso app
```toml
[env]
  APP_URL = "LINK DO APP"
```

- Para o banco de dados vamos criar uma instancia free no Render (https://render.com/)

- Vamos configurar no Secrets a conexão com o banco de dados
```bash
fly secrets set DB_CONNECTION=pgsql --stage
fly secrets set DB_HOST=LINK_DO_BD_RENDER --stage
fly secrets set DB_PORT=5432 --stage
fly secrets set DB_DATABASE=workshopphplaravel --stage
fly secrets set DB_USERNAME=workshopphplaravel --stage
fly secrets set DB_PASSWORD=SENHA_BD_RENDER --stage
fly secrets deploy
```

- Vamos adicionar também no fly.toml as configurações para executar as migrations
```toml
[deploy]
  release_command = "php /var/www/html/artisan migrate --force"
```

- Com os ajustes feitos, vamos fazer o deploy do app
```bash
fly deploy
```

- Caso o migration não seja executado podemos forçar com o comando
```bash
fly ssh console -C "php /var/www/html/artisan migrate --force"
```
