# DTP16 public website

## Setup
This is a [Jekyll](http://jekyllrb.com/) page

1. Make sure you have Ruby installed (I recommend using [rbenv](http://rbenv.org/))
2. Install jekyll gem: `gem install jekyll`
3. Clone repo: `git clone git@github.com:DTP16/dtp16-web.git`
4. Go to app folder: `cd dtp-16-web`

## Development
1. Go to app folder
2. Change `baseurl` parameter in `_config.yml` to blank
3. Run `jekyll serve` for development. A local web server will be started on port 4000

## Deployment (for those who have the privileges)
1. Go to app folder
2. Run 'jekyll build'
3. Just copy the `_site` folder to your public www directory, 
e.g. `scp -r _site/. dtp16@pcai042.informatik.uni-leipzig.de:public_html/`

