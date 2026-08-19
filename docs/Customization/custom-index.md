# Configuration files

This page gives a high-level view of the main configuration files used by TeSS and how they fit together. It is intended as a developer-oriented map rather than a full reference of every possible option.

## Overview

TeSS separates configuration by responsibility:

- [config/tess.yml](../config/tess.yml): application behavior, branding, and feature flags
- [config/secrets.yml](../config/secrets.yml): sensitive credentials and external service settings
- [config/locales/en.yml](../config/locales/en.yml): default English UI text
- [config/locales/overrides](../config/locales/overrides): local overrides for labels and wording

The actual loading logic is defined in [config/application.rb](../config/application.rb), which reads the instance configuration and merges defaults where needed.


## config/tess.yml — product configuration

This is the main app configuration file for a TeSS instance. It controls how the platform behaves and looks in a given deployment.

It is loaded with:

```ruby
config.tess = config_for(Rails.env.test? ? Pathname.new(Rails.root).join('test', 'config', 'test_tess.yml') : 'tess')
config.tess_defaults = config_for('tess.example')
```

This means:

- [config/tess.yml](../config/tess.yml) is the real instance configuration
- [config/tess.example.yml](../config/tess.example.yml) is the default schema and example values
- missing values can fall back to the example defaults

### Typical content

Examples include:

- base_url
- contact_email and sender_email
- site title, logo, default theme
- feature flags such as events, materials, spaces, registration, etc.
- available locales and default locale
- maps, mailer, notice banner, and UI preferences

This file is essentially the “settings for the TeSS instance” file.

### Default merging behavior

The app merges default settings into the live config in [config/application.rb](../config/application.rb):

```ruby
def self.merge_config(default_config, config, current_path = '')
  default_config.each do |key, value|
    unless config.key?(key)
      config[key] = value
    end
    merge_config(value, config[key], current_path + "#{key}: ") if value.is_a?(Hash) && config[key].is_a?(Hash)
  end
end

merge_config(Rails.configuration.tess_defaults.with_indifferent_access, tess_config)
```

So `tess.yml` is meant to override the default values without having to repeat the entire structure.


## config/secrets.yml — sensitive configuration

This file holds credentials and private configuration for the app and its integrations.

It is loaded with:

```ruby
config.secrets = config_for('secrets')
```

It is split by environment:

- development
- test
- production

and it includes a shared `external_api_keys` block.

### Typical content

Examples include:

- secret_key_base
- database connection settings
- API keys for analytics, maps, or external services
- OAuth and identity provider settings
- Fairsharing, BioPortal, GPT, Indico, ORCID settings
- production SMTP configuration

This file is reserved for values that should not be widely exposed or committed as plain config in the repository.

### Why it is separate from tess.yml

- [config/tess.yml](../config/tess.yml) is about app behavior and presentation
- [config/secrets.yml](../config/secrets.yml) is about credentials and integration secrets

A developer should not put private credentials into the app's main feature config file.


## config/locales/en.yml — default translation file

The file [config/locales/en.yml](../config/locales/en.yml) contains the default English strings used by the application. It stores the base language content for the UI: labels, messages, static text, and translations.

This is separate from the app config files because it deals with human-readable text, not runtime behavior or secrets.

### What it is used for

- general UI labels
- forms and page copy
- default translations used by the app
- shared wording across the system


## config/locales/overrides — per-instance local customizations

TeSS supports a locale override mechanism to customize wording for a specific deployment without modifying the upstream base translations.

This behavior is enabled in [config/application.rb](../config/application.rb):

```ruby
config.i18n.load_path += Dir[Rails.root.join('config', 'locales', 'overrides', '**', '*.{rb,yml}')] unless Rails.env.test?
```

This means files placed under [config/locales/overrides](../config/locales/overrides) are automatically loaded, except in the test environment.

### Typical use cases

- changing a label used across a specific deployment
- customizing wording to match a local institution or project branding
- overriding default English copy without editing upstream files

This is the right place for text-level customization, not for secrets or feature toggles.
