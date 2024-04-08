# All the ways to render partials in Ruby on Rails

Partials in Ruby on Rails are one of those things I often take for granted. Being able to extract pieces of views and share across a monolith type of app is such a game changer for efficiency and productivity.

I find myself using Rails view partials in unique ways at times so as an exercise I wanted to document as many ways as I can recall to render partials. This list will likely grow as time allows so check back often!

Now lets dive in to study all the ways to render a partial with rails.

## Prerequsite steps to follow along

To save time I’ve leveraged my new project Rails UI to scaffold a new app. Here’s the steps to follow along if you wish to.

```bash
rails new flexible_partials
```

Add the Rails UI gem

```ruby
# Gemfile

# Free development preview
gem "railsui", github: "getrailsui/railsui"
```

Back in the terminal `cd` into `flexible_partials` and run

```bash
rails railsui:install
```

To boot the server run

```bash
bin/dev
```

#### Choose a template

Pick your choice of template and install it within the configuration wizard. It should be only a couple clicks.

#### Generate some data

I'll use a ficticious `Project` model to make something to help drive home how flexible partials can be.

```bash
rails g scaffold Project title:string description:text active:boolean
rails db:migrate
```

We need data next. Run `rails console` and add some projects.

```ruby
Project.create(title: "Website redesign", description: "A project dedicated to redesigning our marketing site", active: true)

Project.create(title: "iOS App Development", description: "A project led by iOS team to bring continuity between the web app and iPhone app.", active: true)

Project.create(title: "Help doc overhaul", description: "We need updated screenshots for our help docs. Let's address this here.", active: true)
```

#### Update root route

I've updated my app to accomodate. Be sure to comment out the default Rails UI root path. If you ever need to visit that page again you can do so from `/railsui/start`.

```ruby
# config/routes.rb

Rails.application.routes.draw do
  if Rails.env.development? || Rails.env.test?
    mount Railsui::Engine, at: "/railsui"
  end

  # Inherits from Railsui::PageController#index
  # To overide, add your own page#index view or change to a new root
  # Visit the start page for Rails UI any time at /railsui/start
  # root action: :index, controller: "railsui/page"

  devise_for :users
  get "up" => "rails/health#show", as: :rails_health_check

  resources :projects

  root "projects#index"
end
```

---

