# ventenata-resident-removal-study
Code and data to support the manuscript "How do plant communities and abiotic stress interact to influence a recent annual grass invader, Ventenata dubia?"

## Data

- "2020_vd_heights_by_vegtype":
Ventenata heights (measured in 2020) by plot. 5 heights were measured at each plot. 

- "biomass_data_2019_2020":
Ventenata biomass (measured in 2019 & 2020) and resident biomass (measured in 2020) by plot. Biomass are reported in grams.

### Vegetation type (vegtype) codes:
- ARRI = *Artemisia rigida* = scab-flat
- ARAR = *Artemisia arbuscula* = low sage-steppe
- SEEP = seep/ wet meadow

### Plot codes:
Vegetation type code + plot number (plotno) (1-5)

#### Plot_quad:
Vegetation type + plot number + treatment + subplot number (1-7 for seeded; 1-3 for unseeded control)

#### Treatment codes:
- NN= "no neighbors" = Cleared unseeded control
- CC = "community" = Uncleared unseeded control
- N. = "no neighbors" = Cleared + seed
- C. = "community" = Uncleared + seed

## Scripts

- "removal_treatment_vs_vegtype""
ventenata response to clearing and vegetation type & test of seeding vs. unseeded control

- "vedu_heights_and resident_biomass_by_vegtype"
ventenata heights and resident biomass response to vegetation type
