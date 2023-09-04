# django-avancado-postgresql

>Projeto Django Avançado com Bootstrap e PostgreSQL.
> 
>>Projeto desenvolvido no curso da Geek University - Udemy [Programação Web com Python e Django Framework: Essencial](https://www.udemy.com/course/programacao-web-com-django-framework-do-basico-ao-avancado/)

## Ambiente de Desenvolvimento
Linux, Visual Studio Code, Docker e PostgreSQL

## Documentação
- [DJango](https://www.djangoproject.com/)
- Dica postgreSQL [vivaolinux](https://www.vivaolinux.com.br/artigo/psql-Conheca-o-basico)
## Desenvolvimento:
1. <span style="color:383E42"><b>Preparando ambiente</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Criar repositório no github com `gitignore` e `README.md`
    - Editar `README` e colocar estrutura básica
    - Criar diretório `readmeImages` e colocar imagens para uso no `README.md`
    - Editar `gitignore` e colocar configuração para `python, django, vscode/visualstudio code`
        >Use o site [gitignore.io](https://www.toptal.com/developers/gitignore/)
    
    - Incluir ao `gitignore` o arquivo `privateData.py`
        >São arquivos que não devem ir para o repositório github

    - Criar e ativar ambiente virtual
        ```sh
        python3 -m venv venv
        source venv/bin/activate
        ```
    - Instalação pip - se necessário
        ```sh
        sudo apt update
        sudo apt install python3-pip
        pip3 --version
        ```
    - Instalar o `django`, `psycopg2-binary` (para trabalhar com PostgreSQL), `gunicorn`( servidor para python), `django-std-image`(para trabalhar com imagens)
        ```bash
        sudo apt update
        pip3 install django
        pip3 install psycopg2-binary gunicorn django-static django-stdimage
        ```

    - Criação arquivo requirements
    Contém informaçẽos sobre todas as bibliotecas utilizadas no projeto. Para atualizar o arquivo, basta executar o comando novamente após instalar outras bibliotecas.
        ```sh
        pip freeze > requirements.txt
        ```

    </p>

    </details> 

    ---

2. <span style="color:383E42"><b>Criar container fusion-postgres usando `POSTGRESQL` do `dockerhub`</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - [Documentação dockerhub](https://hub.docker.com/_/mysql/tags)
        - Baixar imagem POSTGRESQL
            ```bash
            docker pull postgres
            ```
        - Cria container 
        Nomeando `--name fusion-postgres` 
        Adiciono informação da porta `-p 5432:5432`
        Informo a senha `POSTGRES_PASSWORD=suasenha`
        ```bash
        docker run -p 5432:5432 --name fusion-postgres -e POSTGRES_PASSWORD=suasenha -d postgres

        ```
        - Iniciar container
            ```bash
            docker start fusion-postgres
            ```
        - Verificar `id` container e `ip` do container
            ```bash
            sudo docker ps
            sudo docker container inspect idcontainer
            ```

        - Acessar container no modo interativo - container em execução
            ```bash
            sudo docker exec -it idcontainer bash
            ```
            - Acessando postgres `database` com usuário `postgres`
                ```bash
                psql -U postgres
                ```
            - Criar database
                ```bash
                create database "fusion";
                ```
            -  Criar usuário no postgres
                ```bash
                create user cristiano superuser inherit createdb createrole password 'surasenha';
                ```

            - Saindo do postgres
                ```bash
                \q
                ```
            - Acessando database `fusion`. Use o  `ip` do container
                >Comandos válidos
                ```bash
                psql -U postgres -d fusion
                psql ipcontainer -U postgres -d fusion

                psql -h ipcontainer -U postgres -d fusion
                ```
            - Listando database
                ```bash
                \l
                ```
            - Sair do container
                ```bash
                exit
                ```

    </p>

    </details> 

    ---

3. <span style="color:383E42"><b>Criação de projeto `fusion` e app `core`</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>
    
    - Criar app no mesmo diretório/pasta que está o projeto.
        >Criar arquivo `privateData.py` com dicionário de dados `myData` contendo as informaçoes que não quero que vá para repositório - Então incluirei o arquivo com a classe no gitignore
        Dicinário `myData`
        ```python
        myData = {
            'SENHA_PSTGRESQL': '',
            'USUARIO_POSTGRESQL': '',
            'SECRET_SETTINGS': '',
            'POSTGRESQL_DB_NAME': '',
            'HOST': '',
        }
        ```
        ```sh
        django-admin startproject fusion .
        django-admin startapp core
        ```
     
    - Configuração em `settings.py`
        - Habilitar acesso
            ```python
            ALLOWED_HOSTS = ['*']
            ```
        - Incluir app `core`
            ```python
            INSTALLED_APPS = [
                'django.contrib.admin',
                'django.contrib.auth',
                'django.contrib.contenttypes',
                'django.contrib.sessions',
                'django.contrib.messages',
                'django.contrib.staticfiles',

                'core',
            ]
            ```
        - Informar diretório `templates`
            ````python
            TEMPLATES = [
                {
                    'BACKEND': 'django.template.backends.django.DjangoTemplates',
                    'DIRS': ['templates'],
                    'APP_DIRS': True,
                    'OPTIONS': {
                        'context_processors': [
                            'django.template.context_processors.debug',
                            'django.template.context_processors.request',
                            'django.contrib.auth.context_processors.auth',
                            'django.contrib.messages.context_processors.messages',
                        ],
                    },
                },
            ]
            ```
        - Configurar databases para PostgreSQL
            ```python
            DATABASES = {
                'default': {
                    'ENGINE': 'django.db.backends.postgresql',
                    'NAME': privateData['POSTGRESQL_DB_NAME'],
                    'USER': privateData['USUARIO_POSTGRESQL'],
                    'PASSWORD': privateData['SENHA_POSTGRESQL'],
                    'HOST': privateData['HOST'],
                    'PORT':'5432',
                    
                }
            }
            ```
        - Definindo `timezone`
            ```python
            # Internationalization
            # https://docs.djangoproject.com/en/4.2/topics/i18n/

            LANGUAGE_CODE = 'pt-br'

            TIME_ZONE = 'America/Sao_Paulo'

            USE_I18N = True

            USE_TZ = True

            ```
        - Configuração para arquivos státicos
            ```python
            import os
            from pathlib import Path
            #...
            STATIC_URL = 'static/'
            MEDIA_URL = 'media/'
            STATIC_ROOT = os.path.join(STATIC_URL, 'staticfiles')
            MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
            #...
            ```
    - Incluir diretórios `core/templates` e `core/static`
    - Incluir rota para app `core` no arquivo `fusion/urls.py`
        >Direciona para rotas do `core/urls.py` - Obs.: Ainda será criado o arquivo de urls do app
        ```python
        from django.contrib import admin
        from django.urls import path, include

        from django.conf.urls.static import static
        from django.conf import settings

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('', include('core.urls')),
        ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
        ```

    </p>

    </details> 

    ---

## Meta
><span style="color:383E42"><b>Cristiano Mendonça Gueivara</b> </span>
>
>>[<img src="readmeImages/githubIcon.png">](https://github.com/sspectro "Meu perfil no github")
>
>><a href="https://linkedin.com/in/cristiano-m-gueivara/"><img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white"></a> 
>
>>[<img src="https://sspectro.github.io/images/cristiano.jpg" height="25" width="25"> - Minha Página Github](https://sspectro.github.io/#home "Minha Página no github")<br>



><span style="color:383E42"><b>Licença:</b> </span> Distribuído sobre a licença `Software Livre`. Veja Licença **[MIT](https://opensource.org/license/mit/)**.