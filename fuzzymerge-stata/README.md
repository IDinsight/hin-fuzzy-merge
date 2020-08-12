Stata code for merging two datasets containing columns with Hindi transliterated text/names.

# Overview

This code was developed to match names in a household census with entries voter rolls for the corresponding areas in four Hindi-speaking states in northern India. It is intended as an example of how to fuzzy match datasets with Hindi transliterated names. Other use cases will require changes to this code, including updates to the list of transliteration standardizations and to the list of other variables that are used to make matches.

This code is motivated by the fact that Hindi names written in Devanagari script are not transliterated in a consistent way to Latin script. Although fuzzy matching programs exist, most are optimized for text originally written in Latin script, and so they perform poorly when applied to Hindi transliterated names. We also found that match rates could be improved substantially by taking a stepwise approach, starting with the most certain matches and progressively loosening restrictions. False matches in fuzzy matching algorithms propagate: an early false match that incorrectly removes an individual from the match pool leads the algorithm to make false matches with other individuals in later steps. By completing more certain matches before moving onto less certain matches, we found that our stepwise algorithm (comprised of 125 sequential merges) reduced false match rates more than running a fuzzy match program a single time.

# Directory Structure

- do
- raw data
-- census data
-- voter roll data
- clean data
-- intermediate merge data
-- final matched data

# Usage

This code can be run by setting the applicable file paths in the master .do file and executing the master .do file, which calls the helper .do files in a specified order.

# Files
- “_master matching.do”: Sets file paths and lists helper .do files in specified order. Executing this file will call all of the helper .do files.
- “0-prep voter and census data.do”: Prepares the two raw datasets (household census & voter rolls) for matching. Calls helper file “0i-transliteration.do” to generate name variants. Drops duplicate entries. Drops observations outside of the study range. Generates variables to be used for matching.
- “0i-transliteration.do”: Generates 5 name variants to original name, applying progressively more aggressive substitutions. For instance, the first name variant standardizes all instances of “singh”, which is often interchangeably spelled as “sinh” and “simha” and misspelled as “sing”. The first name variant also removes instances of “devi”, “shri”, “let”, “sev”, and “bano”, which are inconsistently added as prefixes/suffixes to male or female names. Since different data sources have their own spelling peculiarities and typos, we strongly recommend customizing this list of transliterations to each dataset.
- “1-match voter ids.do”: Matches individuals who provided a valid voter ID with voter roll entries and removes them from the match pool. Fixes typos in voter IDs recorded in census.
- “2-match exact name and relation name.do”: Matches individual’s name and their relative’s name exactly on the original name or one of the 5 variants. Loops over variants, starting with variant with least aggressive substitutions and progressing to variant with most aggressive substitutions. Appends all matches and removes them from the match pool.
- “3-match exact name and fuzzy relation name.do”: Exactly matches individual’s name (or variant) and fuzzy matches their relative’s. This step comes before “5-match fuzzy name and exact relation name.do” to give preference to exact matches on the individual’s name over exact matches on their relative’s name. This .do file loops over the name variants as well as two thresholds for fuzzy matching (first a stricter threshold, requiring fuzzy matches to be very similar, then a more relaxed threshold). As with other fuzzy matching steps in this repository, this .do file uses Stata’s ‘reclink’ command to identify best matches and score the quality of each match. The thresholds correspond to reclink scores; the stricter threshold requires the score to be greater than 0.9, and the looser threshold requires the score to be greater than 0.8. We also drop fuzzy matches with differences in length that are greater than 3 characters, since reclink underweights length differences for our fuzzy match application.
- “4-match exact name and flipped fuzzy relation name.do”: Same as “3-match exact name and fuzzy relation name.do” but flips the relative association (lists father as spouse and vice versa), since voter rolls occasionally make errors in applying the correct label to relatives’ names.
- “5-match fuzzy name and exact relation name.do”: Same as “3-match exact name and fuzzy relation name.do” but fuzzy matches individual’s name and exact matches their relative’s name.
- “6-match exact name, gender, fuzzy age.do”: Relaxes the restriction that relative’s name needs to match, but adds restrictions that gender & age (within five years) must match.
- “7-match fuzzy name and fuzzy relation name.do”: Same as “3-match exact name and fuzzy relation name.do” but fuzzy matches both individual’s name and their relative’s name.
- “8-match within hh fuzzy name and fuzzy age.do”: Loops through every household in the census (defining household as sharing a common kitchen) and every household in voter rolls (defining household as sharing the same house number within a PS-part) that already have one matched voter, and attempts to match other voters in those paired households based on fuzzy name, gender, and age within 15 years.
- “98-manual matches.do”: Inspects remaining non-matches in census and voter rolls and adds any remaining apparent matches. Requires the user to manually inspect the data and flag apparent matches.
- “99-append and clean.do”: Appends the matched data, merges with the full census and voter rolls, and identifies which observations matched and which did not.