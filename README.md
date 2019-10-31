# RoR-Step-by-Step

### 1 Create a new project
```
rails new <proj-name> -d postgresql -T
```
Notes:
* Such commands create a folder named `<proj-name>` inside of which the new Rails project will be created
* Inside the newly created project, a git repo will be initialized (meaning that `git init` is already issued)
* SQlite3 database is default database used when a new Ruby on Rails application is created. If a different database is required, it can be specified with an adhoc option `-d`. To instruct Rails to ignore any database, at all, option `-O` can be used.
* By providing the extra `-T`, the Rails project will skip the generation of any Test files.

### 2 Installing extra gems
```
bundle add <gem-name>
```
Adds the named gem to the Gemfile and run `bundle install`.
`bundle install` can be avoided by using the flag `--skip-install`.

### 2.X Authentification Devise Gem

Devise is a gem that adds authentication to Rails applications and uses the *Cookie / Session* method of web application authentification. For Devise, Authentication is saved in the session on the server and referenced by a cookie saved in the users browser. To install Devise:

*Add Gem*
```
bundle add devise
```
*Open Rails App*
```
code .
```
*Install Gem*
```
rails generate devise:install
```
Devise then generates 4 instructions to be manually modified inside the Rails App. They are (summarised):

1. Define default url options in development environments files `./config/environments/development.rb`

Add this line above the `end` in Rails App:
```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

2. Define root_url in `./config/routes.rb.`
For example: `get "/", to: "pages#home", as: "root"`

3. Add *flash messages* in `./app/views/layouts/application.html.erb.` between `<body>` body tags but above `<%= yield %>`.

```ruby
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

4. Copy Devise views to Rails App. Devise views contain pre-built messaging functionality.

```
rails g devise:views
```

Next, generate the loggedin/logged out Model
```
rails generate devise User
rails db:migrate
```

Check out Devise User routes by running
```
rails routes
```

Notes:
- Initial User Routes do not need to be manually created as Devise creates User routes in the `./config/routes.rb file` at the time of User Model generation and injects this line of code into the routes file `devise_for :users`.

### 2.X Stripe Gem

Web apps taking payment with no backend to hold credit card information will require a payment merchant such as Stripe (that handles PCI compliance).

1. *Install Stripe Gem*
```
bundle add stripe
```
2. Create a new file `stripe.rb` in `./config/initializers`. Add the following code (Rails 5.2 compliant):

```ruby
Stripe.api_key = Rails.application.credentials.dig(:stripe, :secret)
```

3. Login to Stripe>Developers>APIkeys *(location of publishable and secret keys)*

4. Press 'reveal' on secret key and copy it

5. Add command `EDITOR="code --wait" rails credentials:edit`, press *upon pressing enter button YAML file will pop up* - this is where the following information goes.

6. On a new line underneath the YAML file code, add the following:

```ruby
stripe:
  secret: <add secret key>
  public: <add public key>
```
*There must be 2 spaces before the words `stripe` and `public` and one space after their corresponding `:`.

7. Go to Stripe, copy and paste public (publishable) key.

8. Save the keys by exiting the opened credentials file *CLI will annotate the file saving by adding `New credentials encrypted and saved.`

### 3 Setting up the project
```
bundle install
```
After a project is created (or cloned from GitHub) it’s necessary to check whether any gem is needed. This command reads from the Gemfile.

### 4 Database Creation
```
rails db:create
```
When a Rails application is created for the first time, it will not have a database yet. To be create one, those commands are needed.
The command initializes/creates a database (named after the project). To be more accurate, it creates two versions of the same DB. one for development and one for testing purposes.

### 4.X Seeding the database

#### Setup
`rails db:setup` creates the database, loads the schema, and initialises the schema with the seed data (Faker Gem).

#### Resetting
Drop a database and set it up again using `rails db:reset`. This is the same as running:

```ruby
rails db:drop
rails db:setup
```

### 5 Model generation
```
rails g model <Entity> <column>:<datatype> <column>:<datatype>
```
This is a Model scaffold generator that produces two effects:
1. It generates a new migration file (under `./db/migrate/`), containing the code to create a new table (named `<Entities>`) with the specified columns.
2. It generates an active-record file (under `./app/models/`)

