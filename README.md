# Clean Architecture Helper Graphql Extension

<!--- Alguns exemplos. Veja https://shields.io para outros escudos customizavéis. Convém incluir dependências, status do projeto e informações da licença aqui --->
![GitHub repo size](https://img.shields.io/github/repo-size/herlanassis/django-clean-architecture-helper-gql-extension)
![GitHub contributors](https://img.shields.io/github/contributors/herlanassis/django-clean-architecture-helper-gql-extension)
![GitHub stars](https://img.shields.io/github/stars/herlanassis/django-clean-architecture-helper-gql-extension?style=social)
![GitHub forks](https://img.shields.io/github/forks/herlanassis/django-clean-architecture-helper-gql-extension?style=social)
![GitHub issues](https://img.shields.io/github/issues-raw/herlanassis/django-clean-architecture-helper-gql-extension?style=social)
![Twitter Follow](https://img.shields.io/twitter/follow/herlanassis?style=social)

Clean Architecture Helper Graphql Extension é uma `extensão` da aplicação [https://github.com/HerlanAssis/django-clean-architecture-helper](https://github.com/HerlanAssis/django-clean-architecture-helper). Ela permite que os `desenvolvedores Django` criem `aplicações escaláveis`.

Motivação para a construção do aplicativo:
* [Graphql on Django at JOOR](https://medium.com/joor-engineering/graphql-on-django-at-joor-f31dc3251482)


## Pré-requisitos

Antes de começar, verifique se você atendeu aos seguintes requisitos:
<!--- Estes são apenas exemplos de requisitos. Adicione, duplique ou remova conforme necessário --->
* Você instalou o `python 3`?;
* Você instalou uma compatível do `virtualenv`?;
* Você instalou a compatível do `virtualenvwrapper`?;
* Você instalou o `Django 3.x`?;

## Instalando Clean Architecture Helper Graphql Extension

Para instalar o Clean Architecture Helper Graphql Extension, siga estes passos:

1. Adicione a dependência no seu arquivo requirements.txt:
```text
git+git://github.com/herlanassis/django-clean-architecture-helper-gql-extension.git@master#egg=v0.1
```
1. Instale as dependências:
```shell script
pip install -r requirements.txt
```

## Utilizando Clean Architecture Helper Graphql Extension

*ATENÇÃO*

Para usar o Clean Architecture Helper Graphql Extension, siga estes passos:

```python
INSTALLED_APPS = [
   ...
   django_clean_architecture_helper,
   django_clean_architecture_helper_gql_extension,
]
```

Agora, vamos implementar as seguintes operações, GET, CREATE, UPDATE, FILTER e DELETE.

Para continuar é necessário realizar a configuração de um `endpoint graphql`, você pode fazer isso através deste tutorial [Graphene Introdution](https://docs.graphene-python.org/en/latest/quickstart/#introduction).

Neste ponto suponho que você tenha o `graphql` configurado.

Obs: todo o conteúdo referente ao graphql será criado em um módulo chamado `graphql`, sendo assim crie uma pasta chamada `graph` e dentro dela crie um arquivo `__ini__.py`.

#### GET
A primeira operação é uma ação simples para recuperar um `post`, crie um arquivo chamado `types.py` e define um objeto `post`.

```python
import graphene
from django_clean_architecture_helper_gql_extension.types import BaseType
from ..factories import PostPresentationFactory
from .filters import PostFilter


class PostType(BaseType):
    title = graphene.String()
    content = graphene.String()

    class Meta:
        view_factory = PostPresentationFactory
        interfaces = (graphene.relay.Node, )
        filter_class = PostFilter

```
Como pode ser observado acima, o `PostType` recebe em sua `classe Meta` os atributos `view_factory` e `filter_class`.
O `view_factory`é a camada que conecta uma `presentation` com a uma `view`, neste caso com o `graphql`.
O `filter_class` é responsável por gerar atributos que serão
utilizados na query `filter` como parametro de filtro no modelo do banco. A seguir um exemplo de um `filter_class`:

```python
import graphene
from django_clean_architecture_helper_gql_extension.filters import BaseFilter


class PostFilter(BaseFilter):
    def __init__(self):
        super().__init__()
        self.title__iexact = graphene.String()
        self.title__icontains = graphene.String()

```
Para criar o filtro basta criar um atributo de classe com o nome do atributo de modelo a ser filtrado com o sufixo desejado, alguns do sufixos que podem ser utilizados são:
* <field_name>__iexact
* <field_name>__exact
* <field_name>__icontains
* <field_name>__contains

Para saber mais acesse:
* [Django docs: Queries](https://docs.djangoproject.com/en/2.2/topics/db/queries/)
* [Django docs: Querysets](https://docs.djangoproject.com/en/2.2/ref/models/querysets/)

#### FILTER
Antes de implementar o `resolve_filter` é necessário criar uma `connection` com o `PostType`, exemplo:
```python
from from django_clean_architecture_helper_gql_extension.connections import TotalItemsConnection
from .types import PostType


class PostConnection(TotalItemsConnection):
    class Meta:
        node = PostType

```
Depois disso basta adicionar o seguinte código no arquivo `query.py`:
```python
import graphene
from django_clean_architecture_helper_gql_extension.connections import BaseConnectionField
from ..factories import PostPresentationFactory
from .types import PostType
from .connections import PostConnection


class Query(graphene.ObjectType):
    get_post = graphene.relay.Node.Field(PostType)
    filter_posts = BaseConnectionField(PostConnection)

    def resolve_filter_posts(self, info, **kwargs):
        body, status, errors = PostPresentationFactory.create().filter(**kwargs)
        return body
```
Tudo certo! Agora você tem os métods de get, listar e filtrar, _simple and easy_!

#### CREATE & UPDATE
Insira o código abaixo no arquivo de `mutation.py` e você vai ter os métodos de criar e atualizar.

```python
import graphene
from django_clean_architecture_helper.mutations import CreateOrUpdateMutation, DeleteMutation
from ..factories import PostPresentationFactory
from .types import PostType


class PostMutation(CreateOrUpdateMutation):
    class Meta:
        lookup_field = 'id'
        view_factory = PostPresentationFactory
        operations = ['create', 'update']
        response_name = 'post'

    class Input:
        # The input arguments for this mutation
        id = graphene.ID()
        title = graphene.String(required=True)
        content = graphene.String(required=False)

    # The class attributes define the response of the mutation
    post = graphene.Field(PostType)
```

#### DELETE
Ainda no arquivo `mutation.py` adicione o código:
```python
...

class DeletePostMutation(DeleteMutation):
    class Meta:
        lookup_field = 'id'
        view_factory = PostPresentationFactory

    class Input:
        # The input arguments for this mutation
        id = graphene.ID()
```
Depois disso basta adicionar o seguinte código no arquivo `mutation.py`:

```python
...

class Mutation(graphene.ObjectType):
    create_post = PostMutation.Field()
    update_post = PostMutation.Field()
    delete_post = DeletePostMutation.Field()
```
Pronto! Tudo feito por aqui, agora você deve ser capaz de criar, editar e deletar um `Post`.

## TODO

As próximas ações para o projeto são:

* [x] ~~Escrever README~~
* [ ] Escrever Documentação
* [ ] Escrever Testes

## Contribuindo para Clean Architecture Helper Graphql Extension

Para contribuir com Clean Architecture Helper Graphql Extension, siga estes passos:

1. Fork esse repositório.
2. Crie uma branch: `git checkout -b <branch_name>`.
3. Faça suas mudanças e comite para: `git commit -m '<commit_message>'`
4. Push para a branch de origem: `git push origin Clean Architecture Helper Graphql Extension/<location>`
5. crie um pull request.

Como alternativa, consulte a documentação do GitHub em [criando uma pull request](https://help.github.com/pt/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).

## Contribuidores

Agradeço às seguintes pessoas que contribuíram para este projeto:

* [@herlanassis](https://github.com/herlanassis)

## Contato

Se você quiser entrar em contato comigo, entre em contato com herlanassis@gmail.com.

## License
<!--- Se você não tiver certeza de qual licença aberta usar, consulte https://choosealicense.com --->
Este projeto usa a seguinte licença: [Apache 2.0](https://spdx.org/licenses/Apache-2.0.html).
