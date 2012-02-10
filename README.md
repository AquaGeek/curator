# librarian [![Build Status](https://secure.travis-ci.org/pgr0ss/librarian.png)](http://travis-ci.org/pgr0ss/librarian)

Librarian is a model and repository framework for Ruby. It's an alternative to ActiveRecord-like libraries where models are tightly coupled to persistence. Librarian allows you to write domain object that are persistence free, and then write repositories that persist these objects. These ideas are largely taken from the [Repository](http://domaindrivendesign.org/node/123) section of [Domain Driven Design](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215).

Currently, librarian supports Riak for persistence. If you are interested in enhancing librarian to support other data stores, please let me know.

## Usage

Domain objects should include the `Librarian::Model` module:

```ruby
class Note
  include Librarian::Model
  attr_accessor :id, :title, :description, :user_id
end
```

These models can be intiatiated with hashes and used just like regular ruby objects:

```ruby
note = Note.new(:title => "My Note", :description => "My description")
puts note.description
```

Repositories should include the `Librarian::Repository` module:

```ruby
class NoteRepository
  include Librarian::Repository
  indexed_fields :user_id
end
```

Repositories have `save`, `find_by_id`, and `find_by` methods for indexed fields:

```ruby
note = Note.new(:user_id => "my_user")
NoteRepository.save(note)

NoteRepository.find_by_id(note.id)
NoteRepository.find_by_user_id("my_user")
```

### Rails

See [librarian_rails_example](librarian_rails_example) for an example application using librarian.

If you use librarian within Rails, all you need is to add librarian to your Gemfile and create a config/riak.yml with contents like:

```yaml
development:
  :http_port: 8098
  :host: localhost
test:
  :http_port: 8098
  :host: localhost
```

We recommend putting your models in app/models and your repositories in app/repositories. If you do this, don't forget to add app/repositories to the list of autoload paths:

```ruby
# config/application.rb

config.autoload_paths += %W(#{config.root}/app/repositories)
```

You can also use Rails form builder with librarian models:

```erb
<%= form_for @note, :url => { :action => "create" } do |f| %>
  <dl>
    <dt><%= f.label :title %></dt>
    <dd><%= f.text_field :title %></dd>
    <dt><%= f.label :description %></dt>
    <dd><%= f.text_area :description, :size => "60x12" %></dd>
  </dl>
  <%= f.submit "Create" %>
```

### Without Rails

If you are not using Rails, you can configure librarian manually:

```ruby
Librarian.environment = "development"
Librarian.riak_config_file = File.dirname(__FILE__) + "/config/riak.yml"
```

## Under the hood

Librarian stores objects in riak using the id as the key. The value is a json representation of the instance_values of the object. Your repository can implement serialize/deserialize to get different behavior.

The bucket name in riak is librarian:<environment>:collection. The collection is derived from the name of the Repository class, and it can be overriden. For example, if you implement a NoteRepository, the riak bucket will be librarian:development:notes in development mode, and librarian:production:notes in production mode.