Notes:
* Entity must be capitalized
* Entity must be singular
* If Entity represents a join table, the naming convention becomes EntitiesEntity. The priority of Entities over Entity is determined by the alphabetic order. (e.g. If we want to create s join table for 'Books' and 'Genres', the Entity name will be 'BooksGenre' as 'B' comes before 'G')
* Although the Entity keyword is singular, the migration file produces a table with a name in plural.
* Models must be added to Rails (ran in CLI) in the following order: (1) tables have no foreign keys, (2) tables have foreign keys. This is because Rails cannot implement `:references` for the `foreign_key` if the table has no column to reference to.
* `foreign_key` columns reference the `id` of another table and are represented as `user:references` not `user_id:references`.

##### More examples of Model generation
```
rails g model Genre name:string                                                #table with 1 column
rails g model Book title:string
rails g model Author name:string date_of_birth:date                            #table with multiple columns
rails g model BooksGenre book:references genre:references                      #join table w/o additional columns
rails g model Movie title:string rating:integer
rails g model Actor name:string birthdate:date
rails g model ActorsMovie movie:references actor:references character:string   #join table w additional columns
rails g model Image imageable:references{polymorphic} url:string               #polymorphic table
```
### 5.x Rails 5 native field types

| field types | Description |
| ------ | ------ |
| :integer | whole numbers |
| :primary_key | unique key that can uniquely identify each row in a table |
| :decimal | for decimals, more accurate but slower processing time than :float |
| :float | for decimals, less accurate but faster processing time than :decimal |
| :boolean | stores true or false values |
| :binary | is for storing data such as images, audio, or movies |
| :string | for titles or names <=255 characters |
| :text | for paragraphs or descriptions <=30K characters |
| :date | store date only |
| :time | store time only |
| :datetime | store the date and time, unaffected by time zone, format: YYYY-MM-DD HH:MM:SS |
| :references | generates a foreign key column e.g.  album:references creates album_id, use in 'belongs_to' association, supports polymorphism |

### 6 Database Migration
```
rails db:migrate
```
The command applies to any pending migrations. Pending migration means setting up tables in the database (or alter tables or drop tables). When running the migration command, it will look in db/migrate/ for any Ruby files and execute them starting from the oldest.

Notes:
- `rails db:migrate` checks which missing migrations still need to be applied to the database without considering the previouse migrations.

### Check Database Migration Status
```
rails db:migrate:status
```

If you are not sure whether you have run the Rails migration or not, you can use this command to check the migration status.

If the status is down, then that means you have not run 'rails db:migrate' for this migration.


    Status   Migration ID    Migration Name
    --------------------------------------------------
    up     20191014002535  Create books
    up     20191014004145  Add overview to books
    up     20191014005002  Add publisher and length to books
    down    20191014005310  Add date to books                 #you haven't run db:migrate here


##### Migration generation
Another way to generate migrations (other than from the Model generation) is to submit the commands such as:
```
# Table creations
rails g migration CreateAuthors name:string date_of_birth:date
rails g migration CreateBooks title:string author:string rating:integer

# Adding columns
rails g migration AddOverviewToBooks overview:string

# Adding multiple columns
rails g migration AddOverviewToBooks overview:string genre:string blurb:text

# Removing columns
rails g migration RemovePublisherFromBooks column:datatype

# Replacing column (new migration)
rails g migration RemoveTypeFromIngredients type:integer
rails g migration AddKindToIngredients kind:integer
rails db:migrate

# Go to VScode, change models containing old column name
```

#### 6.X Dropping a table

Create a migration to manully drop the table:: 

```bash
rails g migration Drop<Books>Table
rake db:migrate
```

This generates the empty `.rb` file in `./db/migrate/` that mmust be filled to drop the table.

Notes:
- Everytime a new migration is created, it is added to the `./db/migrate/` directory ready to be synched with the database after running `rails db:migrate`.
- Adding a column with an underscore such as `first_name` does not include the underscore: `rails g migration AddFirstNameToUser first_name:string`.

### 7 Model association
Even the simplest databases might have models/tables with some sort of relationships with each other.
Examples are:
* one to one
* one to many
* many to many

Even though at the Database level those relationships are already set up through FK, it's necessarily to replicate the integrity logic in the application layer. To do so, few changes have to be applied in the ruby files under `./app/models/`

```ruby
#[one to one]

model/address.rb

    belongs_to :author

model/author.rb

    has_one :address      #For 'has_one' relationship, the symbol ':address' is singular.

#[one to many] We assume a book only has an author in this case.

model/book.rb

    belongs_to :author

model/author.rb

    has_many :books       #For 'has_many' relationship, the symbol ':books' is plural.

#[join table: many to many]

model/movie.rb
    has_many :actors_movies
    has_many :actors, through: :actors_movies

model/actor.rb
    has_many :actors_movies
    has_many :movies, through: :actors_movies

```

