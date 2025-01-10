# My Blog Repo

## Setup
1. Clone repo
2. Run `bundle install`
3. Run `bundle exec jekyll serve --drafts` to run local server
4. Browse to 'http://localhost:4000/'

## Commands
(Taken from `jekyll-compose` readme.)

```sh
  draft      # Creates a new draft post with the given NAME
  post       # Creates a new post with the given NAME
  publish    # Moves a draft into the _posts directory and sets the date
  unpublish  # Moves a post back into the _drafts directory
  page       # Creates a new page with the given NAME
```

Create new page:

    $ bundle exec jekyll page "My New Page"

Create new post:

    $ bundle exec jekyll post "My New Post"

Create new draft:

    $ bundle exec jekyll draft "My new draft"

Publish draft:

    $ bundle exec jekyll publish _drafts/my-new-draft.md
    # or specify a specific date on which to publish it
    $ bundle exec jekyll publish _drafts/my-new-draft.md --date 2014-01-24

Unpublish post:

    $ bundle exec jekyll unpublish _posts/2014-01-24-my-new-draft.md

Create new post (alternative):

- add a file in the `_posts` or `_drafts` directory
- follow the convention `YYYY-MM-DD-name-of-post.md` and include the necessary front matter

## Links

1. [Jekyll Docs](https://jekyllrb.com/docs/home)
2. [Jekyll Github](https://github.com/jekyll/jekyll)
3. [Jekyll Talk](https://talk.jekyllrb.com/)
