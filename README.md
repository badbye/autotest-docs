# Document For autotest

Online: [http://autotest-docs.readthedocs.io/](http://autotest-docs.readthedocs.io/)

## Edit

The documents of R locate at the `source/R` folder, while the documents of Python locates at the `source/Python` folder.

The documents are using `reStructuredText` format, which is very similar to Markdown but more powerful. Read this [tutorial](http://sphinx-doc-zh.readthedocs.io/en/latest/rest.html) to learn reStructuredText.


## Setup

Setup locally:

```python
sudo pip install sphinx sphinx-autobuild

make html # build, it will generate a `build` folder

cd build/html && python -m SimpleHTTPServer  # open http://localhost:8000/ in Chrome
```

After pushing to Github, the [online document](http://autotest-docs.readthedocs.io/) will update automatically.

