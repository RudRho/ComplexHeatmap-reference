

# More Examples of Making Complex Heatmaps


In the supplementaries of [the ComplexHeatmap paper](http://bioinformatics.oxfordjournals.org/content/early/2016/05/20/bioinformatics.btw313.abstract), there are four comprehensive examples which are applied
on real-world high-throughput datasets. [The examples can be found here.](http://jokergoo.github.io/supplementary/ComplexHeatmap-supplementary1-4/index.html)

Also [my blog](http://jokergoo.github.io/blog.html) has some examples and tips for making better complex heatmaps.


### Add more information for gene expression matrix

Heatmaps are very popular to visualize gene expression matrix. 
Rows in the matrix correspond to genes and more information on these genes can be attached after the expression
heatmap.

In following example, the big heatmap visualize relative expression for genes, then the next is the absolute expression.
Also gene length and gene type (i.e. protein coding or lincRNA) are visualized.



```r
library(ComplexHeatmap)
library(circlize)

expr = readRDS(paste0(system.file(package = "ComplexHeatmap"), "/extdata/gene_expression.rds"))
mat = as.matrix(expr[, grep("cell", colnames(expr))])
base_mean = rowMeans(mat)
mat_scaled = t(apply(mat, 1, scale))

type = gsub("s\\d+_", "", colnames(mat))
ha = HeatmapAnnotation(df = data.frame(type = type))

Heatmap(mat_scaled, name = "expression", km = 5, col = colorRamp2(c(-2, 0, 2), c("green", "white", "red")),
    top_annotation = ha, 
    show_row_names = FALSE, show_column_names = FALSE) +
Heatmap(base_mean, name = "base_mean", show_row_names = FALSE, width = unit(5, "mm")) +
Heatmap(expr$length, name = "length", col = colorRamp2(c(0, 1000000), c("white", "orange")),
    heatmap_legend_param = list(at = c(0, 200000, 400000, 60000, 800000, 1000000), 
                                labels = c("0kb", "200kb", "400kb", "600kb", "800kb", "1mb")),
    width = unit(5, "mm")) +
Heatmap(expr$type, name = "type", width = unit(5, "mm"))
```

<img src="09-examples_files/figure-html/expression_example-1.png" width="960" style="display: block; margin: auto;" />

### Visualize genomic regions and other correspondance

Following example visualizes correlation between methylation and expression, as well as other annotation information (data are randomly generated). In the heatmap, each row corresponds to a differentially methylated regions (DMRs). 
From left to right, heatmaps are:

1. methylation for each DMR (by rows) in samples.
2. direction of the methylation (one column heatmap), i.e. is methylation hyper in tumor or hypo?
3. expression for the genes that are associated with corresponding DMRs (e.g. closest gene).
4. significance for the correlation between methylation and expression (-log10(p-value)).
5. type of genes, i.e. is the gene a protein coding gene or a lincRNA?
6. annotation to gene models, i.e. is the DMR located in the intragenic region of the corresponding gene or the DMR is intergenic?
7. distance from the DMR to the TSS of the corresponding gene.
8. overlapping between DMRs and enhancers (Color shows how much the DMR is covered by the enhancers).



```
## Warning: The input is a data frame, convert it to the matrix.
```

<img src="09-examples_files/figure-html/unnamed-chunk-2-1.png" width="960" style="display: block; margin: auto;" />


## Combine pvclust and heatmap

**pvclust** package provides a robust way to test the stability of the clustering
by random sampling from original data. Here you can organize the heatmap by the clustering
returned from `pvclust()`.


```r
library(ComplexHeatmap)

library(MASS)
library(pvclust)
data(Boston)
boston.pv <- pvclust(Boston, nboot=100)
```

```
## Bootstrap (r = 0.5)... Done.
## Bootstrap (r = 0.6)... Done.
## Bootstrap (r = 0.7)... Done.
## Bootstrap (r = 0.8)... Done.
## Bootstrap (r = 0.9)... Done.
## Bootstrap (r = 1.0)... Done.
## Bootstrap (r = 1.1)... Done.
## Bootstrap (r = 1.2)... Done.
## Bootstrap (r = 1.3)... Done.
## Bootstrap (r = 1.4)... Done.
```

```r
plot(boston.pv)
```

<img src="09-examples_files/figure-html/unnamed-chunk-3-1.png" width="576" style="display: block; margin: auto;" />

Since by default `pvclust` clusters columns by 'correlation' method, we scale columns for
`Boston` data set to see the relative trend.


```r
Boston_scaled = apply(Boston, 2, scale)
Heatmap(Boston_scaled, cluster_columns = boston.pv$hclust, heatmap_legend_param = list(title = "Boston"))
```

<img src="09-examples_files/figure-html/unnamed-chunk-4-1.png" width="576" style="display: block; margin: auto;" />

## Make a same plot as heatmap()


```r
set.seed(123)
mat = matrix(rnorm(100), 10)
heatmap(mat, col = topo.colors(50))
```

<img src="09-examples_files/figure-html/unnamed-chunk-5-1.png" width="576" style="display: block; margin: auto;" />

Compare to the native `heatmap()`, `Heatmap()` can give more accurate interpolation
for colors for continous values.


```r
Heatmap(mat, col = topo.colors(50), color_space = "sRGB",
    row_dend_width = unit(2, "cm"), 
    column_dend_height = unit(2, "cm"), row_dend_reorder = TRUE,
    column_dend_reorder = TRUE)
```

<img src="09-examples_files/figure-html/unnamed-chunk-6-1.png" width="576" style="display: block; margin: auto;" />

## The measles vaccine heatmap

Following code reproduces the heatmap introduced [here](https://biomickwatson.wordpress.com/2015/04/09/recreating-a-famous-visualisation/) and [here](https://benjaminlmoore.wordpress.com/2015/04/09/recreating-the-vaccination-heatmaps-in-r/).


```r
mat = readRDS(paste0(system.file("extdata", package = "ComplexHeatmap"), "/measles.rds"))
ha1 = HeatmapAnnotation(dist1 = anno_barplot(colSums(mat), bar_width = 1, gp = gpar(col = NA, fill = "#FFE200"), 
    border = FALSE, axis = TRUE))
ha2 = rowAnnotation(dist2 = anno_barplot(rowSums(mat), bar_width = 1, gp = gpar(col = NA, fill = "#FFE200"), 
    border = FALSE, which = "row", axis = TRUE), width = unit(1, "cm"))
ha_column = HeatmapAnnotation(cn = function(index) {
    year = as.numeric(colnames(mat))
    which_decade = which(year %% 10 == 0)
    grid.text(year[which_decade], which_decade/length(year), 1, just = c("center", "top"))
})
Heatmap(mat, name = "cases", col = colorRamp2(c(0, 800, 1000, 127000), c("white", "cornflowerblue", "yellow", "red")),
    cluster_columns = FALSE, show_row_dend = FALSE, rect_gp = gpar(col= "white"), show_column_names = FALSE,
    row_names_side = "left", row_names_gp = gpar(fontsize = 10),
    column_title = 'Measles cases in US states 1930-2001\nVaccine introduced 1961',
    top_annotation = ha1,
    bottom_annotation = ha_column) + ha2

decorate_heatmap_body("cases", {
    i = which(colnames(mat) == "1961")
    x = i/ncol(mat)
    grid.lines(c(x, x), c(0, 1), gp = gpar(lwd = 2))
    grid.text("Vaccine introduced", x, unit(1, "npc") + unit(5, "mm"))
})
```

<img src="09-examples_files/figure-html/unnamed-chunk-7-1.png" width="960" style="display: block; margin: auto;" />

## Session info


```r
sessionInfo()
```

```
## R version 3.4.4 (2018-03-15)
## Platform: x86_64-apple-darwin15.6.0 (64-bit)
## Running under: macOS High Sierra 10.13.2
## 
## Matrix products: default
## BLAS: /Library/Frameworks/R.framework/Versions/3.4/Resources/lib/libRblas.0.dylib
## LAPACK: /Library/Frameworks/R.framework/Versions/3.4/Resources/lib/libRlapack.dylib
## 
## locale:
## [1] en_GB.UTF-8/en_GB.UTF-8/en_GB.UTF-8/C/en_GB.UTF-8/en_GB.UTF-8
## 
## attached base packages:
## [1] grid      methods   stats     graphics  grDevices utils     datasets 
## [8] base     
## 
## other attached packages:
## [1] pvclust_2.0-0         MASS_7.3-50           RColorBrewer_1.1-2   
## [4] circlize_0.4.5        ComplexHeatmap_1.99.0
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.18        bookdown_0.7        digest_0.6.17      
##  [4] rprojroot_1.3-2     backports_1.1.2     magrittr_1.5       
##  [7] evaluate_0.11       GlobalOptions_0.1.1 stringi_1.2.4      
## [10] GetoptLong_0.1.7    rmarkdown_1.10      rjson_0.2.20       
## [13] tools_3.4.4         stringr_1.3.1       xfun_0.3           
## [16] yaml_2.2.0          compiler_3.4.4      colorspace_1.3-2   
## [19] shape_1.4.4         htmltools_0.3.6     knitr_1.20
```