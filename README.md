# Projeto Base Django com Autenticação (django-allauth)

Este é um projeto base robusto para aplicações Django, focado em um sistema de autenticação completo e seguro utilizando a biblioteca `django-allauth`. O guia abaixo detalha todos os passos para construir este projeto do zero.

## Funcionalidades

* Estrutura de projeto Django 5.x.
* Sistema de autenticação completo (Cadastro, Login, Logout).
* Verificação de e-mail obrigatória para novos usuários.
* Login flexível com `username` ou `e-mail`.
* Recuperação de senha por e-mail.
* Gerenciamento seguro de credenciais com variáveis de ambiente (`.env`).
* Templates customizados e estilizados com CSS.

## Passo a Passo da Construção

### 1. Preparação do Ambiente e Criação do Projeto

Primeiro, configuramos o ambiente de desenvolvimento.

1.  **Crie e acesse a pasta do projeto:**
    ```bash
    mkdir nome_do_projeto
    cd nome_do_projeto
    ```

2.  **Crie e ative um ambiente virtual:**
    * **Windows:**
        ```bash
        python -m venv venv
        .\venv\Scripts\activate
        ```
    * **Linux/Mac:**
        ```bash
        python3 -m venv venv
        source venv/bin/activate
        ```

3.  **Instale o Django:**
    ```bash
    pip install Django
    ```

4.  **Crie o projeto Django:**
    ```bash
    django-admin startproject config .
    ```

### 2. Gerenciamento Seguro de Credenciais

Para evitar expor dados sensíveis, usamos a biblioteca `django-environ`.

1.  **Instale a biblioteca:**
    ```bash
    pip install django-environ
    ```

2.  **Crie o arquivo `.env`** na raiz do projeto para guardar as variáveis:
    ```ini
    # .env
    SECRET_KEY='sua_secret_key_aqui'
    DEBUG=True
    EMAIL_HOST_USER='seu_email@gmail.com'
    EMAIL_HOST_PASSWORD='sua_senha_de_app_do_google'
    ```

3.  **Crie o arquivo `.gitignore`** para garantir que o `.env` não seja enviado para o repositório:
    ```
    # .gitignore
    .env
    venv/
    __pycache__/
    db.sqlite3
    staticfiles/
    ```

### 3. Instalação e Configuração do `django-allauth`

O coração do nosso sistema de autenticação.

1.  **Instale a biblioteca:**
    ```bash
    pip install django-allauth
    ```

2.  **Configure o `settings.py`** (`config/settings.py`):
    * Adicione os apps necessários ao `INSTALLED_APPS`:
        ```python
        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',

            # Apps de terceiros
            'allauth',
            'allauth.account',
            'allauth.socialaccount',
        ]
        ```
    * Adicione as configurações do `allauth`, e-mail e arquivos estáticos no final do arquivo:
        ```python
        # config/settings.py

        # ARQUIVOS ESTÁTICOS (CSS, JS, IMAGENS)
        STATIC_URL = 'static/'
        STATICFILES_DIRS = [ BASE_DIR / 'static' ]
        STATIC_ROOT = BASE_DIR / 'staticfiles'

        # CONFIGURAÇÕES DO ALLAUTH
        AUTHENTICATION_BACKENDS = [
            'django.contrib.auth.backends.ModelBackend',
            'allauth.account.auth_backends.AuthenticationBackend',
        ]
        SITE_ID = 1
        LOGIN_REDIRECT_URL = '/'
        LOGOUT_REDIRECT_URL = '/'

        ACCOUNT_LOGIN_METHODS = ["username", "email"]
        ACCOUNT_EMAIL_VERIFICATION = "mandatory"
        ACCOUNT_SIGNUP_FIELDS = ["username", "email"]
        ACCOUNT_UNIQUE_EMAIL = True

        # CONFIGURAÇÃO DE E-MAIL
        EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
        EMAIL_HOST = 'smtp.gmail.com'
        EMAIL_PORT = 587
        EMAIL_USE_TLS = True
        EMAIL_HOST_USER = env('EMAIL_HOST_USER') 
        EMAIL_HOST_PASSWORD = env('EMAIL_HOST_PASSWORD')
        DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
        ```

### 4. Configuração das Rotas (URLs)

Conectamos as views do `allauth` e da nossa homepage ao roteador principal.

1.  **Crie o arquivo de views** em `config/views.py`:
    ```python
    # config/views.py
    from django.shortcuts import render

    def homepage(request):
        return render(request, 'homepage.html')
    ```

2.  **Atualize o arquivo de rotas principal** `config/urls.py`:
    ```python
    # config/urls.py
    from django.contrib import admin
    from django.urls import path, include
    from django.conf import settings
    from django.conf.urls.static import static
    from . import views

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('accounts/', include('allauth.urls')),
        path('', views.homepage, name='homepage'),
    ]

    if settings.DEBUG:
        urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
    ```

### 5. Criação do Banco de Dados

Aplicamos as migrações para criar as tabelas necessárias.

```bash
python manage.py migrate
```

### 6. Estrutura de Templates e Estilo (Frontend)

Criamos a estrutura visual do site.

1.  **Crie as pastas** `templates/account` e `static/css` na raiz do projeto.

2.  **Crie o template base** em `templates/base.html`:
    ```html
    {% load static %}
    <!DOCTYPE html>
    <html>
    <head>
        <title>Meu Projeto</title>
        <link rel="stylesheet" href="{% static 'css/style.css' %}">
    </head>
    <body>
        <main>
            {% block content %}{% endblock %}
        </main>
    </body>
    </html>
    ```

3.  **Crie os templates customizados** do `allauth` dentro de `templates/account/`. Exemplo para `login.html`:
    ```html
    {% extends "base.html" %}
    {% block content %}
    <div class="auth-container">
        <h2>Entrar</h2>
        <form class="login" method="POST" action="{% url 'account_login' %}">
            {% csrf_token %}
            {{ form.as_p }}
            <div class="links">
                <a href="{% url 'account_reset_password' %}">Esqueceu a senha?</a>
            </div>
            <button type="submit">Entrar</button>
        </form>
    </div>
    {% endblock %}
    ```

4.  **Adicione o CSS** em `static/css/style.css`.
    *(Use o CSS aprimorado que forneci anteriormente).*

### 7. Finalização e Teste

1.  **Colete os arquivos estáticos** (importante para o `urls.py` funcionar corretamente):
    ```bash
    python manage.py collectstatic
    ```

2.  **Inicie o servidor:**
    ```bash
    python manage.py runserver
    ```

3.  **Acesse as páginas no navegador:**
    * Homepage: `http://127.0.0.1:8000/`
    * Login: `http://127.0.0.1:8000/accounts/login/`
    * Cadastro: `http://127.0.0.1:8000/accounts/signup/`

Seu projeto base agora está completo e funcional!
