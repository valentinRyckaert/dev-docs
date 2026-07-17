# Database

HEP Training works with a PostgreSQL database.

## General Structure

### Users & Access

#### users
- **Links**: `roles`, `profiles`, `spaces`, `subscriptions`, `stars`, `collections`, `workflows`, `learning_paths`, `events`, `materials`, `content_providers`, `sources`, `bans`, `collaborations`, `space_roles`
- **Purpose**: application accounts, owner and author of resources, permissions, administration.

#### roles
- **Links**: `users`
- **Purpose**: definition of global authorization roles.

#### profiles
- **Links**: `users`
- **Purpose**: public profile information for a user.

#### bans
- **Links**: `users` (banned user, banning user)
- **Purpose**: user moderation and shadowbanning.

#### groups
- **Links**: `users`, `spaces`
- **Purpose**: intermediary for private spaces feature

### Spaces & Organization

#### spaces
- **Links**: `users`, `materials`, `events`, `workflows`, `collections`, `learning_paths`, `learning_path_topics`, `subscriptions`, `sources`, `space_roles`
- **Purpose**: multi-tenancy, dedicated spaces with enabled/disabled features.

#### space_roles
- **Links**: `spaces`, `users`
- **Purpose**: roles and permissions specific to a space.

#### subscriptions
- **Links**: `users`, `spaces`
- **Purpose**: alerts based on saved searches and email sending based on criteria.

### Content Sources

#### content_providers
- **Links**: `users`, `nodes`, `materials`, `events`, `learning_paths`, `sources`, `content_providers_users`
- **Purpose**: organization / content provider that groups resources and sources.

#### content_providers_users
- **Links**: `content_providers`, `users`
- **Purpose**: association of editors to a content provider.

#### nodes
- **Links**: `users`, `content_providers`, `node_links`, `staff_members`
- **Purpose**: network of entities or partners related to content.

#### staff_members
- **Links**: `nodes`
- **Purpose**: staff contacts and roles associated with a node.

#### sources
- **Links**: `content_providers`, `users`, `spaces`, `source_filters`, `external_resources`
- **Purpose**: ingestion origins / scrapers allowing import or update of resources.

#### source_filters
- **Links**: `sources`
- **Purpose**: filtering rules applied during ingestion.

#### external_resources
- **Links**: `sources`
- **Purpose**: external resources referenced by an ingestion source.

### Core Resources

#### materials
- **Links**: `users`, `content_providers`, `spaces`, `collections`, `event_materials`, `learning_path_topic_items`, `node_links`, `ontology_term_links`, `people`, `stars`, `link_monitors`
- **Purpose**: educational resources that can be cataloged, collected, and linked to other objects.

#### events
- **Links**: `users`, `content_providers`, `spaces`, `collections`, `event_materials`, `learning_path_topic_items`, `node_links`, `ontology_term_links`, `people`, `llm_interactions`, `link_monitors`
- **Purpose**: training events, conferences, and sessions published in the catalog.

#### learning_paths
- **Links**: `users`, `content_providers`, `spaces`, `learning_path_topic_links`, `stars`, `people`, `ontology_term_links`, `node_links`
- **Purpose**: structured learning paths composed of topics and resources.

#### learning_path_topics
- **Links**: `spaces`, `learning_path_topic_links`, `learning_path_topic_items`
- **Purpose**: internal topics of a learning path.

### Junction Tables and Resource Relationships

#### collection_items
- **Links**: `collections`, polymorphic resources (`materials`, `events`)
- **Purpose**: ordered link between a collection and its resources.

#### event_materials
- **Links**: `events`, `materials`
- **Purpose**: bidirectional association between events and related materials.

#### learning_path_topic_links
- **Links**: `learning_paths`, `learning_path_topics`
- **Purpose**: ordering and association of topics within a learning path.

#### learning_path_topic_items
- **Links**: `learning_path_topics`, polymorphic resources (`materials`, `events`)
- **Purpose**: resources associated with a learning path topic.

#### node_links
- **Links**: `nodes`, polymorphic resources
- **Purpose**: linking node-type entities to resources.

#### ontology_term_links
- **Links**: polymorphic resources
- **Purpose**: classification of resources by ontology terms.

#### people
- **Links**: polymorphic resources, `profiles`
- **Purpose**: authors, contributors, and other people associated with a resource.

#### stars
- **Links**: `users`, polymorphic resources
- **Purpose**: marking/favoriting a resource by a user.

#### collaborations
- **Links**: `users`, polymorphic resources
- **Purpose**: managing collaborators on resources.

#### edit_suggestions
- **Links**: polymorphic resources
- **Purpose**: proposed edits to existing content.

#### field_locks
- **Links**: polymorphic resources
- **Purpose**: field locking to prevent editing conflicts.

#### link_monitors
- **Links**: polymorphic resources
- **Purpose**: tracking the status of external links on resources.

#### widget_logs
- **Links**: polymorphic resources
- **Purpose**: logging interactions with the embedded widget.

#### friendly_id_slugs
- **Links**: polymorphic resources
- **Purpose**: friendly URL slugs for various models.

#### groups_spaces
- **Links**: `groups`, `spaces`
- **Purpose**: intermediary for many-to-many relation between groups and spaces, used for private spaces

#### group_membership
- **Links**: `users`, `groups`
- **Purpose**: intermediary for many-to-many relation between groups and users (and owners), used for private spaces

### Search, Analytics, and Logs

#### autocomplete_suggestions
- **Purpose**: autocomplete values for search.

#### ahoy_visits
- **Links**: `users`
- **Purpose**: web visit tracking for analytics.

#### ahoy_events
- **Links**: `ahoy_visits`, `users`
- **Purpose**: browsing event tracking / analytics.

#### activities
- **Purpose**: general activity / action history on resources.