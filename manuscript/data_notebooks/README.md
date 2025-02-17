# Data wrangling notebooks

## 1_merging_raw_iNat_exports.ipynb
Notebook for merging together the raw iNat export files.

## 2_gpt_vision_taxonphotos.ipynb
The notebook for the GPT-color labeling of the unique taxa from the iNat exports. Note: This is a messy notebook due to many small batches and server errors, etc. For a sleeker GPT demonstration look at notebook 2a below. 

## 2a_GPT_demo.ipynb
A pared-down demonstration of the GPT-labeling pipeline.

## 3_merge_GPT_taxon_colors_with_observations.ipynb
Notebook for merging the taxon-specific color labels from GPT with the iNaturalist observation data output from `1_merging_raw_iNat_exports.ipynb`

## 4_maxent_runs.ipynb
Notebook for automating Maxent runs.

## 5_TRY_data_cleaning.ipynb

Notebook for cleaning TRY data for downstream comparison.

## 6_validating_GPT_TRY.html

R Markdown for comparing our manually validated results to each other, GPT, and TRY. The validation CSVs are in: `flower_color_phenology/manuscript
/final_data/TRY_GPT_validation/