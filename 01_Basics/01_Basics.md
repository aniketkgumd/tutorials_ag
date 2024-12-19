## Basics of High Energy Physics Computing and Analysis

This tutorial introduces some basic concepts and tools you will need to know for doing research in High Energy Physics (HEP). We will cover data structures, ROOT, ROOT DataFrames, histograms, and plotting.

## Data Structures

As you heard in your lectures: particle colliders collide particles (sometimes, bunches of particles). Detectors take "photographs" of these collisions: from various parts of the detector, we get information about the particles produced in the collision, such as their energy, momentum, charge, etc.. Many layers of algorithms combine this information to "reconstruct" the particles produced in the collision. This are the data we work with. 

Each collision is dubbed an "event", and our data is organized in sets of "events". Each event is a collection of particles, and for each particle, we store various information about its reconstructed and measured properties. You can visualize the basic information we read off from out detector for each event as,

```
  Event 0
  |--- Particle 0
  |------- Energy: 10 GeV
  |------- Momentum: (1, 2, 3) GeV
  |------- Charge: 1
  |------- ...
  |--- Particle 1
  |------- Energy: 20 GeV
  |------- Momentum: (4, 5, 6) GeV
  |------- Charge: -1
  |------- ...
  |--- ...
  Event 1
  |--- Particle 0
  |------- Energy: 30 GeV
  |------- Momentum: (7, 8, 9) GeV
  |------- Charge: 1
  |------- ...
  ...
```

Data are stored in `.root` files, and are organized in a "tree" structure. This structure is a bit different than what is shown above.

```
  Events
  |--- Energies [ [10, 20], [30, ...], ... ] 
  |--- Momenta [ [(1, 2, 3), (4, 5, 6)], [(7, 8, 9), ...], ... ]
  |--- Charges [ [1, -1], [1, ...], ... ]
  |--- ...
```

Namely, each 'property' is stored into a separate 'branch' of the tree as arrays, with the first element of the array corresponding to the first event, the second element to the second event, and so on.
This is a more efficient way to store data.
Let's open a `.root` file and take a look at it. Start up root with,

``` sh
root
```

Now, in the root command line, open a `.root` file with,

``` cpp
root [0] TFile *file = TFile::Open("/ceph/submit/data/group/cms/store/fccee/samples/winter2023/CLD_FullSim/wzp6_ee_mumuH_ecm240/mumuH_rec_16610_191.root");
root [1] file->ls()
TFile**         /ceph/submit/data/group/cms/store/fccee/samples/winter2023/CLD_FullSim/wzp6_ee_mumuH_ecm240/mumuH_rec_16610_191.root
 TFile*         /ceph/submit/data/group/cms/store/fccee/samples/winter2023/CLD_FullSim/wzp6_ee_mumuH_ecm240/mumuH_rec_16610_191.root
  KEY: TTree    events;38       events data tree [current cycle]
  KEY: TTree    events;37       events data tree [backup cycle]
  KEY: TTree    configuration_metadata;1        configuration_metadata data tree
  KEY: TTree    metadata;1      metadata data tree
  KEY: TTree    podio_metadata;1        metadata tree for podio I/O functionality
root [2] TTree *tree = (TTree*)file->Get("events");
root [3] tree->Print();
******************************************************************************
*Tree    :events    : events data tree                                       *
*Entries :     1000 : Total =      2170688008 bytes  File  Size = 1258710237 *
*        :          : Tree compression factor =   1.72                       *
******************************************************************************
*Br    0 :BuildUpVertices : Int_t BuildUpVertices_                           *
*Entries :     1000 : Total  Size=      26008 bytes  File Size  =       9507 *
*Baskets :       38 : Basket Size=      32000 bytes  Compression=   1.21     *
*............................................................................*

...
```

As you see, the `.root` file contains a `TTree` called `events`. This tree contains 1000 entries, or events, and each entry contains many branches. For example, we see one called `BuildUpVertices`. This branch contains an array of integers, which is the number of vertices for each event.

> *Exercise:* Do this yourself.

> *Hint:* did you set up your environment?

## ROOT and Analysis Concepts

ROOT is CERN's data analysis framework built for C++ and Python (PyROOT), and provides the backbone for the code you will be running. CERN provides a [good primer](https://root.cern.ch/root/htmldoc/guides/primer/ROOTPrimer.html#root-in-python), which covers all the basics.

