# Workshop - PHP Laravel - Intro
Para visualizar o projeto navegue pelas branchs que representam cada etapa do desenvolvimento

# Requisitos do projeto
- PHP 8 ou superior

## Etapas

- [Etapa 1 - Configuração do projeto](https://github.com/felipez3r0/workshop-php-laravel-intro/tree/etapa1-configuracao)

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