![Root path - Flexible partials](https://f001.backblazeb2.com/file/webcrunch/flexible-partials-root-path.png)

With all of that out of the way you should see something like this when heading to the root path.

Let's get on to the good stuff...

## Flexible partials

Partials are like a swiss-army knife for the Rails view layer. Hopefully showing you how versatile they are could inspire you to leverage them in new ways.

### The `render` Method:

This is the most common way to render a partial. You can use it in your views or templates like this

```ruby
<%= render partial: "shared/project" %>
```

You can simplify this to the following:

```ruby
<%= render "shared/project" %>
```

Variables can also be passed by appending “locals: {}” to the partial

```ruby
<%= render partial: "shared/project", locals: { name: "Andy" } %>
```

This again can be simplified to the following:

```ruby
<%= render "shared/project", name: "Andy" %>
```

There are a variety of ways to customize the behavior of `render`. You can render the default view for a Rails template, or a specific template, or a file, or inline code, or nothing at all. You can render text, JSON, or XML. You can specify the content type or HTTP status of the rendered response as well.

You can pass render in controllers and helpers all the same, just omit the ERB specific characters.

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit" # renders app/views/books/edit.html.erb
  end
end
```

You can see how conventions with Rails play an important role with partials. All things being equal, the following are additional ways to leverage `render` in a controller.

```ruby
render :edit
render action: :edit
render "edit"
render action: "edit"
render "books/edit"
render template: "books/edit"
```

Which one you use is really a matter of preference, but I’d argue the simpler the better.

**Pro tip:**

If you want to see what returns exactly you can use `render_to_string` instead of `render` . This returns an escaped string of whatever might be in your partial without sending any response back to the browser.

### The render `:template` option

```ruby
render template: "products/show"
```

Prior to Rails 2.2 if you wanted to be explicit with what partial returned you would need to pass the `template` option. Nowadays, Rails knows the view belongs to a different controller based on the embedded slash character in the path passed. So it can be minimized to the following:

```ruby
render "products/show"
```

### `render` with Collection

A collection is essentially another name for an array of objects in rails. Think of a set of books that are grouped and returned from your database as an example.

```ruby
<%= render partial: "books/book", collection: @books %>
```

What’s super cool is the code above can be greatly simplified to the following:

```ruby
<%= render @books %>
```

Assuming you have a basic CRUD model for your `Book` resource. Rails conventions already assume you have a collection of `@books` in your `index` action within a `BooksController` class. This then returns the array of objects (collection) to the `index` view and will render the collection automatically. The expectation of a `app/views/books/_book.html.erb` partial exists.

So to summarize, in your view you needn’t write the following:

```ruby
<% @books.each do | book| %>
  <%= render "book", book: book %>
<% end
```

Because such conventions exists Rails will know what to do when this is there instead:

```ruby
<%= render @books %>
```

Truly powerful stuff once you wrap your head around what’s going on.

Outside of the assumed logic there might be times where you do need to be more explicit and render a collection:

```ruby
<%= render partial: "products/price", collection: @prices %>
```

Notice the difference in naming conventions and less standard locations for views and partials.

## `render` with `:as` Option

If you're rendering a collection and want to use a different local variable name inside the partial, you can specify it with the `:as` option:

```ruby
<%= render partial: "shared/product", collection: @items, as: :product %>
```

Here you can customize the local variables instance name for when it’s utilized inside `_product.html.erb` . `product` becomes the variable within the partial.

### `render` with `:spacer_template`

You can use this option to specify a spacer template when rendering a collection with separators. Basically think of a different partial that renders after the first. It might be great for some sort of horizontal rule element (`hr)` or something related to UI you render after each partial instance.

```ruby
<%= render partial: "shared/item", collection: @items, spacer_template: "shared/spacer" %>
```

### `render :inline`

I don’t imagine you’d use this one much but just sharing as it’s a possibility.

```ruby
<%= render inline: '<%= render "shared/my_partial" %>' %>
```

## `render :file`

You can render a partial from a specific file path outside of the views directory if necessary. Again, this might be rare but it’s great to be able to have options.

```ruby
<%= render file: "lib/resources/_resume.html.erb" %>
```

## `render :action`

If you have a partial associated with a specific action in your controller, you can render it like this:

```ruby
<%= render action: "edit" %>
```

This would look for a specific action based on the corresponding controller in place. Wherever you render the partial would dictate which controller Rails resolves to.

## `render :string`

If you want to render a partial from a string of HTML or text, you can use the `:string` option:

```ruby
<%= render string: "<p>This is a partial rendered from a string.</p>" %>
```

# Bonus points with strict locals

Sometimes, a partial expects a local variable to be passed, and we forget to pass such a local. This results in an error `undefined local variable or method`. To compensate for this issue, you can define what are known as [strict locals](https://github.com/rails/rails/pull/45602).

These allow a partial to define which locals they accept. You do this with an ERB style comment at the top of the file.

```erb
<%# issues/_card.html.erb %>
<%# locals: (title: "Default title", comment_count: 0) %>
<h2><%= title %></h2>
<span class="comment-count"><%= comment_count %></span>
```

For comparison sake you would have had to do something like this to avoid errors before:

```erb
<%# issues/_card.html.erb %>
<% title = local_assigns[:title] || "Default title" %>
<% comment_count = local_assigns[:comment_count] || 0 %>
<h2><%= title %></h2>
<span class="comment-count"><%= comment_count %></span>
```

This still works but it’s pretty gnarly code isn’t it?

### Roll your own slots

While not 100% in parity with something like ViewComponent, partials can behave like slots if you render them as layouts. I use this a lot to reduce code duplication and dry up my views. Rails UI in particular uses a partial layout to bring a consistent authenetication experience to a given set of views from the Devise gem.

In its simplest form here's what I mean by rendering a layout

```erb
<%= render layout "some/partial" do %>
  <h1>Hello world</h1>
  <p>Some code can go here</p>
