---
title: "making_validation_csv"
author: "Patrick McKenzie"
date: "2024-01-30"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(rinat)
```

## Preparing a validation dataframe to share with Robin and Andrea for us all to validate. The goal is to sample the top 500 most abundance species in the full dataset that have data both from GPT-labeling and from TRY color data. 

## Load in TRY color data
```{r}
# full cleaned TRY data, encompassing more species than in our dataframe
#trydata <- read.csv('../data/TRY_cleaned_colordata.csv',stringsAsFactors = FALSE)
trydata <- read.csv('../data/cleaned_matched_colors.csv',stringsAsFactors = FALSE, row.names = 1)
```

## Load in full dataset with matched GPT color data

```{r}
# note that any "NA" or "UNKNOWN" colors have been removed
dat <- read.csv('../data/fulldata_cleaned_matched_GPT_colors.csv',stringsAsFactors = FALSE)
```

## Get ordered list of most common species in the data

```{r}
sp_occs <- sort(table(dat$binomial),decreasing=T)
ordered_names <- names(sp_occs)
```

## Find the top 250 most abundant species present in both color datasets

```{r}
binoms <- integer(0)
idx <- 1
while (length(binoms) < 250) {
  bin <- ordered_names[idx]
  if (tolower(bin) %in% trydata$species_name) {
    binoms <- c(binoms,bin)
  }
  
  idx <- idx + 1
}

binoms
```


## Make sure we have a list of acceptable colors

```{r}
allcolors_try <- trydata$OrigValueStr
unique(tolower(allcolors_try))

unique(dat$color)
```

## We might just want to lump colors strategically -- that is, anything "pink" "purple" or "violet" is correct, and anything "maroon" or "brown" is correct with either label. 

## What do we want in the final dataframe for validation? Perhaps: species name, representative photo? Is that all? I can match against GPT and TRY answers later.

## Get representative photos of the top 250 most abundant species

```{r}
# this dataframe has a representative inaturalist photo for each taxon
gpt_labeled_taxa <- read.csv('../data/FULL_gpt_labeled_taxon.csv')

photo_urls <- integer(0)
for (bin in binoms) {
  url <- gpt_labeled_taxa[gpt_labeled_taxa$binomial == bin,]$photo_url
  photo_urls <- c(photo_urls,url)
}

validation_df <- data.frame('binomial'=binoms, 'photo_url'=photo_urls)

# un-comment to write this out
#write.csv(validation_df,
#          '../TRY_GPT_validation/empty_scoring_df.csv',
#          row.names = FALSE)
```

# Load in some of the validated data:

```{r}
patrick <- read.csv('../TRY_GPT_validation/patrick_validation.csv')$color
robin <- read.csv('../TRY_GPT_validation/robin_validation.csv')$color
andrea <- read.csv('../TRY_GPT_validation/andrea_validation.csv')$color

try_cols <- integer(0)
for (spname in binoms) {
  sampled <- tolower(sample(trydata[trydata$species_name==tolower(spname),'color'],1))
  try_cols <- c(try_cols, sampled)
}

gpt_cols <- integer(0)
for (spname in binoms) {
  sampled <- sample(dat[dat$binomial==spname,'color'],1)
  gpt_cols <- c(gpt_cols, sampled)
}

purples <- c("pink","violet","purple")
reds <- c("red")
oranges <- c("orange")
browns <- c("brown","maroon","black")
yellows <- c("yellow")
blues <- c("blue")
whites <- c("white")
greens <- c("green")

# Lump similar colors
lumped_try_cols <- integer(0)
for (col in try_cols) {
  if (col %in% purples) {
    lumped_try_cols <- c(lumped_try_cols,'purple')
    #print(col)
  } else if (col %in% browns) {
    lumped_try_cols <- c(lumped_try_cols,'brown')
    #print(col)
  } else {
    lumped_try_cols <- c(lumped_try_cols,col)
  }
}
  
lumped_gpt_cols <- integer(0)
for (col in gpt_cols) {
  if (col %in% purples) {
    lumped_gpt_cols <- c(lumped_gpt_cols,'purple')
    #print(col)
  } else if (col %in% browns) {
    lumped_gpt_cols <- c(lumped_gpt_cols,'brown')
    #print(col)
  } else {
    lumped_gpt_cols <- c(lumped_gpt_cols,col)
  }
}

lumped_patrick <- integer(0)
for (col in patrick) {
  if (col %in% purples) {
    lumped_patrick <- c(lumped_patrick,'purple')
    #print(col)
  } else if (col %in% browns) {
    lumped_patrick <- c(lumped_patrick,'brown')
    #print(col)
  } else {
    lumped_patrick <- c(lumped_patrick,col)
  }
}


lumped_andrea <- integer(0)
for (col in andrea) {
  if (col %in% purples) {
    lumped_andrea <- c(lumped_andrea,'purple')
    #print(col)
  } else if (col %in% browns) {
    lumped_andrea <- c(lumped_andrea,'brown')
    #print(col)
  } else {
    lumped_andrea <- c(lumped_andrea,col)
  }
}

lumped_robin <- integer(0)
for (col in robin) {
  if (col %in% purples) {
    lumped_robin <- c(lumped_robin,'purple')
    #print(col)
  } else if (col %in% browns) {
    lumped_robin <- c(lumped_robin,'brown')
    #print(col)
  } else {
    lumped_robin <- c(lumped_robin,col)
  }
}

sum(lumped_patrick == lumped_andrea)
sum(lumped_patrick == lumped_robin)
sum(lumped_andrea == lumped_robin)

sum(lumped_patrick == lumped_try_cols)
sum(lumped_patrick == lumped_gpt_cols)
sum(lumped_andrea == lumped_try_cols)
sum(lumped_andrea == lumped_gpt_cols)
sum(lumped_robin == lumped_try_cols)
sum(lumped_robin == lumped_gpt_cols)


# Now redo to be extra permissive for TRY data
lumped_all_try <- integer(0)
for (color in trydata$color) {
  color <- tolower(color)
  if (color %in% purples) {
    lumped_all_try <- c(lumped_all_try,'purple')
    #print(col)
  } else if (color %in% browns) {
    lumped_all_try <- c(lumped_all_try,'brown')
    #print(col)
  } else {
    lumped_all_try <- c(lumped_all_try,color)
  }
}
trydata$lumped_color <- lumped_all_try

num_containing_patrick_label <- 0
num_containing_andrea_label <- 0
num_containing_robin_label <- 0
for (idx in 1:250) {
  subdf <- trydata[trydata$species_name == tolower(binoms[idx]),]
  if (lumped_patrick[idx] %in% subdf$lumped_color) {
    num_containing_patrick_label <- num_containing_patrick_label + 1
  }
  if (lumped_andrea[idx] %in% subdf$lumped_color) {
    num_containing_andrea_label <- num_containing_andrea_label + 1
  }
    if (lumped_robin[idx] %in% subdf$lumped_color) {
    num_containing_robin_label <- num_containing_robin_label + 1
  }
}
num_containing_patrick_label
num_containing_andrea_label
num_containing_robin_label
```

