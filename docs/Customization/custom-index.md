# Configuration on TeSS

TeSS can be customized and configured in a variety of ways to suit the needs of your deployment.

## General configuration

General TeSS settings are configured in `config/tess.yml`. Examples of settings include:

- `base_url` - The base URL that this TeSS instance is deployed at. Used to generate full URLs in emails etc.
- `contact_email` - The address to which support requests should be sent.
- `site` - Various options for customizing the name and logo of this TeSS instance.
- `feature` - Enable/disable types of resource (Event, Material etc.).
- `feature` > `disabled` - Hide various fields.

Secure settings such as database credentials, API keys, email settings etc. can be configured in `config/secrets.yml`.