Notes
* `has_one` is followed by a singular symbol
* `has_many` is followed by a plural symbol
* a join table must have two `belongs_to` keywords
* Sometime it's tricky to distinguish when to use `has_one` from `belongs_to`.
The rule of thumb is "Foo belongs to Bar if table Foo has a bar_id column".


Validation
Sometimes it's required to put in place an extra validation (aka constraints) on one or more column of a model. For example, there might be a reason only accept a record where a numeric value falls between a range. Or when a new record has a mandatory field (mandatory as in cannot be left empty/nil). An example on how to enforce validation rules, it's required to add similar instructions
```
validates :name, :price, presence: true
validates :price, numericality: {greater_than: 10}
```

### 8 Routes configuration

Routes are configured inside `./config/routes.rb`

There are two possible options.
##### Option 1
```ruby
1. get "/entities", to: "entities#index", as: "entities"
2. get "/entities/new", to: "entities#new", as: "new_entity"
3. post "/entities", to: "entities#create"
4. get "/entities/:id", to: "entities#show", as: "entity"
5. get "/entities/:id/edit", to: "entities#edit", as: "edit_entity"
6. put "/entities/:id", to: "entities#update"
7. patch "/entities/:id", to: "entities#update"
8. delete "/entities/:id", to: "entities#destroy"
```
Notes
* must include one of or both routes 6 & 7
* :id routes return anything (wildcard) and are to be placed under the root and name-associated route paths i.e. "/entities" & "/entities/new"
* routes 3, 6, 7, & 8 do not need prefixes as routes 1, 2, 4, & 5 point to the same path and hold the prefix
* only 'get' routes will get a 'xxx.html.erb' file in our View
* if you want to check whether you have set up routes correctly, run 'rails routes' in your terminal. If there is an error message, that means you did not set up your routes correctly.

##### Option 2
Resource routing allows you to quickly declare all of the common routes for a given resourceful controller. Instead of declaring separate routes for your index, show, new, edit, create, update and destroy actions, a resourceful route declares them in a single line of code.

```ruby
resources :entities
```

### 9 Controller creation

Controller files have to be manually created under `./app/controller/`. The naming convention for such files is `entities_controller.rb` (everything lower-case and plural form).

<b>!!! Important !!!</b> 
- You will only need to manually create a model if you initially run 'rails g migration' to create a table(e.g. rails g migration AddOverviewToBooks overview:text).
- If you run 'rails g model' to create a table(e.g. rails g model Books overview:text), you are NOT required to manually create a model. The 'rails g model' command will run rails model and rails migration at the same time for you.
- In this case, we will usually run 'rails g model' to create a table. (You will only 'rails g migration' to create a table when you do NOT need a model.)

The content of each controller starts with a class declaration having the following structure:

```ruby
class MilkshakesController < ApplicationController

end
```

##### Methods definition
A controller is expected to have a method for each route defined in the `routes.rb` file.

Example: if the route-file contains an action corresponding to `entities#index`, then the controller-file is supposed to implement the `index` method.

```ruby
class EntitiesController < ApplicationController
    def index
    end
end
```

### 10 View creation
In the MVC design pattern, Views are the interface of an application. In Ruby on Rails, that refers to the actual webpage returned to a user.

Views are usually placed under `./app/views` and they connect to the Controller via manually created directories and HTML.erb files of the routes. The directories are named after the Controllers. SASS files for the views are located in `./app/assets/stylesheets`.

Example:
Route: get `"/milkshakes", to: "milkshakes#index", as: "milkshakes"`
**M**odel: `./app/models/milkshake.rb`
**V**iew: `./app/views/milkshakes/index.html.erb`
**C**ontroller: `./app/controllers/milkshakes_controller.rb`
SASS: `./app/assets/stylesheets/milkshake_index.scss`

Notes
* directories are pluralised
* HTML.erb View files are necessary to render Ruby code
* Ruby code a HTML.erb file is written between `<% %>`, with the addition of `<%= %>` if the result is to be printed to the screen
* the SASS file is the home of the Rails app HTML and CSS
* CSS classes are added to the HTML.erb file

### 10.X Forms

```ruby
<main id="milkshake-index">
    <div class="banner-img"></div>
    <h1>All Milkshakes</h1>
```
```ruby
    <form action="<%= milkshakes_path %>" method="GET">
        <input type="search" name="search" />
    </form>
    <%= link_to("Create Milkshake", new_milkshake_path) %>
```
`link_to` is a Rails helper method that hyperlinks pages or the user like the `<a href>` anchor tag and hypertext reference attribute in HTML.

