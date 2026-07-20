# What is a feature?

A feature is a functionnality which can be activated from the `config/tess.yml` file.

HEP Training has a specific configuration where some features are enabled and others not.

You can find the details of each feature [here](./list-features.md).

## HEP Training features configuration

```yaml
feature:
    # TeSS main resources
    events: true
    materials: true
    elearning_materials: false
    workflows: false
    collections: true
    learning_paths: true
    content_providers: true
    sources: false
    nodes: false
    spaces: true
    api_system_for_groups: https://authorization-service-api.web.cern.ch/api/v1.0

    # Resources' features
    subscription: true
    geocoding: false
    materials_disabled: []
    content_providers_disabled: []
    bioschemas_testing: false
    collection_curation: true
    auto_parse_vars: [] # available features to auto parse from description: ['keywords', 'target_audience']
    controlled_vocabulary_vars: [] # available features: ['target_audience']

    # User login
    invitation: false
    registration: true
    login_through_oidc_only: false # when true, only the first oidc authentication will be available in TeSS, useful when you have your org SSO. Caution! invitation and registration must be false when using this feature.

    # User rights
    user_source_creation: true
    trainers: false
    edit_suggestions: false

    # UI
    sticky_navbar: true # when true, allows navbar (and header_notice if enabled) to stick to the top of the window and shrink when scrolling

    # Possible features to disable:
    #  biotools, topics, operations, sponsors, fairshare, county, ardc_fields_of_research,
    #  other_types, subsets, syllabus, approved_editors, address_finder, 
    #  visibility (if included, material or event can be hidden from the materials/events list)
    disabled: ['ardc_fields_of_research', 'other_types', 'subsets', 'syllabus', 'approved_editors']
```