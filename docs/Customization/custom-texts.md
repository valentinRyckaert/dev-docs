# Customize texts

Much of the static text content of TeSS is sourced from YML files stored in `config/locales`, e.g. `en.yml`. 

If you wish to alter any of this text, instead of modifying the files directly, create a new YML file under 
`config/locales/overrides` with the same locale suffix (e.g. `config/locales/overrides/my_app.en.yml`).

In that file, you add any strings you want to override, for example:
```yml
  en:
    home:
      welcome: Welcome to my new training portal!
```

Files in the `overrides` directory will be automatically loaded, except when in the Rails `test` environment. 

Read more about Rails' internationalization here: https://guides.rubyonrails.org/i18n.html