ventenata response to clearing and vegetation type & test of seeding
vs. unseeded control
================
Claire Tortorelli
August 18, 2021

read in data

``` r
bio <- read_csv(here("data", "biomass_data_2019_2020.csv"))
```

### Organize data for analysis

``` r
#add col for plot number, plot, and treatment: C = community, uncleared; N = no neighbors, cleared (apologies for the confusing codes)

bio$plotno <- factor(substr(bio$plot_quad,5,6))
bio$plot <- factor(substr(bio$plot_quad,1,6))
bio$trt <- factor(substr(bio$plot_quad,8,9))

#add factor for just seeded ("C." or "N.") or unseeded (no "CC" or "NN")
bio$trt2 <- recode(bio$trt, 'C.'='seeded', 'N.' = 'seeded', 'CC' = 'unseeded', 'NN'='unseeded')

#convert veg type to factor
bio$vegtype <- factor(substr(bio$plot_quad,1,4), levels = c("ARRI", "ARAR" , "SEEP"))

#remove ARAR03_N+1 from analysis - rodent activity overturned the entire quadrat.
bio <- bio[-which(bio$plot_quad == "ARAR03_N.1"),]

#Adjust for zeros using stahel method for log transformation
bio$stahel_other20.bio <- bio$resident20_g + quantile(bio$resident20_g, .25)^2/quantile(bio$resident20_g, .75) #there are 6 zeros, all in cleared subplots

# 21 zeros
bio$stahel_vd19biomass <- bio$vedu19_g + quantile(bio$vedu19_g, .25)^2/quantile(bio$vedu19_g, .75) 


bio$stahel_vd20biomass <- bio$vedu20_g + 0.01
#decided not to use the Stahel method for 2020 vedu biomass because it added 0.3g which was quite a bit more than the other biomass adjustments (closer to 0.01)

#explore data
bio %>% group_by(vegtype, trt) %>%
  summarise(mean = mean(stahel_vd20biomass))
```

### Fit model

Model VEDU biomass response to removal treatment by veg type split block
design receives random effect for block (plotno) and plot control for
2019 biomass (indicates potential seed bank and microsite suitability)

``` r
#model treatment 
mtrt <- lme(log(stahel_vd20biomass) ~ trt*vegtype + log(stahel_vd19biomass), random = ~ 1|plotno/plot, data = bio)

plot(mtrt) #residuals look symmetrical 
summary(mtrt)

mtrt.emm <- emmeans(mtrt, specs = pairwise ~ trt|vegtype, type = "response", adjust = "none")

summary(mtrt.emm) 
#no effect of clearing or vegetation type
#large difference between seeded and unseeded subplots
```

Model effect of seed addition vs. unseeded controls, not accounting for
clearing

``` r
#model just seeded vs. unseeded treatment 
mtrt2 <- lme(log(stahel_vd20biomass) ~ trt2*vegtype + log(stahel_vd19biomass), random = ~ 1|plotno/plot, data = bio)

plot(mtrt2) #residuals look good
summary(mtrt2)

mtrt2.emm <- emmeans(mtrt2, specs = pairwise ~ trt2, type = "response", adjist = "none")

summary(mtrt2.emm)
#strong effect of seeding compared to unseeded controls 233% increase in vedu biomass
```

### Plot results

Extract means for plotting

``` r
(means_table = mtrt.emm$emmeans %>%
summary(infer = c(TRUE, FALSE) ) %>%
as.data.frame() )

#rename levels for plotitng

means_table$vegtype  <- factor(means_table$vegtype, levels = c("ARRI", "ARAR", "SEEP"), labels = c("scabland", "low sage-steppe", "wet meadow"))



means_table$trt  <- factor(means_table$trt, levels = c("C.", "N.", "CC", "NN"), labels = c("uncleared + seed", "cleared + seed", "uncleared + unseeded", "cleared + unseeded"))
```

plot 2020 vedu biomass (post seeding) by treatment (cleared
vs. uncleared) and vegetation type

``` r
( g1 = ggplot(subset(means_table, trt %in% c("uncleared + seed", "cleared + seed")), aes(y = response, x = vegtype, group = trt, color = vegtype)) +
# Define stock as group this week as well as set x and y axes
geom_point(position = position_dodge(width = .75), size = 3 ) + # Add points, dodge by group
geom_errorbar(aes(ymin = lower.CL, ymax = upper.CL,
linetype = trt, width = 0.5), size = 1,
position = position_dodge(width = .75) ) + # Add errorbars, dodge by group
theme_bw(base_size = 14) +
labs(y = expression(italic("V. dubia")~" mean biomass (g)"))+
scale_color_manual(values = c("#A6761D", "#E6AB02", "#666666"))+
    guides(color = FALSE)+
    scale_y_continuous(breaks = c(2,4,6,8), limits = c(0,8.8))+
theme(legend.direction = "horizontal", # make legend horiz
legend.position = "bottom", # change legend position
panel.grid.minor = element_blank(),
axis.title.x=element_blank(),# Remove gridlines
panel.grid.major.x = element_blank() ))
```

plot unseeded control

``` r
# plot unseeded controls

( g2 = ggplot(subset(means_table, trt %in% c("uncleared + unseeded", "cleared + unseeded")), aes(y = response, x = vegtype, group = trt, color = vegtype)) +
# Define stock as group this week as well as set x and y axes
geom_point(position = position_dodge(width = .75), size = 3 ) + # Add points, dodge by group
geom_errorbar(aes(ymin = lower.CL, ymax = upper.CL,
linetype = trt, width = 0.5), size = 1,
position = position_dodge(width = .75) ) + # Add errorbars, dodge by group
theme_bw(base_size = 14) +
labs(y = expression(italic("V. dubia")~" mean biomass (g)"))+
scale_color_manual(values = c("#A6761D", "#E6AB02", "#666666"))+
    guides(color = FALSE)+
  scale_y_continuous(breaks = c(2,4,6,8), limits = c(0,8.8))+

theme(legend.direction = "horizontal", # make legend horiz
legend.position = "bottom", # change legend position
panel.grid.minor = element_blank(),
axis.title.x=element_blank(),# Remove gridlines
panel.grid.major.x = element_blank() ))
```

combine figures with cowplot

``` r
library(cowplot)

# svg(filename = "vedu_response_to_removal_trt.svg",
#     width = 8.5,
#     height = 4.5)

cowplot::plot_grid(g1, g2, labels = c("(a)", "(b)"))

# dev.off()
```
