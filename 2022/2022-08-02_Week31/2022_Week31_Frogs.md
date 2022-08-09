\#TidyTuesday <br>Week 31 of 2022 \| Oregon Spotted Frogs
================
Jadey N Ryan <br>@jadeynryan <br>GitHub \| Twitter

## Data source

\#TidyTuesday data source: [USGS Oregon Spotted
Frogs](https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-08-02)

### Citation

> Pearl, C.A., Rowe, J.C., McCreary, B., and Adams, M.J., 2022, Oregon
> spotted frog (*Rana pretiosa*) telemetry and habitat use at Crane
> Prairie Reservoir in Oregon, USA: U.S. Geological Survey data release,
> <https://doi.org/10.5066/P9DACPCV>.

## Load packages and data

``` r
library(cowplot)
library(dplyr)
library(ggplot2)
library(ggtext)
library(glue)
library(janitor)
library(knitr)
library(lubridate)
library(readr)
library(showtext)
library(skimr)
library(stringr)
library(sysfonts)
library(tidyr)
library(waffle)

frogs <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-08-02/frogs.csv')
```

## Data Exploration

In 2018, the USGS studied the Oregon spotted frog to understand their
**movement** and **habitat use** in the late-season (September to
November).

Let’s take a quick peek at beginning of the data frame and then skim
through its structure and summary statistics.

``` r
head(frogs)
```

    # A tibble: 6 × 16
      Site    Subsite HabType Surve…¹ Ordinal Frequ…² UTME_83 UTMN_83 Inter…³ Female
      <chr>   <chr>   <chr>   <chr>     <dbl>   <dbl>   <dbl>   <dbl>   <dbl>  <dbl>
    1 Crane … SE Pond Pond    9/25/2…     268    164.  597369 4846486       0      0
    2 Crane … SE Pond Pond    10/2/2…     275    164.  597352 4846487       1      0
    3 Crane … SE Pond Pond    10/9/2…     282    164.  597345 4846458       2      0
    4 Crane … SE Pond Pond    10/15/…     288    164.  597340 4846464       3      0
    5 Crane … SE Pond Pond    10/22/…     295    164.  597344 4846460       4      0
    6 Crane … SE Pond Pond    11/1/2…     305    164.  597410 4846451       5      0
    # … with 6 more variables: Water <chr>, Type <chr>, Structure <chr>,
    #   Substrate <chr>, Beaver <chr>, Detection <chr>, and abbreviated variable
    #   names ¹​SurveyDate, ²​Frequency, ³​Interval
    # ℹ Use `colnames()` to see all variable names

``` r
skimr::skim(frogs)
```

|                                                  |       |
|:-------------------------------------------------|:------|
| Name                                             | frogs |
| Number of rows                                   | 311   |
| Number of columns                                | 16    |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |       |
| Column type frequency:                           |       |
| character                                        | 10    |
| numeric                                          | 6     |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |       |
| Group variables                                  | None  |

Data summary

**Variable type: character**

| skim_variable | n_missing | complete_rate | min | max | empty | n_unique | whitespace |
|:--------------|----------:|--------------:|----:|----:|------:|---------:|-----------:|
| Site          |         0 |             1 |  13 |  13 |     0 |        1 |          0 |
| Subsite       |         0 |             1 |   5 |  14 |     0 |        6 |          0 |
| HabType       |         0 |             1 |   4 |   9 |     0 |        3 |          0 |
| SurveyDate    |         0 |             1 |   9 |  10 |     0 |       41 |          0 |
| Water         |         0 |             1 |   8 |  13 |     0 |        4 |          0 |
| Type          |         0 |             1 |   9 |  12 |     0 |        4 |          0 |
| Structure     |         0 |             1 |   4 |  14 |     0 |        5 |          0 |
| Substrate     |         0 |             1 |   5 |  17 |     0 |        4 |          0 |
| Beaver        |         0 |             1 |   5 |  14 |     0 |        4 |          0 |
| Detection     |         0 |             1 |   6 |   9 |     0 |        3 |          0 |

**Variable type: numeric**

