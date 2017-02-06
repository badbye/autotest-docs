# Document For autotest

Setup locally:

```python
sudo pip install sphinx sphinx-autobuild

make html # build, it will generate a `build` folder

cd build/html && python -m SimpleHTTPServer  # open http://localhost:8000/ in Chrome
```

After pushing to Github, the [online document](http://autotest-docs.readthedocs.io/) will update automatically.
