# WordPress API explorer

Minimal UI built with [Vue.js](https://vuejs.org/) to debug & check security for WordPress sites.

There are may sites that "restrict" content or files behind payment or user roles, but that content or files are still accessible through WordPress API.
This project helps with checking what data is exposed throught the API. Open the app from link below and fill any WordPress site url:

**→** [**WordPress API explorer**](https://wp-api-explorer.pages.dev/)

## Highlights
- Uses [WordPress REST API](https://developer.wordpress.org/rest-api/) to auto-discover all posts types & content
- Lists all content with pagination
- Displays all media (documents, images, videos, etc)
- Option to run requests through a proxy (if CORS Origin is not whitelisted)

## Development

- clone the repo
- run `npm install` to install all dependencies
- run `npm run serve` to start a local dev server
- make changes 😀
- create a Pull Request with updated code

### More

Please report any issues here on GitHub.

[Any contributions are welcome](CONTRIBUTING.md)