| skim_variable | n_missing | complete_rate |       mean |      sd |         p0 |        p25 |        p50 |        p75 |      p100 | hist  |
|:--------------|----------:|--------------:|-----------:|--------:|-----------:|-----------:|-----------:|-----------:|----------:|:------|
| Ordinal       |         0 |             1 |     296.58 |   21.56 |     255.00 |     277.00 |     296.00 |     317.00 |     333.0 | ▅▇▆▆▇ |
| Frequency     |         0 |             1 |     164.59 |    0.28 |     164.17 |     164.42 |     164.52 |     164.73 |     165.5 | ▆▇▃▂▁ |
| UTME_83       |         0 |             1 |  597179.36 | 1138.76 |  594594.00 |  596454.50 |  597346.00 |  597963.50 |  599330.0 | ▃▂▇▆▂ |
| UTMN_83       |         0 |             1 | 4849688.69 | 2021.47 | 4846443.00 | 4846488.00 | 4850586.00 | 4851039.50 | 4851978.0 | ▆▁▂▆▇ |
| Interval      |         0 |             1 |       4.63 |    3.11 |       0.00 |       2.00 |       4.00 |       7.00 |      12.0 | ▇▅▇▅▂ |
| Female        |         0 |             1 |       0.72 |    0.45 |       0.00 |       0.00 |       1.00 |       1.00 |       1.0 | ▃▁▁▁▇ |

### Date wrangling

Since the study was interested in learning about the frogs’ movement
through the late-season, let’s wrangle the SurveyDate column so we can
explore their preferred habitats by month.

``` r
# Convert survey date class from character to date
frogs$SurveyDate <- as.Date(frogs$SurveyDate, "%m/%d/%y")

# Create new column for month of observation
frogs$Month <- lubridate::month(frogs$SurveyDate,
                                label = TRUE,
                                abbr = FALSE)

# Move Month after SurveyDate
frogs <- dplyr::relocate(frogs,
                         Month,
                         .after = SurveyDate)
```

### Some frog questions!

*Frequency* is the unique transmitter frequency associated with each
individual frog. To make this more clear, we will change the variable
name to FrogID and change the class from Numeric to Factor.

``` r
frogs <- dplyr::rename(frogs, FrogID = Frequency)
frogs$FrogID <- factor(frogs$FrogID)
```

**How many frogs were tracked?**

``` r
n_frogs <- dplyr::n_distinct(frogs$FrogID)
```

There were **32** frogs tracked in the study!

**How many times was each frog observed?**

``` r
frog_obs <- frogs %>%
  group_by(FrogID) %>%
  summarize(Count = n()) %>% 
  janitor::adorn_totals(where = "row")

knitr::kable(frog_obs)
```

| FrogID  | Count |
|:--------|------:|
| 164.169 |    10 |
| 164.195 |    10 |
| 164.357 |    12 |
| 164.371 |    11 |
| 164.381 |    10 |
| 164.395 |    10 |
| 164.407 |     9 |
| 164.417 |    11 |
| 164.432 |    10 |
| 164.445 |    10 |
| 164.457 |    10 |
| 164.47  |    10 |
| 164.482 |    13 |
| 164.495 |     5 |
| 164.506 |    10 |
| 164.52  |    12 |
| 164.533 |    10 |
| 164.545 |    10 |
| 164.557 |    12 |
| 164.57  |    11 |
| 164.582 |    12 |
| 164.595 |    11 |
| 164.732 |    10 |
| 164.768 |    10 |
| 164.796 |    12 |
| 164.861 |     8 |
| 164.941 |     9 |
| 165.095 |    10 |
| 165.12  |     3 |
| 165.129 |     3 |
| 165.209 |    10 |
| 165.496 |     7 |
| Total   |   311 |

``` r
summary(frog_obs$Count)
```

       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
       3.00   10.00   10.00   18.85   11.00  311.00 

A frog was observed as few as **3** times and at most **13** times. The
average number of observations for the frogs was about **10**. There
were 311 observations of the 32 frogs.

**Which habitat types did the frogs prefer each month?**

``` r
tbl_HabType <- table(frogs$Month,
                     frogs$HabType,
                     dnn = c("Month", "HabType")) %>%
  data.frame() %>%
  subset(Freq > 0) %>%
  dplyr::arrange(Month, desc(Freq)) %>%
  split(.[, "Month"], drop = TRUE) %>%
  purrr::map_df(., janitor::adorn_totals)
knitr::kable(tbl_HabType)
```

| Month     | HabType   | Freq |
|:----------|:----------|-----:|
| September | Reservoir |   25 |
| September | Pond      |   19 |
| September | River     |    8 |
| Total     | \-        |   52 |
| October   | Reservoir |   85 |
| October   | Pond      |   37 |
| October   | River     |   19 |
| Total     | \-        |  141 |
| November  | Reservoir |   63 |
| November  | Pond      |   39 |
| November  | River     |   16 |
| Total     | \-        |  118 |

**Note:** There were more than double the observations in October and
November than September. We’ll need to make this evident in the final
graphic.

