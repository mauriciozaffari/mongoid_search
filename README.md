Mongoid Search
============

Mongoid Search is a simple full text search implementation for Mongoid ORM.

Installation
--------

In your Gemfile:

    gem 'mongoid_search'

Then:

    bundle install

Examples
--------

    class Product
      include Mongoid::Document
      include Mongoid::Search
      field :brand
      field :name

      references_many :tags
      refereced_in    :category

      search_in :brand, :name, :tags => :name, :category => :name
    end

    class Tag
      include Mongoid::Document
      field :name

      referenced_in :product
    end

    class Category
      include Mongoid::Document
      field :name

      references_many :products
    end

Now when you save a product, you get a _keywords field automatically:

    p = Product.new :brand => "Apple", :name => "iPhone"
    p.tags << Tag.new(:name => "Amazing")
    p.tags << Tag.new(:name => "Awesome")
    p.tags << Tag.new(:name => "Superb")
    p.save
    => true
    p._keywords

Now you can run search, which will look in the _keywords field and return all matching results:

    Product.search("apple iphone").size
    => 1

Note that the search is case insensitive, and accept partial searching too:

    Product.search("ipho").size
    => 1
    
Assuming you have a category with multiple products you can now use the following
code to search for 'iphone' in products cheaper than $499

    @category.products.where(:price.lt => 499).csearch('iphone').asc(:price)

In this case we have to use csearch, an alias for search, because since v2.0.0
Mongoid defines it's own Criteria.search method.


Options
-------

match:
  _:any_ - match any occurrence
  _:all_ - match all ocurrences
  Default is _:any_.

    search_in :brand, :name, { :tags => :name }, { :match => :any }

    Product.search("apple motorola").size
    => 1

    search_in :brand, :name, { :tags => :name }, { :match => :all }

    Product.search("apple motorola").size
    => 0

allow_empty_search:
  _true_ - match any occurrence
  _false_ - match all ocurrences
  Default is _false_.

    search_in :brand, :name, { :tags => :name }, { :allow_empty_search => true }

    Product.search("").size
    => 1
    
ignore_list:
  Pass in an ignore list location. Keywords in that list will be ignored.
  
    search_in :brand, :name, { :tags => :name }, { :ignore_list => Rails.root.join("config", "ignorelist.yml") }

  The list should look like:
    
    ignorelist:
      a, an, to, from, as
      
  You can include how many keywords you like.

TODO
----

* Strip html with sanitize (https://github.com/rgrove/sanitize)
* Rewrite and test relevant search
* Move all configurations to a configuration file. Maybe /config/mongoid_search.yml.

