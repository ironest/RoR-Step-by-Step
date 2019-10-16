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

### 4 Database
```
rails db:create
rails db:migrate
```

When a Rails application is created for the first time, it will not have a database yet. To be create one, those commands are needed.

The first command initializes/creates a database named after the project. To be more accurate, it creates two versions of the same DB. one for development and one for testing purposes.

The second command populates any migration.