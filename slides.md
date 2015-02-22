% title: Engineering a full python stack for biophysical computation
% author: Kyle A. Beauchamp
% favicon: figures/membrane.png

---
title: erooM's Law for R&D Efficiency
subtitle: Producing a drug costs $2B and 15 years, with 95% fail rate

<center>
<img height=400 src=figures/eroom.png />
</center>


<footer class="source"> 
Scannell, 2012
</footer>


---
title: How can computers help design drugs?

<center>
<img height=475 src=figures/cruise.png />
</center>

<footer class="source"> 
Figure credit: @jchodera, Tom Cruise
</footer>


---
title: Predictive atomistic models
subtitle: Designing the binding, specificity, and signaling of chemotherapeutics

<center>
<img height=300 src=figures/multiscale-cartoon.png />
<video id="sampleMovie" src="movies/shaw-dasatanib-2.mov" loop=\"true\ autoPlay=\"true\  width="512" height="384"></video>
</center>


<footer class="source"> 
Movie Credit: Shan et al: J. Am. Chem. Soc. (2011).  Dasatinib / src
</footer>


---
title: Biophysical modeling at scale: theory and software
class: segue dark nobackground

---
title: How do we reach biological timescales?
subtitle: Challenge: $10^5$ atoms, $10^{12}$ iterations.

<center>
<img height=450 src=figures/protein_timescales.jpg />
</center>

<footer class="source">
Church, 2011.
</footer>


---
title: Molecular dynamics is (finally) fast
subtitle: A \$999 GPU buys 128 ns / day on solvated proteins

<center>
<img height=420 src=figures/TitanNew.jpg />
</center>

<footer class="source"> 
GTX Titan, DHFR, OpenMM 6.2 (openmm.org) See also: Gromacs, AMBER, Charmm, NAMD, DESMOND, ACEMD
</footer>

---
title: OpenMM
subtitle: GPU accelerated molecular dynamics

- Extensible C++ library with Python wrappers
- Hardware backends for CUDA, OpenCL, CPU

<center>
<img height=300 src=figures/openmm.png />
</center>

<footer class="source"> 
openmm.org <br>
Eastman et al, 2012.
</footer>

---
title: OpenMM Powers Folding@Home

- Largest distributed computing project
- 100,000 CPUs, 10,000 GPUs, 40 petaflops!

<center>
<img height=300 src=figures/folding-icon.png />
</center>


<footer class="source"> 
http://folding.stanford.edu/
</footer>

---
title: Traditional Simulation Analysis

<center>
<img height=400 src=figures/NewPaths1.png />
</center>
---
title: Massively parallel simulation?

<center>
<img height=400 src=figures/NewPaths2.png />
</center>


---
title: Introduction to Markov State Models

<center>
<img height=400 src=figures/NewPaths3.png />
</center>


---
title: Counting Transitions

<center>

<img height=300 src=figures/NewPaths-2State.png />

$\downarrow$

$A = (111222222)$

---
title: Counting Transitions

<center>

$A = (111222222)$

$\downarrow$

$C = \begin{pmatrix} C_{1\rightarrow 1} & C_{1\rightarrow 2} \\\ C_{2 \rightarrow 1} & C_{2 \rightarrow 2} \end{pmatrix} =  \begin{pmatrix}2 & 1 \\\ 0 & 5\end{pmatrix}$

</center>

---
title: Estimating Rates

<center>
$C = \begin{pmatrix} C_{1\rightarrow 1} & C_{1\rightarrow 2} \\\ C_{2 \rightarrow 1} & C_{2 \rightarrow 2} \end{pmatrix} = \begin{pmatrix}2 & 1 \\\ 0 & 5\end{pmatrix}$

$\downarrow$

$T = \begin{pmatrix} T_{1\rightarrow 1} & T_{1\rightarrow 2} \\\ T_{2 \rightarrow 1} & T_{2 \rightarrow 2} \end{pmatrix} = \begin{pmatrix}\frac{2}{3} & \frac{1}{3} \\\ 0 & 1\end{pmatrix}$

</center>


---
title: Application: Millisecond Protein Folding


<center>
<img height=450 src=figures/NTL9_network.jpg />
</center>

<footer class="source"> 
Voelz, Bowman, Beauchamp, Pande. J. Am. Chem. Soc., 2010
</footer>


---
title: (Future) Application: Ligand Binding

<center>
<img height=450 src=figures/gprc_ligands.jpg />
</center>

<footer class="source"> 
Kobilka, 2012.  See also Shukla, 2013.
</footer>

---
title: MSMBuilder
subtitle: Finding meaning in massive simulation datasets



<center>
<img height=300 src=figures/msmbuilder.png />
</center>


<footer class="source"> 
msmbuilder.org <br>
https://github.com/msmbuilder/msmbuilder
</footer>


---
title: MSMBuilder3: Design

