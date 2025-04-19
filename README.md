# 8st Blog Posts

I built my personal website using [Hexo](https://hexo.io) with [Butterfly theme](https://github.com/jerryc127/hexo-theme-butterfly) on github pages

## Branches

The `main` branch will cover the source materials and enviroments setting for hexo, and the `gh-pages` branch is the release branch.

## Development

Use local machine to start server by 
```bash
$ hexo server
```

## Deployment

When everything is ready to deploy, generate the contents for github to deploy.
```bash
$ hexo clean && hexo generate
```

And then upload to the `gh-pages` branch to deploy.
```bash
$ hexo deploy
```

Also, upload the change in source materials to main branch using `git`.