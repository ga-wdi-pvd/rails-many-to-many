# Many-to-Many Relationships (using has_many :through) in Rails

## Learning Objectives

* Investigate the difference between a one-to-many and a many-to-many relationship
* Distinguish what real world relationships can be formulated by many-to-many relationships
* Analyze the role of a join table in a many-to-many relationship
* Construct a Model to represent a join table
* Use `has_many through` to connect two models via a join model in Rails
* Use a many-to-many relationship to implement a feature in a Rails app

## Framing

Let's think back to what we have learned so far in this unit. We now have the ability to model real world entities and their relationships, and have built web apps that have persisted data about these models.

---

Up to this point, we have focused on domains that only have two models, i.e. `Artists`, and `Songs`, which in turn have a strict one-to-many relationships. At it's core, we expressed these relationship with ActiveRecord methods, and linked the tables via a foreign key on the child table.

This type of relationship is probably the most common, but today we will be looking at another widely used and very useful relationship that will help build out more features and different types of features to our Rails apps.

## Many-to-Many Relationships (10 minutes)

Put simply, Many-to-Many relationships arise when one or more records in a table, has a relationship with one or more records in another table.

Many-to-many relationships are fairly common in apps. Examples might include:

* `Posts` can be sorted into multiple `Categories`, `Categories` contain many `Posts`.
* `Theaters` can show many `Movies`, and `Movies` may appear in many `Theaters`.
* `Playlists` contain many `Songs`, `Songs` can be on multiple `Playlists`.
* `Doctors` can have many `Patients`, a `Patient` can have more than one `Doctor`

Unlike one-to-many relationships, we can't just add a foreign key to one of the
two tables (the `belongs_to` table) to store these associations. We'd run into a
problem where the column would need to store multiple ids, rather than just the
one id in a one-to-many relationship.

Instead, we must create a new table, a ***join table*** to store these associations.

### Join Tables

A join table is a separate intermediate table in our database whose job is to store information
about the relationship between our two models of the many-to-many. For each
many-to-many relationship, we'll need one join table.

![Join Table ERD](/Screen Shot 2016-12-19 at 10.21.37 AM.png)