<div style="float:right; margin-top:-100px">
<img src="figures/flow-chart.png" height="600">
</div>

Builds on [scikit-learn](http://scikit-learn.org/stable/) idioms:

- Everything is a `Model`.
- Models are `fit()` on data.
- Models learn `attributes_`.
- Models `transform()` data.
- `Pipeline()` concatenate models.
- Meta-models like grid search.

<footer class="source"> 
http://rmcgibbo.org/posts/whats-new-in-msmbuilder3/
</footer>

---
title: Re-engineering sklearn for timeseries
subtitle: fit() acts on lists of arrays, rather than $n_{samples} \times n_{features}$ arrays


<pre class="prettyprint" data-lang="python">

from msmbuilder import markovstatemodel

trajectories = [np.array([0, 0, 0, 1, 1, 1, 0, 0])]

msm = markovstatemodel.MarkovStateModel()
msm.fit(trajectories)

msm.transmat_

array([[ 0.75      ,  0.25      ],
       [ 0.33333333,  0.66666667]])

</pre>

This is different from sklearn, which uses 2D arrays!!!

---
title: MSMBuilder3: Demo

<pre class="prettyprint" data-lang="python">
from msmbuilder import example_datasets, cluster, markovstatemodel
from sklearn.pipeline import make_pipeline

dataset = example_datasets.alanine_dipeptide.fetch_alanine_dipeptide()  # From Figshare!
trajectories = dataset["trajectories"]  # List of MDTraj Trajectory Objects

clusterer = cluster.KCenters(n_clusters=10, metric="rmsd")
msm = markovstatemodel.MarkovStateModel()

pipeline = make_pipeline(clusterer, msm)
pipeline.fit(trajectories)

</pre>

<footer class="source"> 
msmbuilder.org <br>
Also has command line interface.
</footer>

---
title: MDTraj

<center>
<img height=250 src=figures/mdtraj_logo-small.png />
</center>


<footer class="source"> 
mdtraj.org <br>
McGibbon et al, 2014
</footer>


---
title: MDTraj
subtitle: Read, write, and analyze single trajectories with only a few lines of Python.

- Used by MSMBuilder for trajectory featurization
- Multitude of formats (PDB, DCD, XTC, HDF, CDF, mol2)
- Geometric trajectory analysis (distances, angles, RMSD)
- Numpy / SSE kernels enable Folding@Home scale analysis


<footer class="source"> 
mdtraj.org <t>
McGibbon et al, 2014 <t>
Haque, 2014
</footer>

---
title: Trajectory munging with MDTraj
subtitle: Lightweight Pythonic API

<pre class="prettyprint" data-lang="python">
import mdtraj as md

trajectory = md.load("./trajectory.h5")
indices, phi = md.compute_phi(trajectory)
</pre>


<center>
<img height=300 src=figures/phi.png />
</center>

<footer class="source"> 
mdtraj.org <t>
McGibbon et al, 2014 <t>
</footer>


---
title: Python packaging blues

<pre>

<font color="red">User: I couldn't really install the mdtraj module on my computer [...]
User: I tried easy_install and other things and that didn't work for me.</font>

</pre>

<center>
<img height=370 src=figures/dependencies0.png />
</center>



---
title: Facile package sharing with conda

<pre>

<font color="red">User: I couldn't really install the mdtraj module on my computer [...]
User: I tried easy_install and other things and that didn't work for me.</font>

<font color="blue">Me: Installing mdtraj should be a one line command:
Me: `conda install -c https://conda.binstar.org/omnia mdtraj`</font>

<font color="red">User: Success!</font>

</pre>


---
title: A full stack for biophysical computation
subtitle: omnia.md

<pre class="prettyprint" data-lang="bash">
conda config --add channels http://conda.binstar.org/omnia/
conda install omnia
</pre>

- OpenMM 6.2
- MDTraj 1.3
- MSMBuilder 3.0
- Yank 1.0 (beta)
- pymbar 2.1
- EMMA

<footer class="source"> 
omnia.md
</footer>


---
title: Benchmarking Small Molecule Forcefields at the Scale of NIST
class: segue dark nobackground

---
title: Models are made to be broken
subtitle: How can we falsify and refine computer based models?

- Chemistry and biophysics are labor-intensive
- Thousands of parameters = thousands of measurements
- Reproducibilty, scalability, archival

<center>
<img height=250 src=figures/robot_image.jpg />
</center>

---
title: Small molecule forcefields need work

<center>
<img height=500 src=figures/mobley_dielectric.png />
</center>

<footer class="source"> 
Fennell, Mobley.  J. Phys. Chem. B. 2014
</footer>


---
title: Data access is killing forcefields

- Forcefields should be consistent with all available data
- Most datasets are heterogeneous, offline, and static

<center>
<img height=325 src=figures/qc_supply_grinder_510482.jpg />
<img height=300 src=figures/stack-of-old-books.jpg />
</center>

<footer class="source"> 
See also work by Wang, Pande, Swope, Case, MacKerell, Ponder, Best, Hummer, Bruschweiler.
</footer>


---
title: WANTED: Reliable, machine-readable, open archive of physicochemical measurements!
class: segue dark nobackground


---
title: NIST Thermodynamics Research Center

<center>
<img height=500 src=figures/boulder3.jpg />
</center>


---
title: Data Capture at NIST/TRC: ThermoML


<center>
<img height=540 src="figures/pipeline.png"/>
</center>

---
title: ThermoML is rapidly growing

<center>
<img height=475 src=figures/thermoml_by_time_new.png />
</center>

<footer class="source"> 
Figure from Chiraco, J. Chem. Eng. Dat., 2013.
</footer>


---
title: Can we leverage ThermoML for forcefield validation?
class: segue dark nobackground

---
title: Density and dielectric constants as forcefield tests

- Sensitive to nonbonded parameters
- Simple ensemble average geometric interpretation

$$\rho = \langle \frac{M}{V} \rangle$$

$$\epsilon = 1 + \frac{4\pi}{3} \frac{\langle \mu \cdot \mu \rangle - \langle \mu \rangle \cdot \langle \mu \rangle}{V k_B T}$$

<footer class="source"> 
See also van der Spoel, JCTC, 2011 and Fennell, 2012.
</footer>


---
title: How many measurements are there?

<center>
<img height=450 src=figures/funnel.png />
</center>

---
title: Benchmarking neat liquid densities and dielectric constants

- OpenMM 6.2
- GAFF + AM1-BCC (Antechamber + OpenEye)
- Converge each density to 0.0002 g / mL ($\approx$ expt. error)


<center>
<img height=200 src=figures/openmm.png />
</center>

<footer class="source"> 
PME + Langevin 1 fs + Monte Carlo Barostat + Fixed HBond Constraints + 1000 molecules per box
</footer>


---
title:  Densities are in the ballpark

<center>
<img height=500 src=figures/densities_thermoml.png />
</center>

---
title: Inverse dielectric constant is proportional to interaction strength

$$U(r) = \frac{1}{4 \pi \epsilon} \frac{q_1 q_2}{r} \propto \frac{1}{\epsilon}$$

---
title:  Static dielectric constants are consistently underestimated

<center>
<img height=450 src=figures/dielectrics_thermoml_nocorr.png />
</center>


---
title: Fixed charges fail to capture polarizability
subtitle: Observed: $\epsilon \approx 2.0$, Predicted: $\epsilon \approx 1.0$, $\Delta \Delta G \approx$ 2 kcal / mol

<center>
<img height=400 src=figures/nonpolar_molecules.png />
</center>

---
title: Atom counting predicts molecular polarizability to within 2%

<center>
<img height=100 src=figures/sales_title.png />
</center>

$$\alpha = 1.53 n_C + 0.17 n_H + 0.57 n_O + 1.05 n_N + 2.99 n_S + \\ 2.48 n_P + 0.22 n_F + 2.16 n_{Cl} + 0.32 $$

$$\epsilon_{corrected} = \epsilon_{MD} + 4 \pi N  \frac{\alpha}{\langle V \rangle}$$


<footer class="source"> 
Sales, 2002
</footer>
---
title: Empirical atomic polarizability corrections reduce bias

<center>
<img height=450 src=figures/dielectrics_thermoml.png />
</center>

---
title: Where do we go from here?

- Scale up, real-time simulation, web frontend
- Perform new experiments in automated wetlab
- Bayesian (MCMC) forcefield / experimental design
- Polarizable forcefields

---
title: Conclusions

- Distributed GPU computing enables millisecond simulation
- MSMBuilder / MDTraj builds machine learning models of conformational dynamics
- omnia.md: Conda packaging of OpenMM, MSMBuilder, MDTraj, and more
- ThermoML: towards automated benchmarking of atomistic models

---
title: People

- John Chodera + ChoderaLab (MSKCC)
- Julie Behr (MSKCC)
- Kenneth Kroenlein (NIST)
- Robert McGibbon (Stanford)
- Peter Eastman (Stanford)
- Vijay Pande + PandeLab (Stanford)
- Joy Ku (Stanford)
- Jason Swails (Rutgers)
- Justin MacCallum (U. Calgary)

<footer class="source"> 
Jan-Hendrik Prinz, Danny Parton, Bas Rustenburg, Sonya Hanson
Greg Bowman, Christian Schwantes, TJ Lane, Vince Voelz, Imran Haque, Matthew Harrigan, Carlos Hernandez, Bharath Ramsundar, Lee-Ping Wang
Frank Noe, Martin Scherer, Xuhui Huang, Sergio Bacallado, Mark Friedrichs
</footer>
