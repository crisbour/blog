# Blog

My blog build with Quarto

Based on template
[build-your-own-blog](https://github.com/ivelasq/build-your-own-blog-exercises)

Actions inspired from [wasimlorgat blog](https://github.com/seeM/blog).

Other blogs of interest to draw ideas from:
- https://github.com/Bioconductor/biocblog/tree/main
- https://github.com/pjastam/blog-quarto
- [Quarto
Workshop](https://github.com/jadeynryan/parameterized-quarto-workshop/tree/main)
=> build presentation as part of Quarto

## Tools

Use [quarto-actions](https://github.com/quarto-dev/quarto-actions) to build in
publish the website.
In order to support Rmarkdown, package needs to be added:

```sh
R -e 'install.packages(c("rmarkdown", "knitr"))'
```
