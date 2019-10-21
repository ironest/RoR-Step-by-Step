# RoR-Step-by-Step

### 1 Create a new project
```
rails new <proj-name> -d postgresql -T
```
Notes:
* Such commands creates a folder named `<proj-name>` inside of which the new Rails project will be created
* Inside the newly created project, a git repo will be initialized (meaning that `git init` is already issued)
* SQlite3 database is default database used when a new Ruby on Rails application is created. If a different database is required, it can be specified with an adhoc option `-d`. To instruct Rails to ignore any database, at all, option `-O` can be used.
* By providing the extra `-T`, the Rails project will skip the generation of any Test files.

### 2 Installing extra gems
```
bundle add <gem-name>
```
Adds the named gem to the Gemfile and run `bundle install`.
`bundle install` can be avoided by using the flag `--skip-install`.


### 3 Setting up the project
```
bundle install
```
After a project is created (or cloned from github) it’s necessary to check whether any gem is needed. This commands reads from the Gemfile.

### 4 Database Creation
```
rails db:create
```
When a Rails application is created for the first time, it will not have a database yet. To be create one, those commands are needed.
The command initializes/creates a database (named after the project). To be more accurate, it creates two versions of the same DB. one for development and one for testing purposes.

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
* If Entity represents a join table, the naming convention becomes EntitiesEntity. The priority of Entities over Entity is determined by the alphabetic order.
* Even though the Entity keyword is singular, the migration file produces a table having a plural form.

##### More examples of Model generation
```
rails g model Author name:string date_of_birth:date
rails g model Genre name:string
rails g model Book title:string
rails g model BooksGenre book:references genre:references
rails g model Movie title:string rating:integer
rails g model Actor name:string birthdate:date
rails g model ActorsMovie movie:references actor:references character:string
rails g model Image imageable:references{polymorphic} url:string
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
The command applies any pending migrations. Pending migration means setting up tables in the database (or alter tables or drop tables). When running the migration command, it will look in db/migrate/ for any Ruby files and execute them starting from the oldest.

##### Migration generation
Another way to generate migrations (other than from the Model generation) is to submit the commands such as:
```
# Table creations
rails g migration CreateAuthors name:string date_of_birth:date
rails g migration CreateBooks title:string author:string rating:integer

# Adding columns
rails g migration AddOverviewToBooks overview:string

# Removing columns
rails g migration RemovePublisherFromBooks column:datatype

# Replacing column (new migration)
rails g migration RemoveTypeFromIngredients type:integer
rails g migration AddKindToIngredients kind:integer
rails db:migrate

# Go to VScode, change models containing old column name
```

### 7 Model association
Even the simplest of the databases likely has models/tables with some osort of relationship with each other.
Examples are:
* one to one
* one to many
* many to many

Even though at the Database level those relationship are already set up through FK, it's necessarily to replicate the integrity logic in the application layer. To do so, few changes have to be applied in the ruby files under `./app/models/`

```ruby
belongs_to :movie
has_one :address
has_many :actors_movies
has_many :actors, through: :actors_movies
```

Notes
* `has_one` is followed by a singular symbol
* `has_many` is followed by a plural symbol
* a join table must have two `belongs_to` keywords

Validation
Sometimes it's required to put in place an extra validation (aka constraints) on one or more column of a model. For example, there might be a reason only accept a record where a numeric value falls between a range. Or when a new records has a mandatory field (mandatory as in cannot be left empty/nil). An exmaple on how to enforce validation rules, it's required to add similar instructions
```
validates :name, :price, presence: true
validates :price, numericality: {greater_than: 10}
```

### 8 Routes confiuguration
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

##### Option 2
Resource routing allows you to quickly declare all of the common routes for a given resourceful controller. Instead of declaring separate routes for your index, show, new, edit, create, update and destroy actions, a resourceful route declares them in a single line of code.

```ruby
resources :entities
```

### 9 Controller creation

Controller files have to be manually created under `./app/controller/`. The naming convention for such files is `entities_controller.rb` (everything lower-case and plural form).

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
In the MVC design patter, Views are the interface of an application. In Ruby on Rails, that refers to the actual web-page returned to a user.

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
`link_to` is a Rails helper method that hyperlinks pagesfor the user like the `<a href>` anchor tag and hypertext reference attribute in HTML.

The text and URL after the `link_to` method are arguements `("Create Milkshake", new_milkshake_path)`.

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

### X Rails command table

| Command | Description |
| ------ | ------ |
| rake router | Displays all Rails routes |

### X Shortcuts

| Command | shorthand |
| ------ | ------ |
| rails server | rails s |
| rails generate | rails g |

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

The `params` structure is particularly convenient for actions such as CREATE and UPDATE where the whole data received under the key `toy` can be used as a whole to insert or modify an entire record in the database. However, such conveniency comes with risks: an attacker could potentially inject malicious code (from the browser or through postman) and corrupt the `params` variable.

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

Implementing a form in HTML can be tedius.
Luckily, Rails offer a special HELPERS out of the box to generate a form, tailored on any model shape.
In the example below, the form is generated on top of the "toy" model:

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