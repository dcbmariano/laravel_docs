
# Visualizações

- [Introdução](#introdução)
- [Criando e renderizando visualizações](#creating-and-rendering-views)
    - [diretórios de exibição aninhada](#nested-view-directories)
    - [Criando a primeira visualização disponível](#creating-the-first-available-view)
    - [Determinando se uma visualização existe](#determining-if-a-view-exists)
- [Passando dados para visualizações](#passing-data-to-views)
    - [Compartilhando dados com todas as visualizações](#sharing-data-with-all-views)
- [Ver compositores](#view-compositores)
    - [Ver criadores](#view-creators)
- [Otimizando visualizações](#optimizing-views)


## Introdução

Obviamente, não é prático retornar strings inteiras de documentos HTML diretamente de suas rotas e controladores. Felizmente, as visualizações fornecem uma maneira conveniente de colocar todo o nosso HTML em arquivos separados. As visualizações separam a lógica do seu controlador/aplicativo da sua lógica de apresentação e são armazenadas no diretório `resources/views`. Uma visualização simples pode ser algo assim:

``` blade

   


    
        
Olá, {{ $name }}

    

```

Como esta visão é armazenada em `resources/views/greeting.blade.php`, podemos retorná-la usando o auxiliar global `view` assim:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> {dica} Procurando mais informações sobre como escrever templates Blade? Confira a [documentação do Blade](/docs/{{version}}/blade) completa para começar.


## Criando e renderizando visualizações

Você pode criar uma visão colocando um arquivo com a extensão `.blade.php` no diretório `resources/views` da sua aplicação. A extensão `.blade.php` informa ao framework que o arquivo contém um [modelo Blade](/docs/{{versão}}/blade). Os modelos blade contêm HTML e diretivas Blade que permitem ecoar valores facilmente, criar instruções "if", iterar sobre dados e muito mais.

Depois de criar uma visão, você pode retorná-la de uma das rotas ou controladores de sua aplicação usando o auxiliar global `view`:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

As visualizações também podem ser retornadas usando a fachada `View`:

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

Como você pode ver, o primeiro argumento passado para o auxiliar `view` corresponde ao nome do arquivo de visualização no diretório `resources/views`. O segundo argumento é uma matriz de dados que deve ser disponibilizada para a exibição. Neste caso, estamos passando a variável `name`, que é exibida na visualização usando [sintaxe Blade](/docs/{{versão}}/blade).


### Diretórios de visualização aninhada

As visualizações também podem ser aninhadas em subdiretórios do diretório `resources/views`. A notação "ponto" pode ser usada para fazer referência a visualizações aninhadas. Por exemplo, se sua visualização estiver armazenada em `resources/views/admin/profile.blade.php`, você pode retorná-la de uma das rotas/controladores do seu aplicativo da seguinte forma:

    return view('admin.profile', $data);

> {note} Os nomes dos diretórios de visualização não devem conter o caractere `.`.


### Criando a primeira visualização disponível

Usando o método `first` da fachada `View`, você pode criar a primeira visão que existe em um determinado array de views. Isso pode ser útil se seu aplicativo ou pacote permitir que as visualizações sejam personalizadas ou substituídas:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);


### Determinando se existe uma visualização

Se você precisar determinar se uma visão existe, você pode usar a fachada `View`. O método `exists` retornará `true` se a visualização existir:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }


## Passando dados para visualizações

Como você viu nos exemplos anteriores, você pode passar uma matriz de dados para visualizações para disponibilizar esses dados para a visualização:

    return view('greetings', ['name' => 'Victoria']);

Ao passar informações dessa maneira, os dados devem ser um array com pares chave/valor. Depois de fornecer dados para uma visualização, você pode acessar cada valor em sua visualização usando as chaves dos dados, como `
   `.

Como alternativa para passar um array completo de dados para a função auxiliar `view`, você pode usar o método `with` para adicionar partes individuais de dados à visualização. O método `with` retorna uma instância do objeto view para que você possa continuar encadeando métodos antes de retornar a view:

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');


### Compartilhamento de dados com todas as visualizações

Ocasionalmente, você pode precisar compartilhar dados com todas as exibições renderizadas pelo seu aplicativo. Você pode fazer isso usando o método `share` da fachada `View`. Normalmente, você deve fazer chamadas para o método `share` dentro do método `boot` de um provedor de serviços. Você pode adicioná-los à classe `App\Providers\AppServiceProvider` ou gerar um provedor de serviços separado para abrigá-los:

    
   
## Ver compositores

Compositores de exibição são retornos de chamada ou métodos de classe que são chamados quando uma exibição é renderizada. Se você tiver dados que deseja vincular a uma exibição sempre que essa exibição for renderizada, um compositor de exibição poderá ajudá-lo a organizar essa lógica em um único local. Os compositores de exibição podem ser particularmente úteis se a mesma exibição for retornada por várias rotas ou controladores em seu aplicativo e sempre precisar de um dado específico.

Normalmente, os compositores de visualização serão registrados em um dos [provedores de serviços](/docs/{{version}}/providers) do seu aplicativo. Neste exemplo, vamos supor que criamos um novo `App\Providers\ViewServiceProvider` para hospedar essa lógica.

Usaremos o método `composer` da fachada `View` para registrar a view composer. O Laravel não inclui um diretório padrão para compositores de visualização baseados em classes, então você pode organizá-los da maneira que desejar. Por exemplo, você pode criar um diretório `app/View/Composers` para hospedar todos os compositores de visualização do seu aplicativo:

    
   {note} Lembre-se, se você criar um novo provedor de serviço para conter seus registros do compositor de visualização, você precisará adicionar o provedor de serviço ao array `providers` no arquivo de configuração `config/app.php`.

Agora que registramos o compositor, o método `compose` da classe `App\View\Composers\ProfileComposer` será executado toda vez que a view `profile` estiver sendo renderizada. Vamos dar uma olhada em um exemplo da classe composer:

    
   users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  \Illuminate\View\View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }


Como você pode ver, todos os compositores de visualização são resolvidos por meio do [contêiner de serviço](/docs/{{version}}/container), então você pode indicar quaisquer dependências necessárias dentro do construtor de um compositor.


#### Anexando um compositor a várias visualizações

Você pode anexar um view composer a várias views de uma vez passando um array de views como o primeiro argumento para o método `composer`:

    use App\Views\Composers\MultiComposer;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

O método `composer` também aceita o caractere `*` como curinga, permitindo que você anexe um compositor a todas as visualizações:

    View::composer('*', function ($view) {
        //
    });


### Ver criadores

Os "criadores" de visualização são muito semelhantes aos compositores de visualização; no entanto, eles são executados imediatamente após a visualização ser instanciada, em vez de esperar até que a visualização esteja prestes a ser renderizada. Para registrar um criador de visualização, use o método `creator`:

    use App\View\Creators\ProfileCreator;
    use Illuminate\Support\Facades\View;

    View::creator('profile', ProfileCreator::class);


## Otimizando visualizações

Por padrão, as visualizações do modelo Blade são compiladas sob demanda. Quando é executada uma requisição que renderiza uma visão, o Laravel determinará se existe uma versão compilada da visão. Se o arquivo existir, o Laravel determinará se a visualização não compilada foi modificada mais recentemente do que a visualização compilada. Se a visão compilada não existir, ou a visão não compilada foi modificada, o Laravel irá recompilar a visão.

Compilar visualizações durante a solicitação pode ter um pequeno impacto negativo no desempenho, então o Laravel fornece o comando `view:cache` Artisan para pré-compilar todas as visualizações utilizadas por sua aplicação. Para aumentar o desempenho, você pode executar este comando como parte de seu processo de implantação:

```shell
php artisan view:cache
```

Você pode usar o comando `view:clear` para limpar o cache de visualização:

```shell
php artisan view:clear
```
