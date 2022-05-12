# Loggie documentation

This is the source file repository for Loggie documentation.

## Run the website locally
The Loggie documentation website is built using [mkdocs-material](https://squidfunk.github.io/mkdocs-material/).

The official Docker image is a great way to get up and running in a few minutes, as it comes with all dependencies pre-installed.

```
docker pull squidfunk/mkdocs-material

docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
```

Point your browser to localhost:8000 and you should see the website.