A typical analysis code will **loop through the various events**, and perform analysis on the objects in each event. **Selections (or filters, or vetos)** are criteria that are applied to select particular objects, or events. Results are either skimmed and processed versions of the original events, or histograms containing information about rates of certain processes.

Here is a simple script to read in a `.root` file, loop through 10 events,print information about the MC particles of these events, apply a selection on the MC particle energy, and fill a histogram with the energies of the particles.

``` python
import ROOT

# Open the file
file = ROOT.TFile("/ceph/submit/data/group/cms/store/fccee/samples/winter2023/CLD_FullSim/wzp6_ee_mumuH_ecm240/mumuH_rec_16610_191.root")

# Get the tree of events
tree = file.Get("events")

# Create a histogram to store the energies
h = ROOT.TH1D("h", "Energy [GeV]", 100, 0, 1000)

# for loop over the first 10 events, and print information about the MC particles
for i in range(10):
    
    print("Processing event", i)
    tree.GetEntry(i)

    # get the MC particles for this event
    MCParticles = tree.MCParticles
    print("Number of MC particles:", len(MCParticles))

    # loop over all particles in the event
    for j in range(len(MCParticles)):

        # print their information
        print("MC Particle", j)
        print("Momentum (px, py, pz):", MCParticles[j].momentum.x, MCParticles[j].momentum.y, MCParticles[j].momentum.z)
        print("Charge:", MCParticles[j].charge)
        print("PDG ID:", MCParticles[j].PDG)

        # calculate the energy
        energy = (MCParticles[j].momentum.x**2 + MCParticles[j].momentum.y**2 + MCParticles[j].momentum.z**2)**0.5 + MCParticles[j].mass**2
        print("Energy:", energy)

        # if the particle has more than 10 GeV of energy, fill the histogram
        if energy > 10:
            h.Fill(energy)

# draw the histogram, this is done using a Canvas
c = ROOT.TCanvas()
h.Draw()

# label the axes
h.GetXaxis().SetTitle("Energy [GeV]")
h.GetYaxis().SetTitle("Number of particles")

# draw the canvas
c.Draw()

# save the plot to a file
c.SaveAs("energy.png")

# save the histogram to a .root file
file_out = ROOT.TFile("output.root", "RECREATE")
h.Write()
file_out.Close()

print("All done!")
```

In pseudocode, this script does the following,

```
file = open .root file
tree = get the tree from the file

for event in first 10 events:

    for particle in MC particles:
        
        print information about the particle
        
        calculate the energy of the particle

        if the energy is greater than 10 GeV:
            fill the histogram with the energy
```


> *Exercise:* Copy this script into a python file and run it with,
  ``` sh 
  python script.py
  ```

Two files will be created: `energy.png` and `output.root`. The first is a plot of the canvas, with the histogram of the energies, which is great for visualizing. The second is a `.root` file containing the information stored in the histogram, which is useful for storing, and reading back into a script for further analysis.

> *Exercise:* Before moving on to learning more about histograms, modify the script to identify the most energetic muon and anti-muon in each event, and plot their combined invariant mass. How many entries do you expect to see in the histogram? How many do you see? Why is this the case? Try running with more events now. Adjust the histogram range and set the y-axis to a log scale to see the peak better. What is this peak? Show off this peak to your friends. You might've won a Nobel prize if you had done this 50 years ago.

