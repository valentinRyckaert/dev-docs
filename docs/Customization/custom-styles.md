# Customize styles

TeSS' UI is built on [Bootstrap 3](https://getbootstrap.com/docs/3.4/). 
To customize the appearance of TeSS, you can either select one of the pre-made themes, or create your own. 

### Switching theme

To change the site default theme, edit your `config/tess.yml` config file and change (or add if not present) 
the `default_theme` variable under the `site` section to reference one of the configured themes 
(listed under the `themes` section in `config/tess.yml` or `config/tess.example.yml`):

```yml
default: &default
  # ...
  site:
    # ...
    default_theme: green
```

### Creating your own theme

1. Go into the themes folder (`app/assets/stylesheets/themes`) and make a copy of the default theme file (`default.scss`).

2. Add your theme to the list of themes in `config/tess.yml` (see `config/tess.example.yml` for reference). 
   The `primary` and `secondary` properties are colours that will be used in the various SVG icons in TeSS.  
```yml
default: &default
  # ...  
  themes:
    default:
      primary: '#047eaa' # Blue by default
      secondary: '#f47d21' # Orange by default
    # ...
    my_theme:
      primary: '#ff0000'
      secondary: '#ffff00' 
```

3. Enable your theme as the default them according to the section above.

4. Create a symlink for the theme's SVG images:

```shell
    ln -s app/assets/images/modern app/assets/images/themes/my_theme
```

*Alternatively, if you would like to make changes to the SVG images (beyond the colour changes that are automatically applied),
you may want to copy the images instead of making a symlink.*

In your theme's .scss file you can tweak any of the existing variables, add overrides for any Bootstrap variables ([reference](https://github.com/twbs/bootstrap-sass/blob/master/assets/stylesheets/bootstrap/_variables.scss)) 
and add new CSS/Sass styles (look at the other themes in that directory, and the `mixins/_modern_base.scss` mixin for reference).

Be sure to recompile assets (`bundle exec rake assets:recompile`), or rebuild your docker image, 
and restart the application to apply the new styles (except in development mode, where CSS changes should be applied automatically).

You can visit <http://localhost:3000/theme_showcase> on your local instance to see various UI elements with the
current theme applied. 

You can also add a query parameter `theme_preview` to the URL of any page to preview different themes, e.g. <http://localhost:3000/theme_showcase?theme_preview=blue>