The text and URL after the `link_to` method are arguments `("Create Milkshake", new_milkshake_path)`.

Instead of hardcoding the route, the `_path` method can be used.

```ruby
    <section>
        <% @milkshakes.each do |milkshake| %>
            <%= link_to(milkshake) do %>
                <div>
                    <img src="<%= milkshake.image.url %>" />
                    <h2><%= milkshake.name %></h2>
                    <p><%= number_to_currency(milkshake.price / 100.0) %></p>
                </div>
            <% end %>
        <% end %>
    </section>
</main>
```

The `link_to` method takes an optional block `<% image_tag "Example" %>` where the content of the block becomes the clickable hyperlinked item (text, image, other HTML element).

### 10.X Partials

Partials - aka partial templates - allow us to create reuseable HTML code fragments to keep code DRY and save us from altering code in multiple files. The most prominent example of when to use partials is building a login form that you want to display on 10 different pages on your site.

Naming convention for partials is to begin with an underscore: `_form.html.erb`. Partials are usually saved under the corresponding Controller name inside of the Views directory `./app/views/directory_name`. Partials can also be saved in a directories called `./app/views/shared` to keep them from getting lost inside the View directories. Once created, partials communicate with the files where they are required via the render function. Partials are displayed on screen (rendered) using the render method inside a View, below:

```ruby
<%= render "form" %>
```

This will render the file `_form.html.erb`.

```ruby
<%= render "shared/form" %>
```

This will render the file `shared/_form.html.erb`.

Remember
* Partials use local variables **not** class @instance variables


### X Rails command table

| Command | Description |
| ------ | ------ |
| rake router | Displays all Rails routes |

### X Shortcuts

| Command | shorthand |
| ------ | ------ |
| rails server | rails s |
| rails generate | rails g |
| rails console | rails c |

### X Receiving `params` in the Controller

Whenever a user requests a page, the controller has access to an internal structure named `params`. Usually, such object has a JSON structure containing the following information:
```ruby
{"controller"=>"entities", "action"=>"index"}
```
However, depending on which action/route a user navigates to, the structure `params` can have many more fields. For example, a more complex structure can be found after submitting a form:
```ruby
{
    "_method"=>"patch", "authenticity_token"=>"3dAphlkbjQ4YBD2KKx9orzIHM4hRrOQWfpkiTQsdMRTgo8GoHuiSwr2YPbWH4msf76VpHDrvf0a37qurlaLP7A==",
    "toy"=>
    {
        "name"=>"Mario Kart Wii",
        "description"=>"Racing video game developed and published by Nintendo",
        "posted"=>"2018-10-26",
        "user"=>"Chelsea"
    },
    "commit"=>"Update",
    "controller"=>"toys",
    "action"=>"update",
    "id"=>"8"
}
```
When such structure is received by the controller, data sent from the browser is available to be used.
Common scenarios where such data is useful for a CRUD application are:
 * Being able to SHOW a specific resource (through the `:id` key)
 * Being able to DELETE a resource (through the `:id` key)

The `params` structure is particularly convenient for actions such as CREATE and UPDATE where the whole data received under the key `toy` can be used as a whole to insert or modify an entire record in the database. However, such convenience comes with risks: an attacker could potentially inject malicious code (from the browser or through postman) and corrupt the `params` variable.

To prevent such dangerous risk, Rails blocks any attempt to bulk-load (aka mass-assignment) data into the database and spits out the error message **`ForbiddenAttributesError`**.
That said, the mass-assignment is still possible, but an extra step is required by the developer: white-listing all the expected fields:

```ruby
whitelisted_params = params.require(:toy).permit( :name,
                                                  :description,
                                                  :posted,
                                                  :user
                                                )
toy = Toy.create(whitelisted_params)
```

### X Generating Form the "Rails way"

Implementing a form in HTML can be tedious.
Luckily, Rails offer a form HELPERS out of the box to generate a form, tailored on any model shape.
Historically, there have been 3 helpers
* `form_with`
* `form_for`
* `form_tag`
Unfortunately, the last two are already marked as "deprecated" in Rails 5 and completely "removed" in Rails 6.

In the example below, the form is generated on top of the "toy" model using the last available Rails-helper (form_with):

```ruby
<%= form_with(model: toy, local: true) do |form| %>

    <%= form.label :name %>
    <%= form.text_field :name %>
  
    <%= form.label :description %>
    <%= form.text_area :description %>
  
    <%= form.label :date_posted %>
    <%= form.date_select :date_posted %>

    <%= form.submit %>

<% end %>
```
