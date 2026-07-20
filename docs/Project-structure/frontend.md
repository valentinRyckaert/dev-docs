# Frontend

The frontend of this Rails project is built with a classic Rails asset-pipeline structure and server-side rendering.

## Technologies used
- [ERB](../dictionnary.md#embedded-ruby)
- [SCSS](../dictionnary.md#scss)
- [Handlebars](../dictionnary.md#handlebarsjs)
- JavaScript ([jQuery](https://api.jquery.com/), [Turbolinks](https://github.com/turbolinks/turbolinks)...)
- [Bootstrap](https://getbootstrap.com/docs/3.4/)

Other usefull Docs: [Rails asset pipeline](https://guides.rubyonrails.org/asset_pipeline.html), [Sprockets directives](https://github.com/rails/sprockets#sprockets-directives)


## Key directories

#### app/views
Contains Rails ERB templates for pages, partials and layouts.

#### app/assets/stylesheets
Contains SCSS/CSS entrypoints and partial stylesheets.

#### app/assets/javascripts
Contains JavaScript entrypoints and feature-specific scripts.

#### app/assets/javascripts/templates
Contains Handlebars (`.hbs`) templates used client-side.


## Example: frontend flow for workflows

This example from the code ties together ERB, JS, and Handlebars.

### 1. ERB view: modal shell
File: _node_modal.html.erb

- Defines the modal HTML structure
- Includes buttons and form fields
- Leaves dynamic sections empty:
  - `#node-modal-associated-resource-list`
  - `#node-modal-ontology-terms-list`
- Contains inline JS for autocomplete initialization

### 2. Handlebars template: resource form
File: associated_resource_form.hbs

- Defines a reusable client-side form fragment
- Uses placeholders like `{{title}}`, `{{url}}`, `{{type}}`, `{{icon}}`

### 3. JavaScript controller
File: workflows.js

- Manages workflow editor behaviors
- Adds new associated resources using:
  - `HandlebarsTemplates['workflows/associated_resource_form'](...)`
- Populates/removes form fragments dynamically
- Reads values from DOM and serializes them into JS objects
- Handles undo/redo, modal state, and UI interactions

### 4. SCSS styling
File: workflows.scss (imported by application.scss)

- Defines styling for workflow screens, modal layouts, and form controls
- Works together with Bootstrap base styles