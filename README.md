# django-avancado-postgresql

>Projeto Django Avançado com Bootstrap e PostgreSQL. Inclusão de testes.
> 
>>Projeto desenvolvido no curso da Geek University - Udemy [Programação Web com Python e Django Framework: Essencial](https://www.udemy.com/course/programacao-web-com-django-framework-do-basico-ao-avancado/)

## Ambiente de Desenvolvimento
Linux, Visual Studio Code, Docker e PostgreSQL

## Documentação
- [DJango](https://www.djangoproject.com/)
- Dica postgreSQL [vivaolinux](https://www.vivaolinux.com.br/artigo/psql-Conheca-o-basico)
- Tests [model_mommy](https://model-mommy.readthedocs.io/en/latest/basic_usage.html)
- Tests [coverage](https://coverage.readthedocs.io/en/7.3.2/)
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
            >Criação database e usuário
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

4. <span style="color:383E42"><b>Trabalhando com Class Based Views</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Criar arquivo `core/urls.py` no app core
        >Incluir rota para view `IndexView`
        ```python
        from django.urls import path

        from .views import IndexView

        urlpatterns = [
            path('', IndexView.as_view(), name='index'),
        ]
        ```

    - Criar view `IndexView`
        ```python
        from django.views.generic import TemplateView

        class IndexView(TemplateView):
            template_name = 'index.html'
        ```

    </p>

    </details> 

    ---

5. <span style="color:383E42"><b>Definindo os Templates</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Template `core/templates/404.html`
        ```html
        {% load static %}
        <div id="hero-area" class="hero-area-bg">
            <div class="container">      
            <div class="row">
                <div class="col-lg-7 col-md-12 col-sm-12 col-xs-12">
                <div class="contents">
                    <h2 class="head-title">App, Business & SaaS<br>Landing Page Template</h2>
                    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem repellendus quasi fuga nesciunt dolorum nulla magnam veniam sapiente, fugiat! fuga nesciunt dolorum nulla magnam veniam sapiente, fugiat!</p>
                    <div class="header-button">
                    <a href="#" class="btn btn-common">Download Now</i></a>
                    <a href="#" class="btn btn-border video-popup">Learn More</i></a>
                    </div>
                </div>
                </div>
                <div class="col-lg-5 col-md-12 col-sm-12 col-xs-12">
                <div class="intro-img">
                    <img class="img-fluid" src="{% static 'img/intro-mobile.png' %}" alt="">
                </div>            
                </div>
            </div> 
            </div> 
        </div>
        <!-- Hero Area End -->

        </header>
        <!-- Header Area wrapper End -->
        ```

    - Template `core/templates/500.html
        ```html
        {% extends 'base.html' %}
        {% load static %}
        {% block content %}
            <!-- Hero Area Start -->
            <div id="hero-area" class="hero-area-bg">
                <div class="container">
                <div class="row">
                    <div class="col-lg-7 col-md-12 col-sm-12 col-xs-12">
                    <div class="contents">
                        <h2 class="head-title">500<br>Erro de processamento</h2>
                        <p>Infelizmente não foi possível processar a requisição.</p>
                        <div class="header-button">
                        <a href="{% url 'index' %}" class="btn btn-common">Volte para a página principal</i></a>
                        </div>
                    </div>
                    </div>
                    <div class="col-lg-5 col-md-12 col-sm-12 col-xs-12">
                    <div class="intro-img">
                        <img class="img-fluid" src="{% static 'img/intro-mobile.png' %}" alt="">
                    </div>
                    </div>
                </div>
                </div>
            </div>
            <!-- Hero Area End -->
        {% endblock %}
        ```

    - Template `base.html`
        >Template com html padrão para todas as páginas. Incluindo bootstra4, js e css
        ```html
        {% load static %}
        <!DOCTYPE html>
        <html lang="pt-br">
        <head>
            <!-- Required meta tags -->
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

            <title>Fusion</title>

            <!-- Bootstrap CSS -->
            <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}" >
            <!-- Icon -->
            <link rel="stylesheet" href="{% static 'fonts/line-icons.css' %}">
            <!-- Owl carousel -->
            <link rel="stylesheet" href="{% static 'css/owl.carousel.min.css' %}">
            <link rel="stylesheet" href="{% static 'css/owl.theme.css' %}">

            <!-- Animate -->
            <link rel="stylesheet" href="{% static 'css/animate.css' %}">
            <!-- Main Style -->
            <link rel="stylesheet" href="{% static 'css/main.css' %}">
            <!-- Responsive Style -->
            <link rel="stylesheet" href="{% static 'css/responsive.css' %}">

        </head>
        <body>

            <!-- Header Area wrapper Starts -->
            <header id="header-wrap">
            <!-- Navbar Start -->
            <nav class="navbar navbar-expand-md bg-inverse fixed-top scrolling-navbar">
                <div class="container">
                <!-- Brand and toggle get grouped for better mobile display -->
                <a href="{% url 'index' %}" class="navbar-brand"><img src="{% static 'img/logo.png' %}" alt=""></a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
                    <i class="lni-menu"></i>
                </button>
                <div class="collapse navbar-collapse" id="navbarCollapse">
                    <ul class="navbar-nav mr-auto w-100 justify-content-end clearfix">
                    <li class="nav-item active">
                        <a class="nav-link" href="#hero-area">
                        Início
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#services">
                        Serviços
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#team">
                        Equipe
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#pricing">
                        Preços
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#testimonial">
                        Clientes
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="#contact">
                        Contato
                        </a>
                    </li>
                    </ul>
                </div>
                </div>
            </nav>
            <!-- Navbar End -->
            {% block content %} {% endblock %}

            <!-- Go to Top Link -->
            <a href="#" class="back-to-top">
                <i class="lni-arrow-up"></i>
            </a>

            <!-- Preloader -->
            <div id="preloader">
            <div class="loader" id="loader-1"></div>
            </div>
            <!-- End Preloader -->

            <!-- jQuery first, then Popper.js, then Bootstrap JS -->
            <script src="{% static 'js/jquery-min.js' %}"></script>
            <script src="{% static 'js/popper.min.js' %}"></script>
            <script src="{% static 'js/bootstrap.min.js' %}"></script>
            <script src="{% static 'js/owl.carousel.min.js' %}"></script>
            <script src="{% static 'js/wow.js' %}"></script>
            <script src="{% static 'js/jquery.nav.js' %}"></script>
            <script src="{% static 'js/scrolling-nav.js' %}"></script>
            <script src="{% static 'js/jquery.easing.min.js' %}"></script>
            <script src="{% static 'js/main.js' %}"></script>
            <script src="{% static 'js/form-validator.min.js' %}"></script>
            <script src="{% static 'js/contact-form-script.min.js' %}"></script>

        </body>
        </html>

        ```


    - Template `servicos.html`
        ```html
        {% load static %}
        <section id="services" class="section-padding">
            <div class="container">
            <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Our Services</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
            </div>
            <div class="row">
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.3s">
                    <div class="icon">
                    <i class="lni-cog"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Easy To Used</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.6s">
                    <div class="icon">
                    <i class="lni-stats-up"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Awesome Design</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.9s">
                    <div class="icon">
                    <i class="lni-users"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Easy To Customize</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.2s">
                    <div class="icon">
                    <i class="lni-layers"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">UI/UX Design</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.5s">
                    <div class="icon">
                    <i class="lni-mobile"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">App Development</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.8s">
                    <div class="icon">
                    <i class="lni-rocket"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">User Friendly interface</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
            </div>
            </div>
        </section>
        ```
    - Template `core/templates/chamada.html`
        ```html
        {% load static %}
        <section id="cta" class="section-padding">
            <div class="container">
                <div class="row">
                <div class="col-lg-6 col-md-6 col-xs-12 wow fadeInLeft" data-wow-delay="0.3s">
                    <div class="cta-text">
                    <h4>Get 30 days free trial</h4>
                    <p>Praesent imperdiet, tellus et euismod euismod, risus lorem euismod erat, at finibus neque odio quis metus. Donec vulputate arcu quam. </p>
                    </div>
                </div>
                <div class="col-lg-6 col-md-6 col-xs-12 text-right wow fadeInRight" data-wow-delay="0.3s">
                    </br><a href="#" class="btn btn-common">Register Now</a>
                </div>
                </div>
            </div>
        </section>
        ```
    
    - Template `core/templates/clientes.html`
        ```html
        {% load static %}
        <section id="testimonial" class="testimonial section-padding">
            <div class="container">
                <div class="row justify-content-center">
                <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12">
                    <div id="testimonials" class="owl-carousel wow fadeInUp" data-wow-delay="1.2s">
                    <div class="item">
                        <div class="testimonial-item">
                        <div class="img-thumb">
                            <img src="{% static 'img/testimonial/img1.jpg' %}" alt="">
                        </div>
                        <div class="info">
                            <h2><a href="#">David Smith</a></h2>
                            <h3><a href="#">Creative Head</a></h3>
                        </div>
                        <div class="content">
                            <p class="description">Praesent cursus nulla non arcu tempor, ut egestas elit tempus. In ac ex fermentum, gravida felis nec, tincidunt ligula.</p>
                            <div class="star-icon mt-3">
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            </div>
                        </div>
                        </div>
                    </div>
                    <div class="item">
                        <div class="testimonial-item">
                        <div class="img-thumb">
                            <img src="{% static 'img/testimonial/img2.jpg' %}" alt="">
                        </div>
                        <div class="info">
                            <h2><a href="#">Domeni GEsson</a></h2>
                            <h3><a href="#">Awesome Technology co.</a></h3>
                        </div>
                        <div class="content">
                            <p class="description">Praesent cursus nulla non arcu tempor, ut egestas elit tempus. In ac ex fermentum, gravida felis nec, tincidunt ligula.</p>
                            <div class="star-icon mt-3">
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            </div>
                        </div>
                        </div>
                    </div>
                    <div class="item">
                        <div class="testimonial-item">
                        <div class="img-thumb">
                            <img src="{% static 'img/testimonial/img3.jpg' %}" alt="">
                        </div>
                        <div class="info">
                            <h2><a href="#">Dommini Albert</a></h2>
                            <h3><a href="#">Nesnal Design co.</a></h3>
                        </div>
                        <div class="content">
                            <p class="description">Praesent cursus nulla non arcu tempor, ut egestas elit tempus. In ac ex fermentum, gravida felis nec, tincidunt ligula.</p>
                            <div class="star-icon mt-3">
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            </div>
                        </div>
                        </div>
                    </div>
                    <div class="item">
                        <div class="testimonial-item">
                        <div class="img-thumb">
                            <img src="{% static 'img/testimonial/img4.jpg' %}" alt="">
                        </div>
                        <div class="info">
                            <h2><a href="#">Fernanda Anaya</a></h2>
                            <h3><a href="#">Developer</a></h3>
                        </div>
                        <div class="content">
                            <p class="description">Praesent cursus nulla non arcu tempor, ut egestas elit tempus. In ac ex fermentum, gravida felis nec, tincidunt ligula.</p>
                            <div class="star-icon mt-3">
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-filled"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            <span><i class="lni-star-half"></i></span>
                            </div>
                        </div>
                        </div>
                    </div>
                    </div>
                </div>
                </div>
            </div>
            </section>
        ```

    - Template `core/templates/contato.html`
        ```html
        {% load static %}
        <section id="contact" class="section-padding bg-gray">    
            <div class="container">
                <div class="section-header text-center">          
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Countact Us</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row contact-form-area wow fadeInUp" data-wow-delay="0.3s">   
                <div class="col-lg-7 col-md-12 col-sm-12">
                    <div class="contact-block">
                    <form id="contactForm">
                        <div class="row">
                        <div class="col-md-6">
                            <div class="form-group">
                            <input type="text" class="form-control" id="name" name="name" placeholder="Name" required data-error="Please enter your name">
                            <div class="help-block with-errors"></div>
                            </div>                                 
                        </div>
                        <div class="col-md-6">
                            <div class="form-group">
                            <input type="text" placeholder="Email" id="email" class="form-control" name="email" required data-error="Please enter your email">
                            <div class="help-block with-errors"></div>
                            </div> 
                        </div>
                        <div class="col-md-12">
                            <div class="form-group">
                            <input type="text" placeholder="Subject" id="msg_subject" class="form-control" required data-error="Please enter your subject">
                            <div class="help-block with-errors"></div>
                            </div>
                        </div>
                        <div class="col-md-12">
                            <div class="form-group"> 
                            <textarea class="form-control" id="message" placeholder="Your Message" rows="7" data-error="Write your message" required></textarea>
                            <div class="help-block with-errors"></div>
                            </div>
                            <div class="submit-button text-left">
                            <button class="btn btn-common" id="form-submit" type="submit">Send Message</button>
                            <div id="msgSubmit" class="h3 text-center hidden"></div> 
                            <div class="clearfix"></div> 
                            </div>
                        </div>
                        </div>            
                    </form>
                    </div>
                </div>
                <div class="col-lg-5 col-md-12 col-xs-12">
                    <div class="map">
                    <object style="border:0; height: 280px; width: 100%;" data="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d34015.943594576835!2d-106.43242624069771!3d31.677719472407432!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x86e75d90e99d597b%3A0x6cd3eb9a9fcd23f1!2sCourtyard+by+Marriott+Ciudad+Juarez!5e0!3m2!1sen!2sbd!4v1533791187584"></object>
                    </div>
                </div>
                </div>
            </div> 
            </section>
        ```
    
    - Template `core/templates/equipe.html`
        ```html
        {% load static %}
        <section id="team" class="section-padding bg-gray">
            <div class="container">
                <div class="section-header text-center">          
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Meet our team</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row">
                <div class="col-lg-6 col-md-12 col-xs-12">
                    <!-- Team Item Starts -->
                    <div class="team-item wow fadeInRight" data-wow-delay="0.2s">
                    <div class="team-img">
                        <img class="img-fluid" src="{% static 'img/team/team-01.png' %}" alt="">
                    </div>
                    <div class="contetn">
                        <div class="info-text">
                        <h3><a href="#">David Smith</a></h3>
                        <p>Front-end Developer</p>
                        </div>
                        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Quod eos id officiis hic tenetur.</p>
                        <ul class="social-icons">
                        <li><a href="#"><i class="lni-facebook-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-twitter-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-instagram-filled" aria-hidden="true"></i></a></li>
                        </ul>
                    </div>
                    </div>
                    <!-- Team Item Ends -->
                </div>
                <div class="col-lg-6 col-md-12 col-xs-12">
                    <!-- Team Item Starts -->
                    <div class="team-item wow fadeInRight" data-wow-delay="0.4s">
                    <div class="team-img">
                        <img class="img-fluid" src="{% static 'img/team/team-02.png' %}" alt="">
                    </div>
                    <div class="contetn">
                        <div class="info-text">
                        <h3><a href="#">ERIC PETERSON</a></h3>
                        <p>Product Designer</p>
                        </div>
                        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Quod eos id officiis hic tenetur.</p>
                        <ul class="social-icons">
                        <li><a href="#"><i class="lni-facebook-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-twitter-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-instagram-filled" aria-hidden="true"></i></a></li>
                        </ul>
                    </div>
                    </div>
                    <!-- Team Item Ends -->
                </div>
                <div class="col-lg-6 col-md-12 col-xs-12">
                    <!-- Team Item Starts -->
                    <div class="team-item wow fadeInRight" data-wow-delay="0.6s">
                    <div class="team-img">
                        <img class="img-fluid" src="{% static 'img/team/team-03.png' %}" alt="">
                    </div>
                    <div class="contetn">
                        <div class="info-text">
                        <h3><a href="#">DURWIN BABB</a></h3>
                        <p>Lead Designer</p>
                        </div>
                        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Quod eos id officiis hic tenetur.</p>
                        <ul class="social-icons">
                        <li><a href="#"><i class="lni-facebook-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-twitter-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-instagram-filled" aria-hidden="true"></i></a></li>
                        </ul>
                    </div>
                    </div>
                    <!-- Team Item Ends -->
                </div>
                <div class="col-lg-6 col-md-12 col-xs-12">
                    <!-- Team Item Starts -->
                    <div class="team-item wow fadeInRight" data-wow-delay="0.8s">
                    <div class="team-img">
                        <img class="img-fluid" src="{% static 'img/team/team-04.png' %}" alt="">
                    </div>
                    <div class="contetn">
                        <div class="info-text">
                        <h3><a href="#">MARIJN OTTE</a></h3>
                        <p>Lead Designer</p>
                        </div>
                        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Quod eos id officiis hic tenetur.</p>
                        <ul class="social-icons">
                        <li><a href="#"><i class="lni-facebook-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-twitter-filled" aria-hidden="true"></i></a></li>
                        <li><a href="#"><i class="lni-instagram-filled" aria-hidden="true"></i></a></li>
                        </ul>
                    </div>
                    </div>
                    <!-- Team Item Ends -->
                </div>
                </div>
            </div>
        </section>
        ```
    
    - Template `core/templates/features.html`
        ```html
        {% load static %}
        <section id="features" class="section-padding">
            <div class="container">
                <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Awesome Features</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row">
                <div class="col-lg-4 col-md-12 col-sm-12 col-xs-12">
                    <div class="content-left">
                    <div class="box-item wow fadeInLeft" data-wow-delay="0.3s">
                        <span class="icon">
                        <i class="lni-rocket"></i>
                        </span>
                        <div class="text">
                        <h4>Bootstrap 4 Based</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry.</p>
                        </div>
                    </div>
                    <div class="box-item wow fadeInLeft" data-wow-delay="0.6s">
                        <span class="icon">
                        <i class="lni-laptop-phone"></i>
                        </span>
                        <div class="text">
                        <h4>Fully Responsive</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry.</p>
                        </div>
                    </div>
                    <div class="box-item wow fadeInLeft" data-wow-delay="0.9s">
                        <span class="icon">
                        <i class="lni-cog"></i>
                        </span>
                        <div class="text">
                        <h4>HTML5, CSS3 & SASS</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry</p>
                        </div>
                    </div>
                    </div>
                </div>
                <div class="col-lg-4 col-md-12 col-sm-12 col-xs-12">
                    <div class="show-box wow fadeInUp" data-wow-delay="0.3s">
                    <img src="{% static 'img/feature/intro-mobile.png' %}" alt="">
                    </div>
                </div>
                <div class="col-lg-4 col-md-12 col-sm-12 col-xs-12">
                    <div class="content-right">
                    <div class="box-item wow fadeInRight" data-wow-delay="0.3s">
                        <span class="icon">
                        <i class="lni-leaf"></i>
                        </span>
                        <div class="text">
                        <h4>Modern Design</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry</p>
                        </div>
                    </div>
                    <div class="box-item wow fadeInRight" data-wow-delay="0.6s">
                        <span class="icon">
                        <i class="lni-layers"></i>
                        </span>
                        <div class="text">
                        <h4>Multi-purpose Template</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry.</p>
                        </div>
                    </div>
                    <div class="box-item wow fadeInRight" data-wow-delay="0.9s">
                        <span class="icon">
                        <i class="lni-leaf"></i>
                        </span>
                        <div class="text">
                        <h4>Working Contact Form</h4>
                        <p>Lorem Ipsum is simply dummy text of the printing and typesetting industry.</p>
                        </div>
                    </div>
                    </div>
                </div>
                </div>
            </div>
            </section>
        ```

    - Template `core/templates/footer.html`
        ```html
        {% load static %}
        <footer id="footer" class="footer-area section-padding">
            <div class="container">
                <div class="container">
                <div class="row">
                    <div class="col-lg-3 col-md-6 col-sm-6 col-xs-6 col-mb-12">
                    <div class="widget">
                        <h3 class="footer-logo"><img src="{% static 'img/logo.png' %}" alt=""></h3>
                        <div class="textwidget">
                        <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque lobortis tincidunt est, et euismod purus suscipit quis.</p>
                        </div>
                        <div class="social-icon">
                        <a class="facebook" href="#"><i class="lni-facebook-filled"></i></a>
                        <a class="twitter" href="#"><i class="lni-twitter-filled"></i></a>
                        <a class="instagram" href="#"><i class="lni-instagram-filled"></i></a>
                        <a class="linkedin" href="#"><i class="lni-linkedin-filled"></i></a>
                        </div>
                    </div>
                    </div>
                    <div class="col-lg-3 col-md-6 col-sm-12 col-xs-12">
                    <h3 class="footer-titel">Products</h3>
                    <ul class="footer-link">
                        <li><a href="#">Tracking</a></li>
                        <li><a href="#">Application</a></li>
                        <li><a href="#">Resource Planning</a></li>
                        <li><a href="#">Enterprise</a></li>
                        <li><a href="#">Employee Management</a></li>
                    </ul>
                    </div>
                    <div class="col-lg-3 col-md-6 col-sm-12 col-xs-12">
                    <h3 class="footer-titel">Resources</h3>
                    <ul class="footer-link">
                        <li><a href="#">Payment Options</a></li>
                        <li><a href="#">Fee Schedule</a></li>
                        <li><a href="#">Getting Started</a></li>
                        <li><a href="#">Identity Verification</a></li>
                        <li><a href="#">Card Verification</a></li>
                    </ul>
                    </div>
                    <div class="col-lg-3 col-md-6 col-sm-12 col-xs-12">
                    <h3 class="footer-titel">Contact</h3>
                    <ul class="address">
                        <li>
                        <a href="#"><i class="lni-map-marker"></i> 105 Madison Avenue - <br> Third Floor New York, NY 10016</a>
                        </li>
                        <li>
                        <a href="#"><i class="lni-phone-handset"></i> P: +84 846 250 592</a>
                        </li>
                        <li>
                        <a href="#"><i class="lni-envelope"></i> E: contact@uideck.com</a>
                        </li>
                    </ul>
                    </div>
                </div>
                </div>
            </div>
            <div id="copyright">
                <div class="container">
                <div class="row">
                    <div class="col-md-12">
                    <div class="copyright-content">
                        <p>Copyright © 2020 <a rel="nofollow" href="https://uideck.com">UIdeck</a> All Right Reserved</p>
                    </div>
                    </div>
                </div>
                </div>
            </div>
            </footer>
        ```

    - Template `core/templates/hero.html`
        ```html
        {% load static %}
        <div id="hero-area" class="hero-area-bg">
                <div class="container">      
                <div class="row">
                    <div class="col-lg-7 col-md-12 col-sm-12 col-xs-12">
                    <div class="contents">
                        <h2 class="head-title">App, Business & SaaS<br>Landing Page Template</h2>
                        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem repellendus quasi fuga nesciunt dolorum nulla magnam veniam sapiente, fugiat! fuga nesciunt dolorum nulla magnam veniam sapiente, fugiat!</p>
                        <div class="header-button">
                        <a href="#" class="btn btn-common">Download Now</i></a>
                        <a href="#" class="btn btn-border video-popup">Learn More</i></a>
                        </div>
                    </div>
                    </div>
                    <div class="col-lg-5 col-md-12 col-sm-12 col-xs-12">
                    <div class="intro-img">
                        <img class="img-fluid" src="{% static 'img/intro-mobile.png' %}" alt="">
                    </div>            
                    </div>
                </div> 
                </div> 
            </div>
            <!-- Hero Area End -->

            </header>
            <!-- Header Area wrapper End -->
        ```

    - Template `core/templates/index.html`
        ```html
        {% extends 'base.html'  %}
        {% load static %}
        {% block content %}
            <!-- Hero Area Start -->
                {% include 'hero.html' %}
            <!-- Hero Area End -->

            <!-- Services Section Start -->
            {% include 'servicos.html' %}
            <!-- Services Section End -->

            <!-- About Section start -->
            {% include 'sobre.html' %}
            <!-- About Section End -->

            <!-- Features Section Start -->
                {% include 'features.html' %}
            <!-- Features Section End -->

            <!-- Team Section Start -->
                {% include 'equipe.html' %}
            <!-- Team Section End -->

            <!-- Pricing section Start -->
            {% include 'precos.html' %}
            <!-- Pricing Table Section End -->

            <!-- Testimonial Section Start -->
                {% include 'clientes.html' %}
            <!-- Testimonial Section End -->

            <!-- Call To Action Section Start -->
            {% include 'chamada.html' %}
            <!-- Call To Action Section Start -->

            <!-- Contact Section Start -->
            {% include 'contato.html' %}
            <!-- Contact Section End -->

            <!-- Footer Section Start -->
            {% include 'footer.html' %}
            <!-- Footer Section End -->
        {% endblock %}
        ```

    - Template `core/templates/precos.html`
        ```html
        {% load static %}
        <section id="pricing" class="section-padding">
            <div class="container">
                <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Pricing</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row">
                <div class="col-lg-4 col-md-6 col-xs-12">
                    <div class="table wow fadeInLeft" data-wow-delay="1.2s">
                    <div class="icon-box">
                        <i class="lni-package"></i>
                    </div>
                    <div class="pricing-header">
                        <p class="price-value">$10<span> /mo</span></p>
                    </div>
                    <div class="title">
                        <h3>Pro</h3>
                    </div>
                    <ul class="description">
                        <li>1 user</li>
                        <li>10 GB storage</li>
                        <li>Email support</li>
                        <li>Lifetime updates</li>
                    </ul>
                    <button class="btn btn-common">Buy Now</button>
                    </div>
                </div>
                <div class="col-lg-4 col-md-6 col-xs-12 active">
                    <div class="table wow fadeInUp" id="active-tb" data-wow-delay="1.2s">
                    <div class="icon-box">
                        <i class="lni-drop"></i>
                    </div>
                    <div class="pricing-header">
                        <p class="price-value">$35<span> /mo</span></p>
                    </div>
                    <div class="title">
                        <h3>Plus</h3>
                    </div>
                    <ul class="description">
                        <li>10 user</li>
                        <li>30 GB storage</li>
                        <li>Priority email support</li>
                        <li>Lifetime updates</li>
                    </ul>
                    <button class="btn btn-common">Buy Now</button>
                </div>
                </div>
                <div class="col-lg-4 col-md-6 col-xs-12">
                    <div class="table wow fadeInRight" data-wow-delay="1.2s">
                    <div class="icon-box">
                        <i class="lni-star"></i>
                    </div>
                    <div class="pricing-header">
                        <p class="price-value">$150<span> /mo</span></p>
                    </div>
                    <div class="title">
                        <h3>Premium</h3>
                    </div>
                    <ul class="description">
                        <li>Unlimited users</li>
                        <li>Unlimited storage</li>
                        <li>24/7 support</li>
                        <li>Lifetime updates</li>
                    </ul>
                    <button class="btn btn-common">Buy Now</button>
                    </div>
                </div>
                </div>
            </div>
        </section>
        ```

    - Template `core/templates/servicos.html`
        ```html
        {% load static %}
        <section id="services" class="section-padding">
            <div class="container">
            <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Our Services</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
            </div>
            <div class="row">
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.3s">
                    <div class="icon">
                    <i class="lni-cog"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Easy To Used</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.6s">
                    <div class="icon">
                    <i class="lni-stats-up"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Awesome Design</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="0.9s">
                    <div class="icon">
                    <i class="lni-users"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">Easy To Customize</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.2s">
                    <div class="icon">
                    <i class="lni-layers"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">UI/UX Design</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.5s">
                    <div class="icon">
                    <i class="lni-mobile"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">App Development</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                <div class="services-item wow fadeInRight" data-wow-delay="1.8s">
                    <div class="icon">
                    <i class="lni-rocket"></i>
                    </div>
                    <div class="services-content">
                    <h3><a href="#">User Friendly interface</a></h3>
                    <p>Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida. </p>
                    </div>
                </div>
                </div>
            </div>
            </div>
        </section>
        ```

    - Template `core/templates/sobre.html`
        ```html
        {% load static %}
        <div class="about-area section-padding bg-gray">
            <div class="container">
                <div class="row">
                <div class="col-lg-6 col-md-12 col-xs-12 info">
                    <div class="about-wrapper wow fadeInLeft" data-wow-delay="0.3s">
                    <div>
                        <div class="site-heading">
                        <p class="mb-3">Manage Statistics</p>
                        <h2 class="section-title">Detailed Statistics of your Company</h2>
                        </div>
                        <div class="content">
                        <p>
                            Praesent imperdiet, tellus et euismod euismod, risus lorem euismod erat, at finibus neque odio quis metus. Donec vulputate arcu quam. Morbi quis tincidunt ligula. Sed rutrum tincidunt pretium. Mauris auctor, purus a pulvinar fermentum, odio dui vehicula lorem, nec pharetra justo risus quis mi. Ut ac ex sagittis, viverra nisl vel, rhoncus odio.
                        </p>
                        <a href="#" class="btn btn-common mt-3">Read More</a>
                        </div>
                    </div>
                    </div>
                </div>
                <div class="col-lg-6 col-md-12 col-xs-12 wow fadeInRight" data-wow-delay="0.3s">
                    <img class="img-fluid" src="{% static 'img/about/img-1.png' %}" alt="" >
                </div>
                </div>
            </div>
        </div>
        ```
    
    - Rodar projeto para testar
    </p>

    </details> 

    ---

6. <span style="color:383E42"><b>Definindo os Modelos, Funções e Criando Super Usuário</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Editado templates - Tradução de alguns textos

    - Função `get_file_path` em `core/models.py`
        >Cria nome aleatório para o arquivo de imagem feito upload
        Obs.: StdImageField acrescenta código aleatório a nome de arquivo, caso exista arquivo com mesmo nome. Então não precisariámos da função. Mas a função nos permite mais controle/edição
        ```python
        def get_file_path(_instance, filename):
            # Captura extenção do arquivo
            ext = filename.split('.')[-1]
            # Gera um id/código aleatório
            filename = f'{uuid.uuid4()}.{ext}'
            return filename
        ```

    - Model `Base`
        ```python
        class Base(models.Model):
            criados = models.DateField('Criação', auto_now_add=True)
            modificado = models.DateField('Atualização', auto_now=True)
            ativo = models.BooleanField('Ativo?', default=True)

            class Meta:
                abstract = True
        ```

    - Model `Servico`
        ```python
        class Servico(Base):
            ICONE_CHOICES = (
                ('lni-cog', 'Engrenagem'),
                ('lni-stats-up', 'Gráfico'),
                ('lni-users', 'Usuários'),
                ('lni-layers', 'Design'),
                ('lni-mobile', 'Mobile'),
                ('lni-rocket', 'Foguete'),
            )
            servico = models.CharField('Serviço', max_length=100)
            descricao = models.TextField('Descrição', max_length=200)
            icone = models.CharField('Icone', max_length=12, choices=ICONE_CHOICES)

            class Meta:
                verbose_name = 'Serviço'
                verbose_name_plural = 'Serviços'

            def __str__(self):
                return self.servico
        ```

    - Model  `Cargo`
        ```python
        class Cargo(Base):
            cargo = models.CharField('Cargo', max_length=100)

            class Meta:
                verbose_name = 'Cargo'
                verbose_name_plural = 'Cargos'

            def __str__(self):
                return self.cargo

        ```
    - Model `Funcionario`
        ```python
        class Funcionario(Base):
            nome = models.CharField('Nome', max_length=100)
            cargo = models.ForeignKey('core.Cargo', verbose_name='Cargo', on_delete=models.CASCADE)
            bio = models.TextField('Bio', max_length=200)
            imagem = StdImageField('Imagem', upload_to=get_file_path, variations={'thumb': {'width': 480, 'height': 480, 'crop': True}})
            facebook = models.CharField('Facebook', max_length=100, default='#')
            twitter = models.CharField('Twitter', max_length=100, default='#')
            instagram = models.CharField('Instagram', max_length=100, default='#')

            class Meta:
                verbose_name = 'Funcionário'
                verbose_name_plural = 'Funcionários'

            def __str__(self):
                return self.nome
        ```

        - Executar `migrations` e `migrate` 
            >Para criação de arquivo de migração e criação das tabelas no banco
            ```bash
            python manage.py makemigrations
            python manage.py migrate
            ```

        - Criar super `usuário django`
            >Informar nome, email e senha
            ```bash
            python manage.py createsuperuser
            ```
    </p>

    </details> 

    ---

7. <span style="color:383E42"><b>Incluir Modelos ao Painel de Administração e Inserir Dados</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Em `core/admin.py`
        ```python
        from django.contrib import admin

        from .models import Cargo, Servico, Funcionario


        @admin.register(Cargo)
        class CargoAdmin(admin.ModelAdmin):
            list_display = ('cargo', 'ativo', 'modificado')


        @admin.register(Servico)
        class ServicoAdmin(admin.ModelAdmin):
            list_display = ('servico', 'icone', 'ativo', 'modificado')


        @admin.register(Funcionario)
        class FuncionarioAdmin(admin.ModelAdmin):
            list_display = ('nome', 'cargo', 'ativo', 'modificado')
        ```

    - Cadastrar serviços
        ```
        Serviço: Automação Industrial
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: Engrenagem
        
        Serviço: Desing Gráfico
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: Design

        Serviço: Suporte Humanizado
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: Usuários

        Serviço: UI/UX DESIGN Criativo
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: De sign

        Serviço: Desenvolvimento Mobile
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: Design
        
        Serviço: Sistemas Escaláveis
        Descrição: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Icone: Foguete
        ```

    - Inserir cargos
        ```
        Cargo: Programador Backend
        Cargo: Designer
        Cargo: Estagiário
        ```

    - Inserir Funcionários
        ```
        Nome: Paula Fernandes
        Cargo: Programador Backend
        Bio: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Imagem: team-04

        Nome: Felipe Silva
        Cargo: Estagiário
        Bio: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Imagem: team-04

        Nome: Felicity Jones
        Cargo: Designer
        Bio: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Imagem: team-02

        Nome: Cristiano sspectro
        Cargo: Programador Backend
        Bio: Ut maximus enim dolor. Aenean auctor risus eget tincidunt lobortis. Donec tincidunt bibendum gravida.
        Imagem: team-03
        ```

    </p>

    </details> 

    ---

8. <span style="color:383E42"><b>Context Manager Com TemplateView - Envio de Dados ao Template</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>
    
    - Configurando view `IndexView` para envio de do banco de dados para o template
        ```python
        from django.views.generic import TemplateView

        from .models import Servico, Funcionario

        from .models import Servico, Funcionario

        class IndexView(TemplateView):
            template_name = 'index.html'

            def get_context_data(self, **kwargs):
                context = super(IndexView, self).get_context_data(**kwargs)
                context['servicos'] = Servico.objects.order_by('?').all()
                context['funcionarios'] = Funcionario.objects.order_by('?').all()
                return context
        ```

    - Editar template `core/templates/servicos.html`
        >Recebe dados (do banco de dados) enviados pela view
        ```html
        {% load static %}
        <section id="services" class="section-padding">
            <div class="container">
                <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Nossos Serviços</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row">

                {% for s in servicos %}
                <!-- Services item -->
                <div class="col-md-6 col-lg-4 col-xs-12">
                    <div class="services-item wow fadeInRight" data-wow-delay="0.3s">
                    <div class="icon">
                        <i class="{{ s.icone }}"></i>
                    </div>
                    <div class="services-content">
                        <h3><a href="#">{{ s.servico }}</a></h3>
                        <p>{{ s.descricao }}</p>
                    </div>
                    </div>
                </div>
                {% endfor %}
                </div>
            </div>
        </section>
        ```
    
    - Editar template `core/templates/equipe.html` 
        >Utiliza dados do banco de dados
        ```html
        {% load static %}
        <section id="team" class="section-padding bg-gray">
            <div class="container">
                <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Conheça Nossa Equipe</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row">
                {% for f in funcionarios %}
                <div class="col-lg-6 col-md-12 col-xs-12">
                    <!-- Team Item Starts -->
                    <div class="team-item wow fadeInRight" data-wow-delay="0.2s">
                    <div class="team-img">
                        <img class="img-fluid" src="{{ f.imagem.thumb.url }}" alt="{{ f.nome }}">
                    </div>
                    <div class="contetn">
                        <div class="info-text">
                        <h3><a href="#">{{ f.nome }}</a></h3>
                        <p>{{ f.cargo }}</p>
                        </div>
                        <p>{{ f.bio }}</p>
                        <ul class="social-icons">
                        <li><a href="{{ f.facebook }}"><i class="lni-facebook-filled" aria-hidden="true"></i></a></li>
                        <li><a href="{{ f.twitter }}"><i class="lni-twitter-filled" aria-hidden="true"></i></a></li>
                        <li><a href="{{ f.instagram }}"><i class="lni-instagram-filled" aria-hidden="true"></i></a></li>
                        </ul>
                    </div>
                    </div>
                    <!-- Team Item Ends -->
                </div>
                {% endfor %}
                </div>
            </div>
            </section>
        ```

    </p>

    </details> 

    ---

9. <span style="color:383E42"><b>Formulários e Class Based Views</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>
    - Criar arquivo `core/forms.py` que irá conter os formulários

    - Criar formulário `ContatoForm`
        ```python
        from django import forms
        from django.core.mail.message import EmailMessage


        class ContatoForm(forms.Form):
            nome = forms.CharField(label='Nome', max_length=100)
            email = forms.EmailField(label='E-mail', max_length=100)
            assunto = forms.CharField(label='Assunto', max_length=100)
            mensagem = forms.CharField(label='Mensagem', widget=forms.Textarea())

            def send_mail(self):
                nome = self.cleaned_data['nome']
                email = self.cleaned_data['email']
                assunto = self.cleaned_data['assunto']
                mensagem = self.cleaned_data['mensagem']

                conteudo = f'Nome: {nome}\nE-mail: {email}\nAssunto: {assunto}\nMensagem: {mensagem}'

                mail = EmailMessage(
                    subject=assunto,
                    body=conteudo,
                    from_email='contato@fusion.com.br',
                    to=['contato@fusion.com.br',],
                    headers={'Reply-To': email}
                )
                mail.send()  
        ```
    
    - Eição dde `core/views.py`
        >Inclusão classe `ContatoForm`, configuração para retorno a página `index.html`
        Incluído validação para o formulário
        ```python
        from django.views.generic import FormView
        from django.urls import reverse_lazy
        from django.contrib import messages

        from .models import Servico, Funcionario
        from .forms import ContatoForm


        class IndexView(FormView):
            template_name = 'index.html'
            form_class = ContatoForm
            success_url = reverse_lazy('index')

            def get_context_data(self, **kwargs):
                context = super(IndexView, self).get_context_data(**kwargs)
                context['servicos'] = Servico.objects.order_by('?').all()
                context['funcionarios'] = Funcionario.objects.order_by('?').all()
                return context

            def form_valid(self, form, *args, **kwargs):
                form.send_mail()
                messages.success(self.request, 'E-mail enviado com sucesso')
                return super(IndexView, self).form_valid(form, *args, **kwargs)

            def form_invalid(self, form, *args, **kwargs):
                messages.error(self.request, 'Erro ao enviar e-mail')
                return super(IndexView, self).form_invalid(form, *args, **kwargs)
        ```
    
    - Editar template `core/templates/contato.html`
        >Os valores vindos da view são convertidos/usados nos campos do form
        ```html
        {% load static %}
        <section id="contact" class="section-padding bg-gray">
            <div class="container">
                <div class="section-header text-center">
                <h2 class="section-title wow fadeInDown" data-wow-delay="0.3s">Contate-nos</h2>
                <div class="shape wow fadeInDown" data-wow-delay="0.3s"></div>
                </div>
                <div class="row contact-form-area wow fadeInUp" data-wow-delay="0.3s">
                <div class="col-lg-7 col-md-12 col-sm-12">
                    <div class="contact-block">
                    <form id="contato" method="post" action="{% url 'index' %}" autocomplete="off">
                        {% csrf_token %}
                        <div class="row">
                        <div class="col-md-6">
                            <div class="form-group">
                            <input type="text" class="form-control" id="nome" name="nome" placeholder="Nome" required data-error="Please enter your name">
                            <div class="help-block with-errors"></div>
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="form-group">
                            <input type="text" placeholder="E-mail" id="email" class="form-control" name="email" required data-error="Please enter your email">
                            <div class="help-block with-errors"></div>
                            </div>
                        </div>
                        <div class="col-md-12">
                            <div class="form-group">
                            <input type="text" placeholder="Assunto" id="assunto" name="assunto" class="form-control" required data-error="Please enter your subject">
                            <div class="help-block with-errors"></div>
                            </div>
                        </div>
                        <div class="col-md-12">
                            <div class="form-group">
                            <textarea class="form-control" id="mensagem" placeholder="Mensagem" name="mensagem" rows="7" data-error="Write your message" required></textarea>
                            <div class="help-block with-errors"></div>
                            </div>
                            <div class="submit-button text-left">
                            <button class="btn btn-common" id="form-submit" type="submit">Enviar e-mail</button>
                            <div id="msgSubmit" class="h3 text-center hidden"></div>
                            <div class="clearfix"></div>
                            </div>
                        </div>
                        </div>
                    </form>
                    </div>
                </div>
                <div class="col-lg-5 col-md-12 col-xs-12">
                    <div class="map">
                    <object style="border:0; height: 280px; width: 100%;" data="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d34015.943594576835!2d-106.43242624069771!3d31.677719472407432!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x86e75d90e99d597b%3A0x6cd3eb9a9fcd23f1!2sCourtyard+by+Marriott+Ciudad+Juarez!5e0!3m2!1sen!2sbd!4v1533791187584"></object>
                    </div>
                </div>
                </div>
            </div>
        </section>
        ```

    - Incluir código para exibir mensagens no template `core/templates/hero.html`
        ```html
        <!-- .... -->
        <div class="container">
            {% if messages %}
            {% for m in messages %}
                <div class="alert alert-{{ m.tags }}">
                <button type="button" class="close" data-dismiss="alert"></button>
                <strong>{{ m }}</strong>
                </div>
            {% endfor %}
            {% endif %}
        </div>
        </header>
        <!-- Header Area wrapper End -->
        ```
    
    - Inclusão de configuração para envio de e-mail em `fusion/settings.py`
        ```python
        # Email teste console
        # EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

        """
        # Email produção
        EMAIL_HOST = 'localhost'
        EMAIL_HOST_USER = 'no-reply@seudominio.com.br'
        EMAIL_PORT = 587
        EMAIL_USE_TSL = True
        EMAIL_HOST_PAS########SWORD = 'suasenha'
        DEFAULT_FROM_EMAIL = 'contato@seudominio.com.br'
        """
        ```

    </p>

    </details>
    
    ---

10. <span style="color:383E42"><b>Testando Projeto</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    - Remover arquivo `core/tests.py`

    - Instalar o `model_mommy` e `coverage`
        [Documentação model_mommy](https://model-mommy.readthedocs.io/en/latest/basic_usage.html)
        [Documentação coverage](https://coverage.readthedocs.io/en/7.3.2/)
        ```bash
        pip install model_mommy
        pip install coverage
        pip freeze > requirements.txt
        ```
    
    - Criar arquivo `.../django-avancado-postgresql/.coveragerc`
        `source = .` indica que deve testar tudo que está na raiz. Sem essa indicação testaria as biblitecas na `venv` também.
        `omit =` indica os aquivos que não precisa testar
        ```
        [run]
        source = .

        omit =
            */__init__.py
            */settings.py
            */manage.py
            */wsgi.py
            */apps.py
            */urls.py
            */admin.py
            */migrations.py
            */tests/*
        ```

    - Incluir `htmlcov/*` ao `.gitignore` testar comandos `coverage`
        Esse diretório é criado ao utilizar o coverage - Gera relatório de testes em html.
        Caso não container postgres não esteja rodando, deve iniciar container primeiro.

        GitIgnore
        ```gitignore
        #...
        htmlcov/*

        ```

        ```bash
        sudo docker start fusion-postgres
        coverage run manage.py test
        coverage html
        cd htmlcov
        python -m http.server
        ```

    - Criar diretório e arquivo `core/tests/test_models.py`
        Criar método `GetFilePathTestCase` 
        ```python
        import uuid
        from django.test import TestCase
        from model_mommy import mommy

        from core.models import get_file_path


        class GetFilePathTestCase(TestCase):

            def setUp(self):
                self.filename = f'{uuid.uuid4()}.png'

            # Todo método test começa com a palavra "test_"
            def test_get_file_path(self):
                arquivo = get_file_path(None, 'teste.png')
                self.assertTrue(len(arquivo), len(self.filename))


        class ServicoTestCase(TestCase):

            def setUp(self):
                self.servico = mommy.make('Servico')

            def test_str(self):
                self.assertEquals(str(self.servico), self.servico.servico)


        class CargoTestCase(TestCase):

            def setUp(self):
                self.cargo = mommy.make('Cargo')

            def test_str(self):
                self.assertEquals(str(self.cargo), self.cargo.cargo)


        class FuncionarioTestCase(TestCase):

            def setUp(self):
                self.funcionario = mommy.make('Funcionario')

            def test_str(self):
                self.assertEquals(str(self.funcionario), self.funcionario.nome)
        ```

        Remover diretório `htmlcov` e executar teste. O diretório será criado novamente com resultado do teste.
        ```bash
        rm -rf htmlcov
        coverage run manage.py test
        coverage html
        cd htmlcov/
        python -m http.server
        ```

    - Criar arquivo `core/tests/test_forms.py`
        ```python
        from django.test import TestCase

        from core.forms import ContatoForm


        class ContatoFormTestCase(TestCase):

            def setUp(self):
                self.nome = 'Felicity Jones'
                self.email = 'felicity@gmail.com'
                self.assunto = 'Um assunto qualquer'
                self.mensagem = 'Uma mensagem qualquer'

                self.dados = {
                    'nome': self.nome,
                    'email': self.email,
                    'assunto': self.assunto,
                    'mensagem': self.mensagem
                }

                self.form = ContatoForm(data=self.dados)  # ContatoForm(request.POST)
            # Todo método test começa com a palavra "test_"
            def test_send_mail(self):
                form1 = ContatoForm(data=self.dados)
                form1.is_valid()
                res1 = form1.send_mail()

                form2 = self.form
                form2.is_valid()
                res2 = form2.send_mail()

                self.assertEquals(res1, res2)
        ```

    - Criar arquivo `core/tests/test_views.py`
        ```python
        from django.test import TestCase
        from django.test import Client
        from django.urls import reverse_lazy


        class IndexViewTestCase(TestCase):

            def setUp(self):
                self.dados = {
                    'nome': 'Felicity Jones',
                    'email': 'felicity@gmail.com',
                    'assunto': 'Meu assunto',
                    'mensagem': 'Minha mensagem'
                }
                self.cliente = Client()

            def test_form_valid(self):
                request = self.cliente.post(reverse_lazy('index'), data=self.dados)
                self.assertEquals(request.status_code, 302)

            def test_form_invalid(self):
                dados = {
                    'nome': 'Felicity Jones',
                    'assunto': 'Meu assunto'
                }
                request = self.cliente.post(reverse_lazy('index'), data=dados)
                self.assertEquals(request.status_code, 200)
        ```


    </p>

    </details> 

    ---

11. <span style="color:383E42"><b>Publicando</b></span>
    <details><summary><span style="color:Chocolate">Detalhes</span></summary>
    <p>

    >Futuramente incluirei opções de publicação do projeto

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