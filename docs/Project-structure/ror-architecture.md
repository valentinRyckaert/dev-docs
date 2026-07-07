# Ruby On Rails

The TeSS project is built with Ruby On Rails, a backend MVC Framework following the Model-View-Controller (MVC) architecture pattern, which separates the application into three interconnected layers that handle different responsibilities.

!!! note
    This page is a quick reminder of how Ruby on Rails works and how the main architecture works. If you are already familiar with Ruby on Rails, you can use this page to refresh your memory.
    But if you are not familiar with Ruby on Rails, we strongly recommend to check these ressources :
    
    - [Ruby on Rails documentation](https://rubyonrails.org/docs)
    - [Ruby on Rails in 100 seconds](https://www.youtube.com/watch?v=2DvrRadXwWY)
    - [Ruby on Rails full course (4 hours)](https://www.youtube.com/watch?v=fmyvWz5TUWg)

## The MVC Pattern

| Component | Purpose | Files Location |
|---|---|---|
| **Model** | Represents the application's data and business logic. Handles database interactions, validations, and relationships between data entities. | `app/models/` |
| **View** | Presents data to users. Generates HTML, JSON, or other output formats that users see and interact with. | `app/views/` |
| **Controller** | Acts as the traffic director. Receives requests, queries models for data, and tells views what to render. | `app/controllers/` |


## Core Directories

```
app/
├── models/              # Database objects and business logic
├── views/               # HTML templates (ERB, Haml, Slim)
├── controllers/         # Request handlers
├── helpers/             # View helper methods
├── assets/              # JavaScript, CSS, images
└── jobs/                # Background job classes

config/
├── routes.rb            # URL routing rules
├── database.yml         # Database configuration
└── environments/        # Environment-specific settings

db/
├── migrate/             # Database migration files
└── seeds.rb             # Initial database data

test/                    # Test files (units, models, controllers)
```

## Key Conventions

Rails follows **convention over configuration**, meaning sensible defaults reduce the code you need to write:

- A `User` model automatically maps to a `users` database table
- A `UsersController` handles requests for users
- Views for users go in `app/views/users/`
- A controller action `index` automatically renders `index.html.erb`
