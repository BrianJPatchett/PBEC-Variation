# Patchett et al., 2026

Peripheral blood eosinophil counts (PBECs) are widely used to phenotype asthma and guide biologic therapy. However, PBEC values exhibit substantial within-person variability, leading to frequent misclassification when clinical decisions rely on single or sparsely repeated measurements. Of course, there are many parallel examples in medical world and beyond. This repository provides reproducible R code for analyzing longitudinal data using complementary approaches that explicitly account for variability: RPP, PCOT, and Latent burden modeling. RPP (Repeated-Positive Probability) is the probability that a patient will ever exceed a clinically relevant threshold given they had initially presented above that threshold. PCOT (Probability of Crossing Over Threshold) is the probability that the next measurement exceeds a threshold, conditional on the current value. Finally, latent burden modeling utilizes mixed-effects models to estimate patient-specific eosinophil “set points” and quantify how reliably thresholds are met over time
These methods are designed to be interpretable to clinicians while remaining statistically principled

## Data format requirements

All analyses assume a wide-format matrix of repeated PBEC measurements:

 1. Rows: sequential measurements (e.g., clinic visits, lab draws)
 
 2. Columns: individual patients
  
 3. Cells: PBEC values (cells/µL)

Missing values are allowed only as trailing NAs (padding); internal gaps are not permitted.

```{r}
library(lme4)
library(tidyverse)

# Load example data
eos <- read.csv("eos_bs.csv")

# Run the primary latent burden model
source("figure4ab.R")

# Generate Figure 4a (≥150 cells/µL)
print(p4a)
```
This produces a model-implied probability curve showing how reliably patients meet the ≥150 PBEC threshold as a function of long-term eosinophil burden.

## Method 1: Repeated-Positive Probability (RPP)

The Repeated-Positive Probability (RPP) framework quantifies how often patients with a given initial peripheral blood eosinophil count (PBEC) eventually exceed a clinically relevant threshold at least once during subsequent follow-up. Operationally, patients are first stratified according to their initial PBEC into clinically interpretable bins. Within each bin, RPP is defined as the proportion of patients who exhibit at least one subsequent PBEC measurement that meets or exceeds a specified threshold (e.g., ≥150 or ≥300 cells/µL).

RPP is thus a cumulative, patient-level measure that captures long-term instability rather than short-term transitions. It is particularly useful for understanding whether an apparently “low” initial PBEC reliably excludes future eosinophilic activity, or whether patients with borderline values are likely to cross diagnostic or therapeutic thresholds at some point during longitudinal observation. Importantly, RPP does not model the timing or frequency of threshold crossings; it answers a simpler but clinically relevant question: given where a patient starts, how often do they ever cross the threshold at all?

## Method 2: Probability of Crossing Over Threshold (PCOT)
The Probability of Crossing Over Threshold (PCOT) approach focuses on short-term dynamics by estimating the conditional probability that the next PBEC measurement exceeds a predefined threshold, given the current PBEC value. Unlike RPP, which aggregates information across an entire follow-up period, PCOT operates at the level of consecutive measurement pairs and therefore reflects immediate transition risk.

In practice, PBEC values are discretized into bins, and all valid consecutive measurement pairs within individuals are identified. For each bin, PCOT is calculated as the proportion of transitions in which the subsequent measurement exceeds the threshold of interest. This formulation explicitly preserves within-subject correlation by restricting comparisons to adjacent observations from the same individual.

PCOT is well suited for scenarios in which clinicians are interested in near-term stability—for example, assessing whether a patient with a current PBEC just below a treatment cutoff is likely to exceed that cutoff at their next visit. While PCOT does not characterize long-term disease burden, it provides an intuitive, data-driven measure of short-horizon variability.

## Method 3: Latent Eosinophil Burden Modeling  
To move beyond threshold-based summaries, we model longitudinal PBEC measurements using a random-intercept mixed-effects framework that estimates a patient-specific latent eosinophil “set point.” PBEC values are log-transformed to stabilize variance, and each patient is assumed to fluctuate around an individual mean (latent burden), with residual variability capturing within-person fluctuations over time. This model separates stable between-patient differences from transient within-patient noise.

Using the fitted model, we derive the model-implied probability that a future PBEC measurement will exceed a clinically relevant threshold as a continuous function of latent eosinophil burden. These probabilities are not empirical proportions but analytic quantities derived from the assumed conditional distribution of future measurements. To improve clinical interpretability, latent burden is back-transformed to the original PBEC scale and presented as a smooth curve rather than discrete bins.

To account for heterogeneity in eosinophil stability across patients, we further estimate patient-specific residual variances using a two-stage shrinkage approach. This allows construction of variability envelopes that illustrate how exceedance probabilities differ between patients with relatively stable versus highly variable eosinophil profiles. In addition, parametric bootstrap resampling is used to quantify uncertainty in the population-level probability curves.

Together, this modeling approach provides a probabilistic interpretation of commonly used PBEC thresholds, demonstrating that even patients with high average eosinophil burden may not consistently meet stringent cutoffs at individual visits. Unlike RPP and PCOT, which summarize empirical behavior, the latent burden model offers a unified framework that links long-term burden, short-term variability, and threshold reliability

## Citation and Contact

If you use this code in academic work, please cite the accompanying manuscript. Questions, issues, and contributions are welcome via GitHub Issues