Frogs were more often tracked in reservoirs, then ponds, then rivers for
each month of the study.

This wouldn’t be a very interesting graphic. Let’s try something else.

**How does the structure differ based on habitat type?**

``` r
tbl_HabType <- table(frogs$HabType,
                     frogs$Structure,
                     dnn = c("HabType", "Structure")) %>%
  data.frame() %>%
  subset(Freq > 0) %>%
  dplyr::arrange(HabType, desc(Freq)) %>%
  split(.[, "HabType"], drop = TRUE) %>%
  purrr::map_df(., janitor::adorn_totals)
knitr::kable(tbl_HabType)
```

| HabType   | Structure      | Freq |
|:----------|:---------------|-----:|
| Pond      | Herbaceous veg |   50 |
| Pond      | Woody debris   |   29 |
| Pond      | Open           |   15 |
| Pond      | Woody veg      |    1 |
| Total     | \-             |   95 |
| Reservoir | Herbaceous veg |  124 |
| Reservoir | Woody debris   |   32 |
| Reservoir | Open           |   15 |
| Reservoir | Woody veg      |    2 |
| Total     | \-             |  173 |
| River     | Herbaceous veg |   27 |
| River     | Open           |    8 |
| River     | Woody debris   |    4 |
| River     | Leaf litter    |    2 |
| River     | Woody veg      |    2 |
| Total     | \-             |   43 |

This looks more interesting! Let’s move on to the most fun part –
DataViz!

## Data Visualization

First, we’ll set up our fonts, colors, and text.

``` r
# Font
sysfonts::font_add_google("Poppins", "poppins")
showtext::showtext_auto()

# Colors
bg <- "#2C2818"
fg_text <- "#F5F2E5"

palette <- c("#BEC6A7", "#FFEFF0", "#BDE1D8", "#F5C379", "#E0F3FF")

# Text
title <- "Where to spot Oregon spotted frogs?"

subtitle <- "Using radio-telemetry, the U.S. Geological Survey monitored Oregon spotted frogs (*Rana pretiosa*) at the Crane Prairie Reservoir. To learn about their movements and seasonal habitat use, frogs were tracked weekly between September and late November of 2018."

caption <- "**#TidyTuesday** | Week 31 | August 2, 2022 <br>
            **Data**: USGS | doi.org/10.5066/P9DACPCV <br>
            **DataViz**: @jadeynryan on Twitter and GitHub"

label <- "Each circle is one frog observed in that habitat.\nThere were 311 observations of 32 frogs."

action <- stringr::str_wrap("Try spotting Oregon spotted frogs in the herbaceous areas of the reservoir!", 22)
```

Now for the waffle plots!

``` r
plot_frogs <- frogs %>%
  group_by(HabType, Structure) %>% 
  summarize(Count = n(), 
            .groups = "keep") %>% 
  ggplot(aes(fill = Structure, 
             values = Count)) +
  geom_waffle(color = bg,
              n_rows = 12, 
              size = 1,
              flip = TRUE,
              radius = unit(6, "pt")) +
  facet_wrap(~HabType, 
             strip.position = "bottom") +
  labs(
    title = title,
    subtitle = subtitle,
    caption = caption
  ) +
  scale_fill_manual(values = palette) +
  scale_x_discrete(expand = c(0,0)) +
  scale_y_continuous(expand = c(0,0)) +
  coord_equal() +
  theme_minimal() +
  waffle::theme_enhance_waffle() +
  theme(
    # Background
    panel.background = element_rect(fill = bg, 
                                    color = bg),
    plot.background = element_rect(fill = bg,
                                   color = bg),
    # Facet Label
    strip.text = element_text(color = fg_text,
                              face = "bold",
                              size = 16,
                              hjust = 0.5),
    # Text
    plot.title = element_text(color = fg_text,
                              size = 20,
                              face = "bold",
                              margin = margin(12,12,12,12)), 
    plot.subtitle = element_textbox_simple(color = fg_text, 
                                           size = 11, 
                                           lineheight = 1,
                                           margin = margin(2,6,12,0)),
    plot.caption = element_markdown(color = fg_text,
                                    lineheight = 1.2),
    # Legend
    legend.position = "none"
    )

plot_frogs
```

![](2022_Week31_Frogs_files/figure-gfm/unnamed-chunk-10-1.png)

We’ll use `{cowplot}` to draw some explanatory text and create our own
pretty legend.

