# API Django - Cadastro de Produtos

API REST desenvolvida com Django e Django REST Framework para gerenciamento de produtos.

## 📚 Índice

- [Estrutura do Projeto](#estrutura-do-projeto)
- [Módulos do Django](#módulos-do-django)
- [Instalação](#instalação)
- [Uso da API](#uso-da-api)
- [Endpoints](#endpoints)

---

## 🏗️ Estrutura do Projeto

```
api-python/
├── api/                    # Configurações do projeto Django
│   ├── __init__.py
│   ├── settings.py        # Configurações principais
│   ├── urls.py            # URLs principais
│   └── wsgi.py            # Configuração WSGI
├── core/                  # App principal da aplicação
│   ├── __init__.py
│   ├── models.py          # Modelos de dados
│   ├── views.py           # Views/ViewSets
│   ├── serializers.py     # Serializers DRF
│   ├── urls.py            # URLs da app
│   ├── admin.py           # Configuração do admin
│   └── migrations/        # Migrações do banco
├── manage.py              # Script de gerenciamento Django
├── requirements.txt       # Dependências Python
├── Dockerfile             # Configuração Docker
└── docker-compose.yml     # Orquestração de containers
```

---

## 🔧 Módulos do Django

### 1. Models (Modelos) - `django.db.models`

**O que é:** Define a estrutura de dados (tabelas do banco de dados).

**Exemplo no projeto:**

```python
from django.db import models

class Produto(models.Model):
    nome = models.CharField(max_length=100)
    descricao = models.TextField()
    preco = models.DecimalField(max_digits=10, decimal_places=2)
    estoque = models.IntegerField(default=0)
    criado_em = models.DateTimeField(auto_now_add=True)
    atualizado_em = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-criado_em']

    def __str__(self):
        return self.nome
```

**Campos mais comuns:**
- `CharField` - Texto curto (até 255 caracteres)
- `TextField` - Texto longo (ilimitado)
- `IntegerField` - Números inteiros
- `DecimalField` - Números decimais (preços, valores monetários)
- `DateTimeField` - Data e hora
- `BooleanField` - Verdadeiro/Falso
- `ForeignKey` - Relacionamento com outra tabela
- `ManyToManyField` - Relacionamento muitos-para-muitos

---

### 2. Views (Visualizações) - `django.views` / `rest_framework.viewsets`

**O que é:** Contém a lógica que processa requisições HTTP e retorna respostas.

**Exemplo no projeto:**

```python
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework import status
from .models import Produto
from .serializers import ProdutoSerializer

class ProdutoViewSet(viewsets.ModelViewSet):
    queryset = Produto.objects.all()
    serializer_class = ProdutoSerializer

    def list(self, request):
        """Lista todos os produtos"""
        queryset = self.get_queryset()
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

    def create(self, request):
        """Cria um novo produto"""
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

**Tipos de Views:**
- `View` - Classe base para views
- `ListView` - Lista de objetos
- `DetailView` - Detalhes de um objeto
- `CreateView` - Criar novo objeto
- `UpdateView` - Atualizar objeto
- `DeleteView` - Deletar objeto
- `ViewSet` (DRF) - Conjunto de ações CRUD

---

### 3. URLs (Roteamento) - `django.urls`

**O que é:** Mapeia URLs para views específicas.

**Exemplo no projeto:**

```python
# api/urls.py (URLs principais)
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('core.urls')),
]

# core/urls.py (URLs da app)
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProdutoViewSet

router = DefaultRouter()
router.register(r'produtos', ProdutoViewSet, basename='produto')

urlpatterns = [
    path('', include(router.urls)),
]
```

**Funções principais:**
- `path()` - Define uma rota simples
- `include()` - Inclui outras configurações de URL
- `re_path()` - Rotas com expressões regulares

---

### 4. Settings (Configurações) - `django.conf.settings`

**O que é:** Arquivo central com todas as configurações do projeto.

**Principais configurações:**

```python
# Apps instaladas
INSTALLED_APPS = [
    'django.contrib.admin',      # Painel administrativo
    'django.contrib.auth',       # Sistema de autenticação
    'django.contrib.contenttypes',
    'django.contrib.sessions',   # Gerenciamento de sessões
    'django.contrib.messages',
    'django.contrib.staticfiles', # Arquivos estáticos
    'rest_framework',            # Django REST Framework
    'core',                      # Nossa app
]

# Middleware (processadores de requisição)
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

# Banco de dados
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'teste',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': '5432',
    }
}
```

---

### 5. Admin - `django.contrib.admin`

**O que é:** Interface administrativa automática para gerenciar dados.

**Como usar:**
1. Criar superusuário: `python manage.py createsuperuser`
2. Acessar: `http://localhost:8000/admin/`
3. Registrar modelos no `admin.py`:

```python
from django.contrib import admin
from .models import Produto

@admin.register(Produto)
class ProdutoAdmin(admin.ModelAdmin):
    list_display = ['id', 'nome', 'preco', 'estoque', 'criado_em']
    list_filter = ['criado_em', 'atualizado_em']
    search_fields = ['nome', 'descricao']
    readonly_fields = ['criado_em', 'atualizado_em']
```

---

### 6. Migrations (Migrações) - `django.db.migrations`

**O que é:** Sistema de versionamento de mudanças no banco de dados.

**Comandos principais:**
```bash
# Criar arquivos de migração
python manage.py makemigrations

# Aplicar migrações no banco
python manage.py migrate

# Ver status das migrações
python manage.py showmigrations
```

---

### 7. Serializers (Django REST Framework)

**O que é:** Converte objetos Python em JSON e vice-versa.

**Exemplo:**

```python
from rest_framework import serializers
from .models import Produto

class ProdutoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Produto
        fields = ['id', 'nome', 'descricao', 'preco', 'estoque', 
                  'criado_em', 'atualizado_em']
        read_only_fields = ['id', 'criado_em', 'atualizado_em']
```

---

### 8. Middleware

**O que é:** Processa requisições e respostas antes/depois das views.

**Exemplos:**
- `SecurityMiddleware` - Headers de segurança
- `SessionMiddleware` - Gerenciamento de sessões
- `CsrfViewMiddleware` - Proteção CSRF
- `AuthenticationMiddleware` - Autenticação de usuários

---

## 🚀 Instalação

### Pré-requisitos
- Docker e Docker Compose instalados
- Python 3.11+ (se rodar localmente)

### Com Docker (Recomendado)

1. **Clone o repositório ou navegue até a pasta do projeto**

2. **Suba os containers:**
```bash
docker-compose up -d
```

3. **Execute as migrações:**
```bash
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
```

4. **Crie um superusuário (opcional):**
```bash
docker-compose exec web python manage.py createsuperuser
```

5. **Acesse a API:**
- API: `http://localhost:8000/api/produtos/`
- Admin: `http://localhost:8000/admin/`
- Adminer (PostgreSQL): `http://localhost:8085/`

### Sem Docker

1. **Instale as dependências:**
```bash
pip install -r requirements.txt
```

2. **Configure o banco de dados no `settings.py`**

3. **Execute as migrações:**
```bash
python manage.py makemigrations
python manage.py migrate
```

4. **Inicie o servidor:**
```bash
python manage.py runserver
```

---

## 📡 Uso da API

### Criar um Produto

```bash
POST http://localhost:8000/api/produtos/
Content-Type: application/json

{
    "nome": "Notebook Dell",
    "descricao": "Notebook Dell Inspiron 15 com 8GB RAM",
    "preco": "3299.99",
    "estoque": 10
}
```

### Listar Produtos

```bash
GET http://localhost:8000/api/produtos/
```

### Obter Detalhes de um Produto

```bash
GET http://localhost:8000/api/produtos/1/
```

### Atualizar Produto

```bash
PUT http://localhost:8000/api/produtos/1/
Content-Type: application/json

{
    "nome": "Notebook Dell Atualizado",
    "descricao": "Nova descrição",
    "preco": "2999.99",
    "estoque": 15
}
```

### Deletar Produto

```bash
DELETE http://localhost:8000/api/produtos/1/
```

---

## 🔗 Endpoints

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/produtos/` | Lista todos os produtos |
| POST | `/api/produtos/` | Cria um novo produto |
| GET | `/api/produtos/{id}/` | Detalhes de um produto |
| PUT | `/api/produtos/{id}/` | Atualiza um produto completo |
| PATCH | `/api/produtos/{id}/` | Atualização parcial |
| DELETE | `/api/produtos/{id}/` | Deleta um produto |

---

## 🔄 Fluxo de uma Requisição no Django

```
1. URL (urls.py) 
   ↓
2. Middleware (processa requisição)
   ↓
3. View (views.py) - lógica de negócio
   ↓
4. Model (models.py) - busca no banco
   ↓
5. Serializer - converte para JSON
   ↓
6. Response - retorna JSON
   ↓
7. Middleware (processa resposta)
   ↓
8. Cliente recebe resposta
```

---

## 📦 Apps Instaladas

1. **django.contrib.admin** - Painel administrativo
2. **django.contrib.auth** - Sistema de autenticação de usuários
3. **django.contrib.contenttypes** - Framework de tipos de conteúdo
4. **django.contrib.sessions** - Gerenciamento de sessões
5. **django.contrib.messages** - Sistema de mensagens flash
6. **django.contrib.staticfiles** - Gerenciamento de arquivos estáticos
7. **rest_framework** - Django REST Framework
8. **core** - App customizada do projeto

---

## 🛠️ Tecnologias Utilizadas

- **Django 4.2.7** - Framework web Python
- **Django REST Framework 3.14.0** - Framework para APIs REST
- **PostgreSQL 15** - Banco de dados relacional
- **Docker** - Containerização
- **Gunicorn** - Servidor WSGI para produção

---

## 📝 Licença

Este projeto é para fins educacionais.

---

## 👨‍💻 Autor

Desenvolvido para aprendizado de Django e Django REST Framework.

