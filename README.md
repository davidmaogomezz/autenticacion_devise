En esta entrada mostraremos como tener un sistema de registro y autenticación usando la gema Devise. Vamos a crear una aplicación de prueba llamada ```autenticacion```:

```sh
> rails new autenticacion
```

No olvidemos desplazarnos dentro de nuestro aplicación:

```sh
> cd autenticacion
```

Agregamos a nuestro Gemfile la gema Devise ```gem 'devise'```. Devise es una gema muy completa, [en este link puedes encontrar toda la información sobre devise](http://https://github.com/plataformatec/devise). Y procedemos a hacer la instalación con

```sh
> bundle install
```

Ya tenemos instalado Devise en nuestro proyecto. Ahora debemos ejecutar el generador de devise en nuestro proyecto: 

```sh
> rails generate devise:install
```

Veremos en nuestra consola la siguiente información:

```sh
Running via Spring preloader in process 3687
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

Antes de seguir las anteriores intrucciones debemos crear nuestro modelo User:

```sh
> rails generate devise User
Running via Spring preloader in process 2482
      invoke  active_record
      create    db/migrate/20180118173812_devise_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      insert    app/models/user.rb
       route  devise_for :users
```

Ejecutamos la migración

```sh
> rake db:migrate
== 20180118173812 DeviseCreateUsers: migrating ================================
-- create_table(:users)
   -> 0.0030s
-- add_index(:users, :email, {:unique=>true})
   -> 0.0062s
-- add_index(:users, :reset_password_token, {:unique=>true})
   -> 0.0015s
== 20180118173812 DeviseCreateUsers: migrated (0.0124s) =======================
```

Veamos ahora la tabla que devise ha creado por nosotros:

```ruby
  create_table "users", force: :cascade do |t|
    t.string "email", default: "", null: false
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.integer "sign_in_count", default: 0, null: false
    t.datetime "current_sign_in_at"
    t.datetime "last_sign_in_at"
    t.string "current_sign_in_ip"
    t.string "last_sign_in_ip"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["email"], name: "index_users_on_email", unique: true
    t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true
  end
```

Ahora sí podemos seguir las instrucciones que Devise nos da para tener un funcionamiento correcto de esta gema:

Lo primero que nos pide es que agreguemos la linea ```config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }``` en el archivo ```config/environments/development.rb```

Lo segundo que nos pide es que tengamos un root diferente al que por defecto tiene nuestra aplicación. Entonces:

```sh
> rails g controller home index
```

El anterior comando nos creara un controlador home con una vista index. Ahora debemos ir a nuestro archivo ```config/routes.rb``` y decirle a nuestra aplicación cual será nuestra nueva vista root. Entonces veamos como debería quedar el archivo ```config/routes.rb```:

```ruby
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  root to: "home#index"
end
```

Lo tercero que nos pide Devise es agreguemos

```html
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

Al archivo ```app/views/layouts/application.html.erb``` de esta manera Devise podrá mostrar mensajes al usuario cuando se produce alguna excepción haciendo el registro o iniciado sessión.

Para finalizar devise nos dice que podemos generar cada una de las vistas como la de registrar usuario o iniciar sessión corriendo:

```sh
> rails g devise:views
```

Esto nos creará una carpeta llamada ```devise``` en la carpeta ```views``` veamos el contenido de dicha carpeta

```sh
> cd app/views/devise && ls
confirmations/  mailer/  passwords/  registrations/  sessions/  shared/  unlocks/
```

Ahora veamos las rutas que posee nuestra aplicación:

```sh
> rake routes
                  Prefix Verb   URI Pattern                    Controller#Action
        new_user_session GET    /users/sign_in(.:format)       devise/sessions#new
            user_session POST   /users/sign_in(.:format)       devise/sessions#create
    destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
       new_user_password GET    /users/password/new(.:format)  devise/passwords#new
      edit_user_password GET    /users/password/edit(.:format) devise/passwords#edit
           user_password PATCH  /users/password(.:format)      devise/passwords#update
                         PUT    /users/password(.:format)      devise/passwords#update
                         POST   /users/password(.:format)      devise/passwords#create
cancel_user_registration GET    /users/cancel(.:format)        devise/registrations#cancel
   new_user_registration GET    /users/sign_up(.:format)       devise/registrations#new
  edit_user_registration GET    /users/edit(.:format)          devise/registrations#edit
       user_registration PATCH  /users(.:format)               devise/registrations#update
                         PUT    /users(.:format)               devise/registrations#update
                         DELETE /users(.:format)               devise/registrations#destroy
                         POST   /users(.:format)               devise/registrations#create
              home_index GET    /home/index(.:format)          home#index
                    root GET    /                              home#index
```
Debemos agregar ```before_action :authenticate_user!``` al controlador que deseamos proteger con autenticación, en nuestro caso será el controlador ```HomeController``` quedando así:

```ruby
class HomeController < ApplicationController
  before_action :authenticate_user!
  def index
  end
end
```

Probemos visitando por ejemplo ```https://localhost:3000/```