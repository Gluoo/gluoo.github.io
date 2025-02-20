
## Preresuisites:
1. Ruby version 2.7.0 or higher
2. RubyGems
3. GCC and Make

```
$sudo apt-get install ruby-full build-essential zlib1g-dev
$gem install jekyll bundler
```


## Run the websit locally
1. Change directories to the docs folder
```
$ cd docs
```
2. Install the dependencies
```bash
$ bundle install
```
3. Run the Jekyll site locally
```bash
$ bundle exec jekyll serve
```
