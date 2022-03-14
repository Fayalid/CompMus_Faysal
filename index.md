---
author: Faysal el Kaaoichi
output:
  flexdashboard::flex_dashboard:
    md_document:
      variant: markdown_github
    orientation: columns
    storyboard: true
    vertical_layout: fill
title: Portfolio
---

`{r setup, include=FALSE} library(flexdashboard)`

## Introduction

### Introduction

Welcome to my repository! The topic of this corpus that is game music
with a focus on theme music. Over the years games have been evolving not
only in the fields of graphics or game mechanics but also in the field
of music. In the early stages of videogames, in the eighties, games
could not process more than three notes meaning it was not possible to
write a tune with complex multi-layered melodies. Over time this
changed. It is interesting to look at how this progressed and what the
significant changes are. As you can probably tell there is a huge amount
of categories out there. It is possible to make categories based on time
period or game genre or you can make distinctions based on the music
itself. What is also interesting to note is that the popular video games
in the beginning were mainly Japanese such as Mario and Final Fantasy.
So it could also be a fun project to compare Western and Japanese music.
I think that in the beginning because of the classic three notes it is
easier to identify game music but later on there would be a blurry
distinction between regular music and game music. It is interesting to
note that there is a grouping towards a very low valence and a very high
intrumentalness. This means that the music in this corpus, in general,
is pretty negative and uses almost no voices.

``` {.{r}}
```

## Column

### Visualisation

`{r load-packages, include=FALSE} library(dplyr) library(magrittr) library(knitr) library(ggplot2) library(tidyverse) library(spotifyr) library(compmus)`

``` {.{r}}
gp <-get_playlist_audio_features("","37i9dQZF1DXdfOcg1fm0VG?")
gp %>%                    
     ggplot(                     
         aes(
             x = valence,
             y = key,
             size = loudness,
             color =  instrumentalness)
     ) +
     geom_point() +              
     geom_rug(size = 0.1) #+      
#     geom_text(                 
#         aes(
#             x = valence,
#             y = energy
#        ))
```

------------------------------------------------------------------------

It is interesting to note that there is a grouping towards a very low
valence and a very high intrumentalness. This means that the music in
this corpus, in general, is pretty negative and uses almost no voices.

``` {.{r}}
mario <-
  get_tidy_audio_analysis("1RW6xbDYGaw0C6JPvf9rqd?") %>% # Change URI.
  compmus_align(bars, segments) %>%                     # Change `bars`
  select(bars) %>%                                      #   in all three
  unnest(bars) %>%                                      #   of these lines.
  mutate(
    pitches =
      map(segments,
        compmus_summarise, pitches,
        method = "rms", norm = "euclidean"              # Change summary & norm.
      )
  ) %>%
  mutate(
    timbre =
      map(segments,
        compmus_summarise, timbre,
        method = "rms", norm = "euclidean"              # Change summary & norm.
      )
  )
```

### timbre

``` {.{r}}
mario %>%
  compmus_gather_timbre() %>%
  ggplot(
    aes(
      x = start + duration / 2,
      width = duration,
      y = basis,
      fill = value
    )
  ) +
  geom_tile() +
  labs(x = "Time (s)", y = NULL, fill = "Magnitude") +
  scale_fill_viridis_c() +                              
  theme_classic() +
  labs(title = "Cepstrogram")
```

### self-similarity matrix

``` {.{r}}
mario %>%
  compmus_self_similarity(timbre, "cosine") %>% 
  ggplot(
    aes(
      x = xstart + xduration / 2,
      width = xduration,
      y = ystart + yduration / 2,
      height = yduration,
      fill = d
    )
  ) +
  geom_tile() +
  coord_fixed() +
  scale_fill_viridis_c(guide = "none") +
  theme_classic() +
  labs(title = "cosine self-similarity matrix", x = "", y = "")
```

### Comparing Mario theme songs

``` {.{r}}
## Super Mario Bros Classic
super_mario_bros <-
  get_tidy_audio_analysis("1RW6xbDYGaw0C6JPvf9rqd") %>%
  select(segments) %>%
  unnest(segments) %>%
  select(start, duration, pitches)
## Super Mario 64
super_mario_64 <-
  get_tidy_audio_analysis("2ZbxXIAU2zrRgWGfOOhsTd") %>%
  select(segments) %>%
  unnest(segments) %>%
  select(start, duration, pitches)

compmus_long_distance(
  super_mario_bros %>% mutate(pitches = map(pitches, compmus_normalise, "chebyshev")),
  super_mario_64 %>% mutate(pitches = map(pitches, compmus_normalise, "chebyshev")),
  feature = pitches,
  method = "euclidean"
) %>%
  ggplot(
    aes(
      x = xstart + xduration / 2,
      width = xduration,
      y = ystart + yduration / 2,
      height = yduration,
      fill = d
    )
  ) +
  geom_tile() +
  coord_equal() +
  labs(x = "Super Mario Bros", y = "Super Mario 64") +
  theme_minimal() +
  scale_fill_viridis_c(guide = NULL)
```
