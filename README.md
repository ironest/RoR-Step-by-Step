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

### 4 Model generation
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

### 5 Database Migration
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
```

### 6 Model association
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

### 7 Routes confiuguration
Routes are configured inside `./config/routes.rb`

There are two possible options.
##### Option 1
```ruby
get "/entities", to: "entities#index", as: "entities"
get "/entities/new", to: "entities#new", as: "new_entity"
post "/entities", to: "entities#create"
get "/entities/:id", to: "entities#show", as: "entity"
get "/entities/:id/edit", to: "entities#edit", as: "edit_entity"
put "/entities/:id", to: "entities#update"
patch "/entities/:id", to: "entities#update"
delete "/entities/:id", to: "entities#destroy"
```

##### Option 2
Resource routing allows you to quickly declare all of the common routes for a given resourceful controller. Instead of declaring separate routes for your index, show, new, edit, create, update and destroy actions, a resourceful route declares them in a single line of code.

```ruby
resources :entities
```

### 8 Controller creation