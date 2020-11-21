# cloudblurbs.com

Welcome to `cloudblurbs.com` source code. This is the source code behind the [cloudblurbs.com](https://cloudblurbs.com) blog, by [Avishay Bar](https://www.linkedin.com/in/avishaybar).

## Tech

`cloudblurbs` is built with the following tech:
- [Jekyll](https://jekyllrb.com) - Transform your plain text into static websites and blogs.
- [Chalk](https://github.com/nielsenramon/chalk) - a high quality, completely customizable, performant and free blog template for Jekyll.
- [GitHub pages](https://pages.github.com) - Websites for you and your projects, Hosted directly from your GitHub repository.

Other integrations worth mentioning:
- [Disqus](https://disqus.com/)
- [Google Analytics](https://analytics.google.com/analytics/web/)
- [Circle CI](https://circleci.com/)
- [Html-proofer](https://github.com/gjtorikian/html-proofer)

## Building the Blog Locally:

### Installation

If you haven't installed the following tools then go ahead and do so (make sure you have [Homebrew](https://brew.sh/) installed):

    brew install ruby
    brew install npm

On windows, install Ruby and Node with the installers found here:

  - [Ruby](https://rubyinstaller.org/)
  - [Node.js](https://nodejs.org/en/download/)

Next setup your environment:

    npm run setup

### Development

Run Jekyll:

    npm run local

## Deploy to GitHub Pages

Before you deploy, commit your changes to any working branch except the `gh-pages` one and run the following command:

    npm run publish

You can find more info on how to use the `gh-pages` branch and a custom domain [here](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/).

## License

MIT License
