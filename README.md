# Patchett et al., 2024

This paper deals with variation of the peripheral blood eosinophil count. In the paper, we develop new intra-individual metrics called the Probability of Threshold Stability (PTS) and Probability of Change Over Threshold (PCOT). These were inspired by the well-established Probability of Acute Change, which is used regularly in the psychometric literature. In case anyone is interested in replicating our findings or using PCOT or PTS in their own research, I have a brief tutorial on how we did the analysis in the paper. This is geared towards novices to make it accessible for everyone.

## Install and load the required packages

```{r}
p <- c("tidyverse", "egg", "ggh4x")

if (!require(p)) install.packages(p)
lapply(p, library, c = T)
```
Now that we've loaded the packages, we need to make sure our data is in the wide format. This means each column is a patient, and each row is their nth eosinophil count (or whatever measurement you're using). Each column should have the header of x_1, x_2, x_3, ... x_n. If you're comfortable with R code, you can leave the data in long format and just remove the steps that deal with convert from wide to long.

## Tutorial for Probability of Threshold Stability
Loading the data
```{r}
eos <- read.csv(file = "eos.csv", header = TRUE)

except_first <- eos[-c(1), ] 
```
Note that in gather(PID, value, x_1:x_154), you'll have to update the x_154 to be the max number of patients you have.
```{r}
df_one <- except_first %>%
  gather(PID, value, x_1:x_154)%>%
  group_by(PID) %>%
  na.omit %>% 
  summarise(Highest_Value = max(value))

first <- eos[-c(2:nrow(eos)), ] 
```
Same goes here, adjust x_154 to your highest patient number.
```{r}
### Convert from wide to long ###
long <- first %>%
  pivot_longer(
    cols = `x_1`:`x_154`,
    names_to = "PID",
    values_to = "eos"
  )

### merge by PID ###
total <- merge(long, df_one,by="PID")
```
Here, you'll need to adjust "149" and "299" to be the thresholds of interest for your project. Same goes for the labels_two and mutate arguments. Adjust those to reflect the intervals of interest.
```{r}
total$low <- 0
total$low <- ifelse(total$Highest_Value > 149, 1, total$low)
total$high <- 0
total$high <- ifelse(total$Highest_Value > 299, 1, total$high)

labels_two <- c("0-49", "50-99", "100-149", "150-199", "200-249", 
                "250-299", "300-349", "350-399", "400+")

cat_low <- total %>% mutate(new_bin = cut(eos, breaks=c(-1, 50, 100, 
                                                          150, 200,
                                                          250, 300,
                                                          350, 400, Inf),
                                             labels = labels_two))

final_set_low <- cat_low %>% 
  group_by(new_bin) %>% 
  summarize(mean(low))
final_set_high <- cat_low %>% 
  group_by(new_bin) %>% 
  summarize(mean(high))

colnames(final_set_low) <- c("new_bin", "Any_low")
colnames(final_set_high) <- c("new_bin", "Any_high")

p_one <- ggplot(data = final_set_low, mapping = aes(x=new_bin, y=Any_low)) +
  geom_point() +
  theme_classic() +
  ylab("Probability that at least one subsequent reading is ≥150") +
  xlab("Initial PBEC Count") +
  scale_x_discrete(guide = guide_axis(angle = 45)) +
  labs(tag = "2a)") +
  scale_y_continuous(limits = c(0,1))

p_two <- ggplot(data = final_set_high, mapping = aes(x=new_bin, y=Any_high)) +
  geom_point() +
  theme_classic() +
  ylab("Probability that at least one subsequent reading is ≥300") +
  xlab("Initial PBEC Count") +
  scale_x_discrete(guide = guide_axis(angle = 45)) +
  labs(tag = "2b)") +
  scale_y_continuous(limits = c(0,1))

grid.arrange(p_one, p_two, nrow = 1)
```

## Conclusion
I hope this helps you in your project. Please remember to cite the paper if you plan on using this approach. I additionally wrote some code that uses resampling to see how the values change with random initial values.  If you have any questions, email me.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.
