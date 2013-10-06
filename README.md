# Ransack-mongoid

## Notice: Under development!

Ransack-mongoid is an attempt to port most of the core [ransack](https://github.com/ernie/ransack) functionality for use with Mongoid (and possibly other Document based data store mappers)

This project with started in response to

[ransack mongoid issue 120](https://github.com/ernie/ransack/issues/120)

## Getting started

Someday when it works...

In your Gemfile:

```ruby
gem "ransack-mongoid"  # Last officially released gem
```


## Usage

Ransack can be used in one of two modes, simple or advanced.

See the [Ransack README](https://github.com/ernie/ransack) for more info.

In your controller:

```ruby
def index
  @q = Person.search(params[:q])
  @people = @q.result(:distinct => true)
end
```

In your view:

```erb
<%= search_form_for @q do |f| %>
  <%= f.label :name_cont %>
  <%= f.text_field :name_cont %>
  <%= f.label :articles_title_start %>
  <%= f.text_field :articles_title_start %>
  <%= f.submit %>
<% end %>
```

`cont` (contains) and `start` (starts with) are just two of the available search predicates.
See Constants for a full list.

### Advanced Mode

See [Ransack README](https://github.com/ernie/ransack)

### Search solution ideas for Mongoid relations

Here some thoughts how it could be achieved (without knowing much about the details of the Mongoid internals with regards to these matters...)

```ruby
class Post
  include Mongoid::Document

  field :name, type: String

  embeds_many :comments 
  belongs_to  :authors,   class_name: 'User'
  has_many    :reviewers, class_name: 'User'

  enable_ransack # includes Ransack macro module

  # Ransack macro module makes the macro #search_field available
  # creates embedded doc called 'search_reviewers' 
  # of class SearchReview (unless class already exists)
  # with fields name and rating
  # this embedded doc will be auto-updated on after-save
  search_field :reviewers do
    index :name, :rating
  end
end
```

Uses the `after :save` hook to update an embedded index of relational attributes for use with search form. Determines which of the updated attributes are relations, and then for the ones of interest to the search (as defined by use of `#search_field` on the model), it will create embedded docs for those relational fields with a subsset of searchable attributes. 

Chained relations?:

```ruby
search_field :reviewers do
  index :name, :rating

  search_field :boss, for: %w{name title}    
end  
```

In the above example, the for option is a shorthand for a single index of attributes.
The `#index` within the block could be used for more advanced scenarios, perhaps specifying additional options, another block or ... ?

Here some pseudo code to illustrate:

```ruby
after :save do
  ransack_attributes
end

def ransack_attributes
  attributes.each do |att|
    if search_attributes.include?(att) && relationship?(att)
      update_search_field att
    end
  end
end
```

How do we then ensure that it works in the other direction? We need a way for the `belongs_to` (inverse relationship) to notify the owner of the relationship of a change, in order to update the index. This should be possible!

## Contributions

To support the project:

* Use Ransack in your apps, and let us know if you encounter anything that's broken or missing.
  A failing spec is awesome. A pull request is even better!
* Spread the word on Twitter, Facebook, and elsewhere if Ransack's been useful to you. The more
  people who are using the project, the quicker we can find and fix bugs!

## Copyright

Copyright &copy; 2011 [Ernie Miller](http://twitter.com/erniemiller)
