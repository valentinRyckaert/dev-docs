# Solr

Apache Solr is a search platform built on Lucene. In TeSS, Solr powers full-text search, faceted filtering and result ranking for main catalog resources.

> **Note:** If you are not familiar with Solr, read the official docs first: https://solr.apache.org/guide/.

## What Solr is used for in this project

- Main search engine for `Event`, `Material`, `Collection`, `ContentProvider`, `Source`, `LearningPath`, `LearningPathTopic`, `Node`, `Profile`, `User`, and `Workflow`.
- Provides full-text search, facet counts, filtering and pagination.
- Enables search-backed listing in controllers via `SearchableIndex` and `SearchController`.
- Keeps search indexes in sync when resources change.
- Supports advanced filtering on keywords, publication type, content provider, status, and more.

## Main files to inspect

- `config/sunspot.example.yml` — Solr connection settings for development, test and production.
- `app/controllers/search_controller.rb` — top-level search entry point using `Sunspot.search`.
- `app/controllers/concerns/searchable_index.rb` — shared controller concern for Solr-backed index and count endpoints.
- `app/models/*.rb` — several models define `searchable do ... end` blocks for index fields and facets.
  - `app/models/content_provider.rb`
  - `app/models/event.rb`
  - `app/models/material.rb`
  - `app/models/collection.rb`
  - `app/models/learning_path.rb`
  - `app/models/learning_path_topic.rb`
  - `app/models/node.rb`
  - `app/models/profile.rb`
  - `app/models/source.rb`
  - `app/models/user.rb`
  - `app/models/workflow.rb`
- `app/models/collaboration.rb` — triggers reindex after collaborator changes.
- `app/models/collection_item.rb` — updates resource Solr index after collection changes.
- `app/models/link_monitor.rb` — checks whether a resource is searchable before reindexing.
- `solr/conf/solrconfig.xml` — Solr collection configuration.

## Project examples

### Search controller example

`app/controllers/search_controller.rb` uses Solr to execute full-text search across enabled models:

```ruby
@results[model_name.underscore.pluralize.to_sym] = Sunspot.search(model) do
  fulltext search_params
  with('end').greater_than(Time.zone.now) if model_name == 'Event'
  paginate page: 1, per_page: PAGE_SIZE
end
```

This shows how TeSS delegates web search queries to Solr and applies model-specific filtering before returning results.

### Shared listing behavior

`app/controllers/concerns/searchable_index.rb` routes index actions to Solr when enabled:

```ruby
@search_results = @model.search_and_filter(current_user, @search_params, @facet_params,
                                page: page, per_page: per_page, sort_by: @sort_by)
```

That method enables faceted search results, then wraps filtered records into a paginated response.

### Model indexing example

Models declare searchable fields inside `searchable do ... end` blocks. Example from `ContentProvider`:

```ruby
searchable do
  text :title
  text :description
  string :keywords, multiple: true
  integer :count do
    if self.events.count > self.materials.count
      self.events.count
    else
      self.materials.count
    end
  end
  integer :user_id
end
```

This defines how resource fields are indexed and which facets are exposed in Solr.

### Reindexing on change

TeSS keeps search indexes synchronized with active record events. Example from `app/models/collaboration.rb`:

```ruby
after_save do
  Sunspot.index(resource) if TeSS::Config.solr_enabled
end
```

And from `app/models/collection_item.rb`:

```ruby
resource.solr_index if TeSS::Config.solr_enabled
```

These hooks ensure that collections, collaborators and related resources remain searchable after updates.

## Practical notes

- Solr usage is gated by `TeSS::Config.solr_enabled`.
- `config/sunspot.example.yml` is the example template for actual `config/sunspot.yml`.
- Search and faceting behavior is centralized in controller concerns while schema details live in model searchable blocks.
