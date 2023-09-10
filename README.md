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
