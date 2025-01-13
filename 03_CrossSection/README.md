# $Z\rightarrow\mu\mu$ Cross-Section at the Z-Pole

In this tutorial, we will explore the fundamentals to measure the cross-section of the $Z \rightarrow \mu\mu$ process at the Z-pole. The analysis strategy and selection is based on previous measurements done at the CERN [Large Electron Positron collider (LEP)](https://en.wikipedia.org/wiki/Large_Electron%E2%80%93Positron_Collider). You can find more information in [this paper](https://arxiv.org/abs/hep-ex/0002046) about the measurement done by the [L3 Collaboration](https://en.wikipedia.org/wiki/L3_experiment), one of the 4 major experiments at LEP.






## Introduction and concepts

### Cross-Section and Luminosity

A comprehensive summary of cross-section and luminosity definitions can be found [here](https://cds.cern.ch/record/2800578/files/Cross%20Section%20and%20Luminosity%20Physics%20Cheat%20Sheet.pdf). To summarize:

- **Cross-section**: A characteristic of the physical process, measured in units of picobarn (pb), where $1 \text{ barn} = 10^{-24} \text{ cm}^2$.
- **Instantaneous luminosity** ($\mathcal{L}$): Represents the density of particles colliding in the accelerator. Units: $\text{pb}^{-1}\text{s}^{-1}$ or $\text{cm}^{-2}\text{s}^{-1}$.
- **Integrated luminosity** ($\mathcal{L}_{\text{int}}$): The instantaneous luminosity integrated over time, with units of $\text{pb}^{-1}$. Typically, "luminosity" refers to integrated luminosity.

For example, at the Z-pole (91 GeV), the FCCee accelerator will achieve a luminosity of $5\cdot 10^{36}~\text{cm}^{-2}\text{s}^{-1}$ and an integrated luminosity of approximately 100 $\text{ab}^{-1}$ (attobarn), corresponding to about two years of operation. In contrast, at LEP, the instantaneous and integrated luminosities were much lower, with the latter amounting to 44.84 $\text{pb}^{-1}$.

We calculate the $Z\rightarrow\mu\mu$ production rate $R$ (events per second) using a cross-section of 1717.85 pb (obtained from theoretical calulations and confirmed with measurements at LEP):

```math
R = \mathcal{L} \times \sigma = 5\cdot 10^{36} \times 1717.85\cdot 10^{-24} = 8589.25.
```

Thus, at FCCee, approximately 8500 $Z\rightarrow\mu\mu$ events are produced every second! The total number of events $n_{\text{tot}}$ can be computed using the integrated luminosity:

```math
n_{\text{tot}} = \mathcal{L}_{\text{int}} \times \sigma = 100\cdot 10^{6} \times 1717.85 = 1.72\cdot 10^{9}.
```

At LEP, with an integrated luminosity of 44.84 $\text{pb}^{-1}$, the total number of events is:

```math
n_{\text{tot}} = \mathcal{L}_{\text{int}} \times \sigma = 44.84 \times 1717.85 = 77028.394.
```

Clearly, the FCCee will produce far more $Z\rightarrow\mu\mu$ events than LEP!


### Monte Carlo Normalization

In Monte Carlo methods, millions of events are generated to simulate real collisions. These simulations include detailed descriptions of fundamental interactions, decays (as per the Standard Model), detector characteristics, and particle reconstruction. Ideally, the simulations should mirror real data as closely as possible.

To ensure meaningful comparisons with actual data, the number of simulated events should ideally match or exceed the number of real collisions. However, this is often impractical due to computational and storage limitations, as simulated events usually contain far more detailed information than real data, leading to significant disk space usage.

Since the number of Monte Carlo events ($n_{\text{events}}$) is typically smaller than the number of real collisions, normalization is necessary. This involves accounting for:

- The cross-section of the simulated process ($\sigma$),
- The number of generated events ($n_{\text{events}}$), and
- The total integrated luminosity ($\mathcal{L}_{\text{int}}$).

Normalization ensures that histogram integrals reflect these aspects accurately. The normalization weight is computed as:

```math
w = \frac{\mathcal{L}_{\text{int}} \times \sigma}{n_{\text{events}}}.
```



## Run the Analysis: Event Normalization and Plots

To perform the analysis, set up the environment and navigate to the `z_mumu_xsec` tutorial directory:

```shell
cd mit-fcc/tutorials/03_CrossSection
source /work/submit/jaeyserm/software/FCCAnalyses/setup.sh
```

The samples, analysis logic, cuts, and histograms are defined in the `z_mumu_xsec.py` file. For this analysis, the signal sample is $Z \rightarrow \mu\mu$ (`wzp6_ee_mumu_ecm91p2`), and the backgrounds are `wzp6_ee_tautau_ecm91p2` and `p8_ee_gaga_mumu_ecm91p2`. The terminology for the sample names is as follows: `wzp6` means the Whizard event generator coupled to Pythia 6 for the hadronisation; `ee` means electron-positron initial state; `mumu` means the acual process, i.e. 2 muons (see lectures for the different processes and Feynman diagrams that can occur at the Z pole) and `ecm91p2` is the center-of-mass energy of 91.2 GeV (the Z pole). `p8` means the Pythia 8 event generator and hadronisation. Other event generators are also available such as `KKMCee` or `MadGraph`. Event generators differ by their theoretical implementation of the physical process or different numerical methods to evaluate the mathematical expressions. A full list of samples generated for FCC can be found [here](https://submit.mit.edu/~jaeyserm/fcc/samples/ee_FastSim_winter2023_IDEA.html).

We'll adopt the following cuts, inspired by the reference paper above:

- **Cut 1**: Select events with at least one muon.
- **Cut 2**: Select events with at least two muons.
- **Cut 3**: Select events with exactly two muons of opposite charge.
- **Cut 4**: Require the highest momentum muon to have $p > 0.6$ of the beam energy.

These cuts are implemented in the file ```z_mumu_xsec.py```. Inspect the file and try to understand where the cuts are implemented.

Run the analysis by looping over all events for all processes:

```shell
fccanalysis run z_mumu_xsec.py
```

This may take some time, as all samples are loaded, and events are processed. Once complete, a ROOT file is generated for each process in the `output` directory, containing histograms. 

We now proceed to plot the different histograms and try to understand the physics behind it. There are two ways for plotting: 1) a quick plotting tool in `ROOT` or 2) produce your own plots using `matplotlib` and `mpl4hep` (see tutorial 2). This first option is generally used to quickly produce some plots and check the outcome; but we recommed to be familiar with the second option to produce your own pretty plots.

To plot using `ROOT`, open the `plots_root.py` file and modify the `outdir` to point to your web area (and make this directory), then execute:

```shell
fccanalysis plots plots_root.py
```

This generates several plots in the specified `outdir`. Open the plots in your web browser and have a look, and see how they are configured in the `plots_root.py` file. To plot them using the second option, an example is given in the `plots_mpl4hep.py` file (again, modify the `outDir` in the file to point to your web area):

```shell
python plots_mpl4hep.py
```

> *Exercise:* Expand the `plots_mpl4hep.py` script to plot the `muon_max_p_norm` histogram.

Inspect the `cutFlow` histogram, which shows stacked event yields for all processes across different cuts. After the final cut, there are approximately $10^8$ events (signal-dominated, with backgrounds included). However, this count does not align with our earlier calculation. Why?

The discrepancy arises because event normalization has not been applied. The current counts are bare (non-normalized) yields. Event normalization involves considering the cross-section (tabulated as metadata with the sample), the number of events processed, and the integrated luminosity (set by the user, e.g., 44.84 $\text{pb}^{-1}$ for LEP). Use the following script to inspect event normalization:

```shell
python analysis.py --yields
```

This script prints both bare and normalized yields. For the signal, the normalized yield is approximately 71,744.33 events, which is closer to our earlier calculation (77,028.394).

Normalization can be automated during the analysis. Modify these parameters in `z_mumu_xsec.py`:

```python
doScale = True
intLumi = 44.84 # Integrated luminosity in pb
```

Re-run the analysis and regenerate the plots:

```shell
fccanalysis run z_mumu_xsec.py
fccanalysis plots plots_root.py
python plots_mpl4hep.py
```

Adjust the y-axis scale of the `cutFlow` plot to 1e0–10e8 for better visualization. With proper normalization, the plots will now make more sense. Study each plot, compare them with the LEP paper, and evaluate whether the cuts are appropriate.

> *Exercise:* Run the analysis for the FCC-ee luminosity of $100\text{ ab}^{-1}$ and adapt the plot labels accordingly.

> *Exercise:* Plot the momentum distribution of each muon separately, before the 4th cut.



## Calculation of Acceptance

Let’s revisit the previous numbers: the calculated (77,028.394) and simulated (71,744.33) event yields are similar to first order, but there is still a residual difference of 5,284.06 events. This discrepancy arises due to acceptance effects.

Monte Carlo simulations generate physics processes over the entire phase space, encompassing all possible angles and energies. However, real detectors have limitations:

1. **Geometrical Acceptance**: Certain regions, especially near the beam pipe (the “forward region”), cannot be instrumented with detectors due to physical constraints. Additionally, mechanical design may create “dead zones” where detection is impossible. The geometrical acceptance is the ratio of the total active coverage of the detector to the full spherical coverage ($4\pi$).

2. **Kinematical Acceptance**: Detector limitations and event selection cuts further reduce the measurable events. For instance, low-energy particles may be undetectable or lost in the detector.

Together, these limitations define the **fiducial volume**, the effective phase space where events can be detected.

### Defining Acceptance

When measuring the cross-section, the goal is to determine the generation-level cross-section, omitting acceptance effects. To achieve this, we rely on Monte Carlo simulations, which incorporate detailed detector and kinematical descriptions. Cuts applied by physicists further refine the signal phase space.

Acceptance ($A$) is defined as:

$$
A = \frac{n_{\text{sel}}}{n_{\text{total}}},
$$

where:

- $n_{\text{sel}}$: Total number of selected events (not objects or muons!) after all selections and detector simulation.
- $n_{\text{total}}$: Total number of generated events (measured before any cuts).

In our analysis, the acceptance can be derived from the cutFlow table:

- The last bin represents $n_{\text{sel}}$ (total selected events).
- The first bin represents $n_{\text{total}}$ (total generated events).

To calculate the acceptance, use the following script:

```shell
python analysis.py --acceptance
```

The calculated acceptance is `0.931`. This indicates that 93.1% of the total events are selected after detector effects and event selection. Scaling back the 71744.33 selected events by the acceptance yields:

```math
\text{Adjusted events} = \frac{71,744.33}{0.931} = 77061.57895,
```

which matches the initial number of generated events (modulo small numerical roundings).

The acceptance is calculated specifically for the signal process and is independent of the cross-section and luminosity. It depends, however, on the number of events generated: larger samples yield more precise acceptance calculations.

### Tasks

To investigate the dependence of acceptance on the sample size:

1. Calculate the acceptance for different fractions of the signal sample (e.g., from 0.1 to 1). The fraction should be modified in `z_mumu_xsec.py`.
2. Plot acceptance as a function of this fraction.
3. Calculate the statistical error on the acceptance and include it as error bars in the plot.

These exercises will help quantify the precision of the acceptance calculation and its dependence on sample size.



## Calculation of the Cross-Section

With all the elements introduced so far, we can now measure the cross-section for the $Z \rightarrow \mu\mu$ process. The master formula is:

```math
\sigma = \frac{n_{\text{obs}} - n_{\text{bkg}}}{\epsilon \times A \times \mathcal{L}_{\text{int}}},
```

where:

- $\sigma$: Cross-section (pb).
- $n_{\text{obs}}$: Observed number of events in data (not available yet; we use simulation for now).
- $n_{\text{bkg}}$: Number of background events, estimated from simulation.
- $\epsilon$: Efficiency differences between data and Monte Carlo.
- $A$: Acceptance, estimated from simulation.
- $\mathcal{L}_{\text{int}}$: Integrated luminosity.

The observed number of events, $n_{\text{obs}}$, is the count of events in data after applying all selections and cuts. Since no real data is currently available, we use the total number of Monte Carlo events (signal + background) for $n_{\text{obs}}$. The background events, $n_{\text{bkg}}$, are derived similarly but only for background processes.

### Selection Efficiency ($\epsilon$)

The selection efficiency $\epsilon$ is a concept we haven’t introduced yet but is only of importance when we have data. , as it accounts for differences between data and Monte Carlo. During data taking, detector inefficiencies, time-dependent effects, and other small discrepancies arise, which simulations cannot fully model. Scale factors are applied to Monte Carlo to align it with data, encapsulated in the efficiency $\epsilon$. For now, we set $\epsilon = 1$ as a placeholder.

A nice example of time-depedent effects on the accelerator and experiment at LEP are the moon tides (that slightly deformed the tunnel, altering beam orbits) and high speed trains that caused disturbances in accelerator cavities. For an interesting read, refer to this article: [The particle now leaving platform 4](https://www.newscientist.com/article/mg14820060-300-the-particle-now-leaving-platform-4/).

### Tasks

Using the formula and the provided scripts, compute the cross-section by following these steps:

1. **Compute the cross-section for LEP luminosity**: analyze the result and understand its number.
2. **Compute the cross-section for FCC luminosity**: compare this result with the LEP cross-section.
3. **Derive the formula for the statistical uncertainty on the cross-section**: explore how uncertainties propagate in the calculation.