``` r
# x location and hjust for legend so each item is aligned
x <- 0.83

cowplot_frogs <- cowplot::ggdraw() +
  draw_plot(plot_frogs) +
  # Action or takeaway point for the reader of the graphic
  draw_label(
    label = action,
    x = 0.174,
    y = 0.625,
    color = fg_text,
    size = 12,
    fontface = "bold"
  ) +
  # Label explaining that each circle is a frog observed
  draw_label(
    label = label,
    x = 0.02,
    y = 0.045,
    hjust = 0,
    vjust = 0,
    color = fg_text,
    size = 12,
    fontface = "bold"
  ) +
  # Legend
  draw_label(
    label = "Habitat Structure",
    x = x,
    y = 0.71,
    color = fg_text,
    size = 14,
    fontface = "bold"
  ) +
  draw_line(
    x = c(0.70, 0.96),
    y = 0.675,
    color = fg_text
  ) +
  draw_label(
    label = "Woody vegetation",
    x = x,
    y = 0.64,
    size = 14,
    fontface = "bold",
    color = "#E0F3FF"
  ) +
  draw_label(
    label = "Woody debris",
    x = x,
    y = 0.59,
    size = 14,
    fontface = "bold",
    color = "#F5C379"
  ) +
  draw_label(
    label = "Open",
    x = x,
    y = 0.54,
    size = 14,
    fontface = "bold",
    color = "#BDE1D8"
  ) +
  draw_label(
    label = "Leaf litter",
    x = x,
    y = 0.49,
    size = 14,
    fontface = "bold",
    color = "#FFEFF0"
  ) +
  draw_label(
    label = "Herbaceous \nvegetation",
    x = x,
    y = 0.42,
    size = 14,
    fontface = "bold",
    color = "#BEC6A7"
  )

cowplot_frogs
```

![](2022_Week31_Frogs_files/figure-gfm/unnamed-chunk-11-1.png)

Finally, we’ll save the plot!

``` r
ggsave("frogs.png")
```

    Saving 7 x 5 in image

## Session Info

``` r
sessionInfo()
```

    R version 4.2.0 (2022-04-22)
    Platform: x86_64-apple-darwin17.0 (64-bit)
    Running under: macOS Big Sur/Monterey 10.16

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.2/Resources/lib/libRblas.0.dylib
    LAPACK: /Library/Frameworks/R.framework/Versions/4.2/Resources/lib/libRlapack.dylib

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    other attached packages:
     [1] waffle_1.0.1    tidyr_1.2.0     stringr_1.4.0   skimr_2.1.4    
     [5] showtext_0.9-5  showtextdb_3.0  sysfonts_0.8.8  readr_2.1.2    
     [9] lubridate_1.8.0 knitr_1.39      janitor_2.1.0   glue_1.6.2     
    [13] ggtext_0.1.1    ggplot2_3.3.6   dplyr_1.0.9     cowplot_1.1.1  

    loaded via a namespace (and not attached):
     [1] Rcpp_1.0.9         assertthat_0.2.1   digest_0.6.29      utf8_1.2.2        
     [5] plyr_1.8.7         R6_2.5.1           repr_1.1.4         evaluate_0.15     
     [9] highr_0.9          pillar_1.8.0       rlang_1.0.4        curl_4.3.2        
    [13] rstudioapi_0.13    extrafontdb_1.0    DT_0.23            rmarkdown_2.14    
    [17] labeling_0.4.2     extrafont_0.18     htmlwidgets_1.5.4  bit_4.0.4         
    [21] munsell_0.5.0      gridtext_0.1.4     compiler_4.2.0     xfun_0.31         
    [25] pkgconfig_2.0.3    base64enc_0.1-3    htmltools_0.5.2    tidyselect_1.1.2  
    [29] tibble_3.1.8       gridExtra_2.3      fansi_1.0.3        crayon_1.5.1      
    [33] tzdb_0.3.0         withr_2.5.0        grid_4.2.0         jsonlite_1.8.0    
    [37] Rttf2pt1_1.3.10    gtable_0.3.0       lifecycle_1.0.1    DBI_1.1.3         
    [41] magrittr_2.0.3     scales_1.2.0       cli_3.3.0          stringi_1.7.8     
    [45] vroom_1.5.7        farver_2.1.1       snakecase_0.11.0   xml2_1.3.3        
    [49] ellipsis_0.3.2     generics_0.1.2     vctrs_0.4.1        RColorBrewer_1.1-3
    [53] tools_4.2.0        bit64_4.0.5        markdown_1.1       purrr_0.3.4       
    [57] hms_1.1.1          parallel_4.2.0     fastmap_1.1.0      yaml_2.3.5        
    [61] colorspace_2.0-3  