<% end %>
```

Here's how I've put this to real use with Rails UI. Below is the authentication UI for the Hound theme.

```erb
<!-- Hound theme: app/views/devise/_auth_layout.html.erb -->
<div
  class="sm:h-[calc(100vh_-_52px)] pt-10 sm:pt-0 flex flex-col items-center justify-center bg-cover bg-center px-4"
  style="background-image: url('<%= asset_url('fusion.png')%>')">
  <div class="sm:flex-1 flex flex-col justify-center sm:w-[428px] w-full">
    <div>
      <div class="flex justify-center">
        <%= link_to root_path do %> <%= image_tag "logo.svg", alt: "Hound logo",
        class: "w-10 h-auto" %> <% end %>
      </div>

      <div class="mt-6"><%= yield :masthead %></div>

      <div class="bg-white shadow-sm rounded-lg p-8 border border-slate-300/60">
        <%= yield %>

        <!-- Add additional provider svg icons in app/assets/images/omniauth as necessary. These should ideally be full color versions of each provider's logo for max consistency. File name conventions should be lowercase without hyphens or spaces (i.e., google, linkedin, twitter, facebook)
        Default: Google, LinkedIn, Twitter, Facebook. -->

        <% if devise_mapping.omniauthable? && %w{ registrations sessions
        }.include?(controller_name) %>

        <hr class="my-6" />

        <% resource_class.omniauth_providers.each do |provider| %>
        <div class="my-2">
          <%= button_to omniauth_authorize_path(resource_name, provider), class:
          "btn btn-white w-full", data: { turbo: false } do %>
            <%= inline_svg "omniauth/#{provider.gsub(/\s+/, '').downcase}.svg", class: "mr-2 w-5 h-5" %>
          <span>"Sign in with <%= OmniAuth::Utils.camelize(provider) %></span>
          <% end %>
        </div>
        <% end %> <% end %>
      </div>

      <div class="mt-4"><%= render "devise/shared/links" %></div>
    </div>
  </div>
</div>
```

I extract the core UI into this layout which is then rendered like this:

```erb
<!-- render auth_layout partial with block (app/views/devise/_auth_layout.html.erb) -->
<%= render "auth_layout" do %>
  <!-- Devise view code -->
<% end %>
```

Assuming your views follow a consistent design this cleans things up a lot and allows you to slot in any other HTML/ERB you want. As a bonus, you can add additional `yield` and `content_for` statements to pass more dynamic code as necessary.

Here's an example of the Hound template's sign in view for Devise. There's no down _some_ complexity here but I think the ability to tweak and customize this feels very product as opposed to extracting too much away.

```erb
<!-- app/views/devise/sessions/new.html.erb -->
<% content_for :masthead do %>
  <div class="text-center">
    <h1 class="text-3xl font-extrabold text-slate-900 dark:text-slate-100 tracking-tight my-3">Sign in to your account</h1>
    <p class="mb-6 text-slate-700 dark:text-slate-200">Or <%= link_to "sign up", new_registration_path(resource_name), class: "btn-link" %> for an account</p>
  </div>
  <%= render "shared/error_messages", resource: resource %>
<% end %>

<%= render "auth_layout" do %>
  <%= form_for(resource, as: resource_name, url: session_path(resource_name), data: { turbo: false }) do |f| %>

    <div class="form-group">
      <%= f.label :email, class: "form-label" %>
      <div class="relative">
        <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-input focus:pl-10 peer transition", pattern: "[^@\s]+@[^@\s]+\.[^@\s]+", title: "Invalid email address" %>
        <%= icon "envelope", classes: "w-5 h-5 absolute translate-x-0 top-3 text-slate-300 peer-focus:text-indigo-500/80 opacity-0 transition transform peer-focus:opacity-100 peer-focus:translate-x-3 dark:peer-focus:text-indigo-400" %>
      </div>
    </div>

    <div class="form-group">
      <%= f.label :password, class: "form-label" %>
      <div class="relative">
        <%= f.password_field :password, autocomplete: "current-password", class: "form-input focus:pl-10 peer transition" %>
        <%= icon "lock-closed", classes: "w-5 h-5 absolute translate-x-0 top-3 text-slate-300 peer-focus:text-indigo-500/80 opacity-0 transition transform peer-focus:opacity-100 peer-focus:translate-x-3 dark:peer-focus:text-indigo-400" %>
      </div>
    </div>

    <div class="flex flex-wrap justify-between items-center form-group">
      <% if devise_mapping.rememberable? %>
        <div class="flex items-center">
          <%= f.check_box :remember_me, class: "form-input-checkbox" %>
          <%= f.label :remember_me, class: "form-check-label ml-2" %>
      </div>
      <% end %>
      <% if devise_mapping.recoverable? && controller_name != 'passwords' && controller_name != 'registrations' %>
        <%= link_to "Forgot your password?", new_password_path(resource_name), class: "btn-link text-sm" %>
      <% end %>
    </div>

    <%= f.submit "Sign in", class: "btn btn-primary hover:cursor-pointer w-full" %>
  <% end %>
<% end %>
```


### Wrapping up

While I've covered a lot of ways to utilize partials there are still more you can do with them. For instance, rendering a partial from a helper method is pretty handy if you need something simple like a divider or some basic repeatable code. The swiss-army knife capabilties makes me a big fan of partials. I realize there are always pros and cons and perhaps a con is slower performance for some instances but I think it's an equal trade off if the difference is neglegabile.