> Why are they called "join tables"? On a database level, join tables are created using SQL methods like `INNER JOIN` and `OUTER JOIN`. Learn more about them [here](http://www.sql-join.com/).

Each join table should have, at minimum, **two foreign_key columns**. Each foreign key will represent one of the tables it's joining. In the example of `Doctors` and `Patients`, we would create a **new** join table that has a `doctor_id` column and a `patient_id` column.

We can also add columns as needed to store additional information about the relationship. For example, we may choose to add an `date_of_visit` column which stores a `datetime` value representing when the appointment is, and could be different for each doctor + patient visit.

## Join Models & Tables

In order to do many-to-many relationships in Rails, convention says to create a new model to represent our join table. The name can technically be anything we want, but really the model name should be as descriptive as possible, and indicate that it represents an *association*.

### You Do: Naming Join Tables (10 minutes)

In pairs, spend **5 minutes** answering the following questions for the below pairs of models...  
  1. Should the relationship between these two models be represented using a many-to-many relationship?
  2. What would be a descriptive name for their resulting join table?
  3. What would be a useful additional column to include in the join table (e.g., `order`)?

Models  
  1. Authors and Books
  2. Students and Courses  
  3. Users and Groups  
  4. Blog Posts and Categories  
  5. Reddit Posts and Votes  

### We Do: Generating the Model / Migration (10 minutes)

To get started:

1. Clone down [this repo](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-starter)
2. Checkout to the `favorites-starter` branch
2. Run `$ bundle install`
3. Run `$ rails db:drop db:create db:migrate db:seed` 

In order to see how to implement an example of a common many-to-many relationship in Rails, I'm going to build out our tunr app, and you will repeat after me.

<details>
<summary>**Q**. What should the four models in our application be?</summary>

Let's call them: `User`, `Artist`, and `Song`, And now `Favorite`.

</details>

---

> A `User` model and user authentication functionality has already been provided for you. Because of this, you may see some code in here -- particularly in `models/user.rb` and `routes.rb` that was added by the gem Devise

> **Note**: Make sure to work off the `favorites-starter` branch.

For our domain's purposes, let's create a new model `Favorite` to represent the many-to-many relationship between our other two models: `User` and `Song`.

```bash
$ rails g model Favorite song_id:integer user_id:integer
```

> **Note**: When we generated the Favorite model, we created our columns here and not in a migration.

We generate the model just like any other. If we specify the attributes (i.e.,
columns on the command line) Rails will automatically generate the correct
migration for us.

<details>
<summary>**Q**. What files were create running the generate model command?</summary>

```bash
      create    db/migrate/20161221191859_create_favorites.rb
      create    app/models/application_record.rb
      create    app/models/favorite.rb
```

</details>

Let's take a look at the migration file that was created for us...

```rb
# db/migrate/*****_create_favorites.rb

class CreateFavorites < ActiveRecord::Migration
  def change
    create_table :attendances do |t|
      t.references :song_id, index: true, foreign_key: true
      t.references :user_id, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end
```

> **What is `t.references`?** It does the same thing as writing out `belongs_to :model`.

This will generate an Favorite table with `song_id` and `user_id` columns. Take a look at it using `psql` in the Terminal.


### Adding the ActiveRecord Relationships (10 minutes)

Once we create our join model, we need to update our other models to indicate the associations between them. Let's visualize these associations with an ERD.

**Board**: Diagram Favorite Tracker ERD

For example, in our Users/Events example, we should have this...

```ruby
# models/favorite.rb
class Favorite < ActiveRecord::Base
  belongs_to :song
  belongs_to :user
end

# models/song.rb
class Song < ActiveRecord::Base
  has_many :favorites
  has_many :users, through: :favorites
end

# models/user.rb
class User < ActiveRecord::Base
  has_many :favorites
  has_many :events, through: :favorites
end
```

We're essentially defining `Favorite` as an intermediary model/table between `Song` and `User`. An event has many users through `Favorite` and vice versa.

## Break (10 minutes)

### We Do: Add Web Interface to Tunr (15 minutes)

So we've been able to generate associations between our models via the rails console. But what about our end users? How would somebody go about creating/removing a favorite on Tunr?

<details>
<summary>At a high level, what type of code do we need to add to support our new favoriting feature for Tunr?</summary>

We need to add functionality by **modifying our controller, view and routes**.

</details>

---

Before we add anything new, let's do a quick recap of the code we are starting from...

#### Controller

Let's take a look at `songs_controller.rb`...
* What do we currently have in here?
* Can we use any of these actions to handle adding/removing songs? Or do we need to add something new?

```rb
class SongsController < ApplicationController
  # index
  def index
    @songs = Song.all
  end

  # new
  def new
    @artist = Artist.find(params[:artist_id])
    @song = @artist.songs.new
  end

  # create
  def create
    @artist = Artist.find(params[:artist_id])
    @song = @artist.songs.create(song_params)
    redirect_to artist_song_path(@artist, @song)
  end

  #show
  def show
    @song = Song.find(params[:id])
  end

  # edit
  def edit
    @artist = Artist.find(params[:artist_id])
    @song = Song.find(params[:id])
  end

  # update
  def update
    @song = Song.find(params[:id])
    @song.update(song_params)
    redirect_to artist_song_path(@song.artist, @song)
  end

  # destroy
  def destroy
    @song = Song.find(params[:id])
    @song.destroy
    redirect_to songs_path
  end

  private
  def song_params
    params.require(:song).permit(:title, :album, :preview_url, :artist_id)
  end
end
```

We have CRUD functionality for the songs themselves, but that's about it.
* We need to add some actions to our controller that handle this additional functionality. You'll do that for Tunr in the next exercise.
* These will not correspond to RESTful routes.

#### Routes

There's more to this than just updating the `Songs` controller, we also need to make sure that our application has routes to support "favoriting" and "unfavoriting". For the sake of convenience, we have already defined the desired routes for you...

```rb
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users
  root to: 'artists#index'

  resources :artists do
    resources :songs, except: [:index, :show]
  end

  resources :songs, only: [:index, :show] do
  # This creates two custom routes for songs that correspond with controller actions of the same name.
    member do
      post 'add_favorite'
      delete 'remove_favorite'
    end
  end
end
```
> **Hint**: check out the output of `rails routes` to see what those lines generated!

#### View

Great, now that we know we have the necessary routes defined, we need a way for the user to actually interact with our Web app so they can favorite a song.

We've gone ahead a provided some starter code in `app/views/artists/show.html.erb`, so let's look at the interface to how the user will favorite a song...

```erb
<h3>Songs <%= link_to "(+)", new_artist_song_path(@artist) %></h3>
<ul>
  <% @artist.songs.each do |song| %>
    <li>
      <%= link_to "#{song.title} (#{song.album})", song_path(song) %>

      # If this song has already been favorited, set the link to remove favorite.
      <% if song.users.include? current_user %>
        <%= link_to "&hearts;".html_safe, remove_favorite_song_path(song), method: :delete, class: "fav" %>
      # If the song has not been favorited, set the link to add favorite.
      <% else %>
        <%= link_to "&hearts;".html_safe, add_favorite_song_path(song), method: :post, class: "no-fav" %>
      <% end %>

    </li>
  <% end %>
</ul>
```

## Break (10 minutes)

### You Do: Update Songs Controller (20 minutes)

Take **15 minutes** to create the `add_favorite` and `remove_favorite` actions in the **songs controller**. Look at the `artists/show.html.erb` view to see how we route to these actions.

Below are some line-by-line instructions on how to implement `add_favorite` and `remove_favorite`. We encourage you not to look at the solution unless you are stuck!  

#### Instructions

Start out by logging into the application using the "Sign Up" feature. It should be visible in the top-right corner on the home page. Once you've done that, tackle the controller actions.

`add_favorite` should...  
  1. Save the song which you will be favoriting in an instance variable.  
  2. Create a new Favorite instance that...  
    a. Belongs to the song.  
    b. Belongs to the user who is creating the favorite.  
  3. Redirect to the show page for the artist once the song is added.

`remove_favorite` should...  
  1. Save the song you will be un-favoriting in an instance variable.  
  2. Delete the Favorite instance that references the song that is being un-favorited.  
  3. Redirect to the show page for the artist once the song is added.

#### How Do We Get the Logged-In User?

Because we are using Devise to handle user authentication, it gives us access to a `current_user` method that, when called, returns the user who is currently logged in. At a high level, think of it as running something like `User.find_by(logged_in: true)`.  

This means that in your controller you can write code like `Favorite.create(user: current_user)`.

### Testing Our Associations (10 minutes)

Lets play around with our app and check out the data in the `rails c` (console).

#### If You Need the Solution...

[...you can take a peek at it here.](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-solution)

## Closing Q&A (10 minutes)

## Bonus

###  Additional Exercise: Many-to-Many Scribble

If there's time left, spend the remainder of class working on Scribble. If you have completed the required steps, try implementing a many-to-many relationship between `Posts` and `Categories` using a `Tags` join table. This will require creating some new classes.

[More information is available on the Scribble repo.](https://github.com/ga-wdi-exercises/scribble/blob/master/readme.md#bonus)

## Screencasts

* [Screencast on Youtube](https://www.youtube.com/watch?v=JxW8lJzLhxI)
* [Andy 1](https://youtu.be/PXNrk6m4WRg)
* [Andy 2](https://youtu.be/Gr3GV8dUkDE)

## References

* [Rails Guides - Has Many Through](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
* [Tunr Favorites Starter](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-starter)
* [Tunr Favorites Solution](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-solution)
