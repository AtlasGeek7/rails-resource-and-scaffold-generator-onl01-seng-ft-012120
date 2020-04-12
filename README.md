# Resource and Scaffold Generator

## Objectiveshttps://learn.co/tracks/online-software-engineering-structured/rails/crud-with-rails/resource-generator-routes#

1. Use the resource generator
2. Identify the components generated by the resource generator
3. Use the scaffold generator
4. Identify the components and code generated by the scaffold generator
5. Describe the code (except for form partials) related to a scaffold
6. Use the resources route helper to generate the RESTful routes

## Intro

In a [previous lesson](https://github.com/learn-co-curriculum/rails-generators-readme) we reviewed each of the popular generators in Rails. I purposefully left one out: the Rails `scaffold` generator. The reason for this is mainly due to the fact that it's not considered a good practice to use scaffolds in a production application. With that being said, I do think it's important to study scaffolds since they can be a great reference for how we can build CRUD functionality into our apps.

First let's discuss why it's not a great idea to use scaffolds in real world development. Let's start with a case study to see what a scaffold actually creates. Run this command in the terminal:

```bash
rails g scaffold Article title:string body:text
```

Let's review the console log to see what this creates for us:

```shell
invoke  active_record
create    db/migrate/20151128001950_create_articles.rb
create    app/models/article.rb
invoke    rspec
create      spec/models/article_spec.rb
invoke      factory_girl
create        spec/factories/articles.rb
invoke  resource_route
 route    resources :articles
invoke  scaffold_controller
create    app/controllers/articles_controller.rb
invoke    erb
create      app/views/articles
create      app/views/articles/index.html.erb
create      app/views/articles/edit.html.erb
create      app/views/articles/show.html.erb
create      app/views/articles/new.html.erb
create      app/views/articles/_form.html.erb
invoke    rspec
create      spec/controllers/articles_controller_spec.rb
create      spec/views/articles/edit.html.erb_spec.rb
create      spec/views/articles/index.html.erb_spec.rb
create      spec/views/articles/new.html.erb_spec.rb
create      spec/views/articles/show.html.erb_spec.rb
create      spec/routing/articles_routing_spec.rb
invoke      rspec
create        spec/requests/articles_spec.rb
invoke    helper
create      app/helpers/articles_helper.rb
invoke      rspec
create        spec/helpers/articles_helper_spec.rb
invoke    jbuilder
create      app/views/articles/index.json.jbuilder
create      app/views/articles/show.json.jbuilder
invoke  assets
invoke    coffee
create      app/assets/javascripts/articles.coffee
invoke    scss
create      app/assets/stylesheets/articles.scss
invoke  scss
create    app/assets/stylesheets/scaffolds.scss
```

If you remember back to the [lesson on generators](https://github.com/learn-co-curriculum/rails-generators-readme) you will remember that the other generators ( `migrations`, `controllers`, and `resources`) created a structure and backend functionality for our code. However, scaffolds actually go beyond the other generators and create both the front- and back-end code needed for CRUD features. If you start up the Rails server and navigate to `localhost:3000/articles` you will see the screens below:

![Scaffold Screen](https://s3.amazonaws.com/flatiron-bucket/readme-lessons/scaffolds.png)

Showing scaffolds to someone new to Rails is a great way to amaze them. Within less than a minute the scaffold system built an entire CRUD-based feature, and we didn't write a single line of code!

What did the scaffold build for us? If we look through the files that got printed out in the console, we see:

* A migration file

* A Model file

* A controller

* View templates for each of the controller actions that render a view

* The full set of RESTful routes

* And every other component needed for a functional CRUD environment

One thing I really like about using the scaffold generator to teach Rails is how they set up the controller. Below are the contents of the `articles_controller.rb` file:

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]

  # GET /articles
  # GET /articles.json
  def index
    @articles = Article.all
  end

  # GET /articles/1
  # GET /articles/1.json
  def show
  end

  # GET /articles/new
  def new
    @article = Article.new
  end

  # GET /articles/1/edit
  def edit
  end

  # POST /articles
  # POST /articles.json
  def create
    @article = Article.new(article_params)

    respond_to do |format|
      if @article.save
        format.html { redirect_to @article, notice: 'Article was successfully created.' }
        format.json { render :show, status: :created, location: @article }
      else
        format.html { render :new }
        format.json { render json: @article.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /articles/1
  # PATCH/PUT /articles/1.json
  def update
    respond_to do |format|
      if @article.update(article_params)
        format.html { redirect_to @article, notice: 'Article was successfully updated.' }
        format.json { render :show, status: :ok, location: @article }
      else
        format.html { render :edit }
        format.json { render json: @article.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /articles/1
  # DELETE /articles/1.json
  def destroy
    @article.destroy
    respond_to do |format|
      format.html { redirect_to articles_url, notice: 'Article was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_article
      @article = Article.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def article_params
      params.require(:article).permit(:title, :body)
    end
end
```

If you look through the code you'll see a few familiar methods, such as: `index`, `new`, `edit`, `update`, and `show`. If you remember prior lessons you may remember that we had some duplicate code, for example if we had: `show`, `edit`, and `update` actions in the controller we had three different calls such as:

```ruby
@article = Article.find(params[:id])
```

This was necessary so that we could grab the `/:id` parameter from the URL string, but, as you may have noticed, the scaffold implemented an elegant solution to remove the duplicate code:

```ruby
before_action :set_article, only: [:show, :edit, :update, :destroy]
```

The `show`, `edit`, `update`, and `destroy` actions will all have the `set_article` method called before any other code in the action is run. If you want to inspect the code for the `set_article` method, it's declared at the bottom of the controller:

```ruby
def set_article
  @article = Article.find(params[:id])
end
```

As you can see, the method returns the `@article` instance variable that each of the controller actions will automatically have because of the `before_action`. Pretty cool, right?

Another way that scaffolds show how to DRY up controller code can be found in the other `private` method:

```ruby
def article_params
  params.require(:article).permit(:title, :body)
end
```

So, even though scaffolds are great for learning how CRUD works in Rails, it's still considered a bad practice to use them in production applications. The main reason why scaffolds are discouraged in production is because they create so much code and so many files that they can be hard to manage.

From my personal development experience, I've found that my best applications were built by following TDD principles where I created features one element at a time, whereas scaffolds build dozens of processes instantly. Truth be told, when I use a scaffold it usually takes me more time going through each file that was created to remove code/files that I'm not going to be using than it would have taken to build the feature from scratch!


## Scaffold vs Resource Generators

Since we've already discussed that it's discouraged to utilize scaffolds in production applications, it's fair to ask what a good alternative is. That's where the `resource` generator comes into play.

We've discussed [in detail](https://github.com/learn-co-curriculum/rails-generators-readme) what the `resource` generator does, so what makes it a better solution than scaffolds?

The scaffold generator tends to be very 'opinionated' with the code that it builds. By opinionated I mean that it builds out the system in a very specific manner, which is rarely the way that you would want to build your application.

With modern development practices, a large number of Rails apps are leveraging client side MVC setups such as Backbone or AngularJS. These frameworks render the view templates for a Rails application pointless, so if you rely on using scaffolds you're going to have to be removing quite a bit of code after each `generate` command.

If you compare this with the `resource` generator, the code from the `resource` generator will build out the base setup required for the new feature, but it will let you control the implementation.

Let's pretend that you're integrating the ReactJS framework into your Rails application. If you use a scaffold you will instantly have to go through the code and remove a large percentage of the code along with around 50% of the files themselves. Whereas if you run the `resource` generator, it simply creates the: migrations, model, routes, controller, and asset pipeline files. This means that you will be able to instantly start implementing the ReactJS components instead of having to be concerned with which elements need to be deleted.

A road trip is a helpful analogy:

* A `scaffold` generator is like driving double the speed limit and driving past your exit by 50 miles, forcing you to backtrack all the way back to get on the right road.

* A `resource` generator is like going the speed limit and taking the right exit; the `scaffold` may have flown by you initially, but with the `resource` option you'll end up getting to the final destination in one piece.


## Resources

* [Rails Generator Documentation](http://api.rubyonrails.org/classes/Rails/Generators.html)

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/rails-resource-and-scaffold-generator'>Resource Generator/ Routes</a> on Learn.co and start learning to code for free.</p>