> *Hint:* Use the [PDG ID](https://pdg.lbl.gov/2020/reviews/rpp2020-rev-monte-carlo-numbering.pdf) to identify muons. Try using TLorentzVector and the naive formula to calculate the invariant mass, and compare.

## Histograms and Plotting

### Philosophical Detour

Nearly all collider particle physics experiments are, at their core, counting experiments.

The reason is that, for the most part, the observables that we can calculate from Quantum Field Theory (QFT) are cross-sections, which correspond to probabilities of certain physics processes happening.

As an example, at the FCC-ee, we will collide electrons. QFT gives us predictions for the rates for processes like $e^+ e^- \to \mu^+ \mu^-$, $e^+ e^- \to \tau^+ \tau^-$, etc.
These rates, i.e. how often you expect to get two muons out of the collisions, are proportional to the cross-sections for these processes: $\sigma(e^+ e^- \to \mu^+ \mu^-)$.
Measuring these cross sections allows us to compare our theory with experiment.

Cross sections are often parametrized as functions of the outgoing particle momenta or angles: you will often see things like $d\sigma(e^+ e^- \to \mu^+ \mu^-)/d\cos\theta dp$.
In simple words, this is a prediction for how often you expect to see a muon produced with a particular momentum $p$ and angle $\theta$ from $ee$ collisions with a specific center-of-mass energy.
These parametrizations can give us access to more information about the underlying structure of the theory, as you will learn about.

Histograms measure the *frequency* of events in *bins* of certain *variables(s)*.
They provide powerful tools for visualizing and understanding the data.

In the example script you wrote earlier, your histogram was counting how many events produced had a muon and an antimuon, each with at least 10 GeV of energy, parametrized in the invariant mass of the muon-antimuon pair.
It was clear when you plotted the histogram that there was a peak!

When comparing data to theory, histograms can also be used to compare the *shapes* and *rates* of the distributions, using statistical fits.

### Aside: Why and Why Not Historgams? Unbinned versus Binned Approaches

A valid question could be: why do you need to bin events into histograms to compare theory and data? Binning is a simplification, a smoothing of the underlying distribution, which causes you to lose some sensitivity. So why do it?

It makes more sense to think about it in reverse: what allows some experiments like CMS to use histograms?
Answer: you look at ridiculous amount of data which not only allows -- but *requires* -- simplification through binning.
Doing an unbinned fit on CMS data would probably mean the sun explodes before your fit converges.

However, this is not the case in general! For some analyses (e.g. flavour physics) where event counts are 'small' (< 10000 events), histograms are not useful ways to perform fits to theory, for some of the following reasons,

- As we said already: binning is a smoothing of the underlying distribution, which can cause you to lose sensitivity to certain features. It is theoretically better to not do it, if you can afford to. With few events to fit, this is possible.
- Binning little data is not practical. Binned uncertainties can be larger and only approach the unbinned ones for large number of data points.
- The uncertainties on each bin count are Poissonian (because we're counting). And that's only symmetric for large numbers, so we'd have to have asymmetric constraints on each bin, which is hard to deal with.


### In Practice

As you saw in the example script, we can make histograms by looping over the collision events, and counting how many events fall into each bin for a specific variable.
To do so, we used ROOT's `TH1D`, which is a 1D histogram; are also `TH2D`, `TH3D`, etc., for 2D and 3D histograms.
You can also use `numpy`, `pandas`, `hist` (suggested), and other python libraries to make, manipulate, and plot histograms.

In our example, we stored the histogram in a `.root` file.
Let's open this up, and read in the histogram.
There are existing and strong personal preferences for which tools you like to use to do this type of analysis: we will introduce you to using python with `hist` and `maplotlib` to work with histograms and plot them, but you are free to play around with other tools you may be more familiar with!

The python code walks you through the basics of reading in a histogram using `uproot`, and plotting it using `matplotlib` and `mplhep`.

``` python
import uproot

# open the .root file containing histograms (change this to read the file you created earlier!)
file = uproot.open("output.root")

# print the contents of the file
print(file.keys())

# read in the histogram
h = file["h"]

# this shold be a TH1D object, since we created it with ROOT
print(h)

import hist

# let's convert it to hist
h = h.to_hist()
print(h)

import matplotlib.pyplot as plt
import mplhep as hep 

# tjese set the default style of the plotting libraries to the CMS style
hep.style.use("CMS")
plt.style.use(hep.style.CMS)

# initialize a figure and axis using matplotlib
fig = plt.figure()
ax = fig.subplots()

# use mplhep to plot the histogram on the axis, add a nice label
hep.histplot(h, label="$e^+e^-\\to\mu^+\mu^-$", ax=ax)

# add some labels and a legend -- this is very very important!!
ax.set_ylabel("Events")
ax.legend(loc=(1.01, 0), fontsize='x-small')
ax.set_yscale("log")
ax.set_xlabel("$E$ [GeV]")

# the finishing touch, add the experiment label!
# for now setting the luminosity to 1 as an example, since we didn't do any normalization
hep.label.exp_label(exp="FCC-ee", ax=ax, lumi=1, data=False, com=240)

# let's save this to a file
# (the bbox_inches="tight" makes sure the labels are not cut off)
fig.savefig("pretty_energy.pdf", bbox_inches="tight")
```

> *Exercise*: Run this as a script or a notebook.

> *Exercise*: Plot the histogram of the invariant mass. Make sure to select a nice x-range to center around the area of interest. Add a dashed black line at the value of $m_Z$, and put it in the legend.

# Conclusion

In this tutorial, you learned some of the most fundamental tools and concepts used in everyday high energy physics research: data structures, analysis frameworks, histograms, and plotting.
Further tutorials will dive deeper into these topics, and build up to how real analyses are performed.