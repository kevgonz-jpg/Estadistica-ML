la estadística es una herramienta esencial para cualquier rama del conocimiento, ya sea la ingeniera, estudios sociales, medicina, etc. La estadística permite obtener conclusiones mediante un conjunto de datos o suposiciones dadas, a diferencia de cualquier otro medio para obtener conclusiones la estadística permite realizar conclusiones basándose en los números presentados. Para esto veamos un ejemplo practico con un conjunto de datos que muchos serán capaces de tener conclusiones si necesidad de ser un experto.

Veamos el conjunto de datos de los ICFES, algo que muchos (por no decir todos) los estudiantes estarán familiarizados y naturalmente serán capaces de realizar algunas conclusiones

<center>fewfw<sup>fewfw</sup></center>

- [x] vvsvsdvsv    
- [x] vsfsfv
- [x] vsfvfs
- [x] bwbwtb
- [ ] tebw

|     | jknñ  | dvvsf |     |
| --- | ----- | ----- | --- |
|     | vfsdf | vffv  |     |
|     | vsfvs | vsffv |     |
|     |       |       |     |

## Representación Gráfica de un conjunto de datos

```R
require(plotly)
data("iris")

X <- subset(iris, select = -c(Species))

prin_comp <- prcomp(X, rank. = 3)

components <- prin_comp[["x"]]
components <- data.frame(components)
components$PC2 <- -components$PC2
components$PC3 <- -components$PC3
components = cbind(components, iris$Species)

tot_explained_variance_ratio <- summary(prin_comp)[["importance"]]['Proportion of Variance',]
tot_explained_variance_ratio <- 100 * sum(tot_explained_variance_ratio)

tit = 'Total Explained Variance = 99.48'

fig <- plot_ly(components, x = ~PC1, y = ~PC2, z = ~PC3, color = ~iris$Species, colors = c('#636EFA','#EF553B','#00CC96') ) %>%
  add_markers(size = 12)


fig <- fig %>%
  layout(
    title = tit,
    scene = list(bgcolor = "#e5ecf6")
  )

print(fig)
```

```R
# Load data
data("mtcars")
dfm <- mtcars
# Convert the cyl variable to a factor
dfm$cyl <- as.factor(dfm$cyl)
# Add the name colums
dfm$name <- rownames(dfm)
# Inspect the data
head(dfm[, c("name", "wt", "mpg", "cyl")])



# Calculate the z-score of the mpg data
dfm$mpg_z <- (dfm$mpg -mean(dfm$mpg))/sd(dfm$mpg)
dfm$mpg_grp <- factor(ifelse(dfm$mpg_z < 0, "low", "high"), 
                      levels = c("low", "high"))
# Inspect the data
head(dfm[, c("name", "wt", "mpg", "mpg_z", "mpg_grp", "cyl")])

library(ggpubr)
ggbarplot(dfm, x = "name", y = "mpg",
          fill = "cyl",               # change fill color by cyl
          color = "black",            # Set bar border colors to white
          palette = "jco",            # jco journal color palett. see ?ggpar
          sort.val = "asc",           # Sort the value in dscending order
          sort.by.groups = TRUE,      # Sort inside each group
          x.text.angle = 90           # Rotate vertically x axis texts
)
```


![[README-unnamed-chunk-2-1 1.gif]]![[README-unnamed-chunk-4-1.gif]]

---
