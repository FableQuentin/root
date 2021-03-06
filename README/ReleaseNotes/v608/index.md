% ROOT Version 6.08 Release Notes
% 2015-11-12
<a name="TopOfPage"></a>

## Introduction

ROOT version 6.08/00 is scheduled for release in May, 2016.

For more information, see:

[http://root.cern.ch](http://root.cern.ch)

The following people have contributed to this new version:

 David Abdurachmanov, CERN, CMS,\
 Bertrand Bellenot, CERN/SFT,\
 Rene Brun, CERN/SFT,\
 Philippe Canal, Fermilab,\
 Cristina Cristescu, CERN/SFT,\
 Olivier Couet, CERN/SFT,\
 Gerri Ganis, CERN/SFT,\
 Andrei Gheata, CERN/SFT,\
 Christopher Jones, Fermilab, CMS,\
 Wim Lavrijsen, LBNL, PyRoot,\
 Sergey Linev, GSI, http,\
 Pere Mato, CERN/SFT,\
 Lorenzo Moneta, CERN/SFT,\
 Axel Naumann, CERN/SFT,\
 Danilo Piparo, CERN/SFT,\
 Timur Pocheptsov, CERN/SFT,\
 Fons Rademakers, CERN/IT,\
 Enric Tejedor Saavedra, CERN/SFT,\
 Liza Sakellari, CERN/SFT,\
 Manuel Tobias Schiller,\
 David Smith, CERN/IT,\
 Matevz Tadel, UCSD/CMS, Eve,\
 Vassil Vassilev, Fermilab/CMS,\
 Wouter Verkerke, NIKHEF/Atlas, RooFit

<a name="core-libs"></a>

## Core Libraries

ROOT prepares for [cxx modules](http://clang.llvm.org/docs/Modules.html). One of
the first requirements is its header files to be self-contained (section "Missing
Includes"). ROOT header files were cleaned up from extra includes and the missing
includes were added.

This could be considered as backward incompatibility (for good however). User
code may need to add extra includes, which were previously resolved indirectly
by including a ROOT header. For example:

  * TBuffer.h - TObject.h doesn't include TBuffer.h anymore. Third party code,
    replying on the definition of TBufer will need to include TBuffer.h, along
    with TObject.h.
  * TSystem.h - for some uses of gSystem.
  * GeneticMinimizer.h
  * ...

Other improvements, which may cause compilation errors in third party code:
  * If you get std::type_info from Rtypeinfo.h, type_info should be spelled
    std::type_info.

Also:
  * TPluginManager was made thread-safe [ROOT-7927].

### Containers
* A pseudo-container (generator) was created, ROOT::TSeq<T>. This template is inspired by the xrange built-in function of Python. See the example [here](https://root.cern.ch/doc/master/cnt001__basictseq_8C.html).

### Dictionaries

Fix ROOT-7760: fully allow the usage of the dylib extension on OSx.

### Interpreter Library

Exceptions are now caught in the interactive ROOT session, instead of terminating ROOT.

## Parallelisation
* Three methods have been added to manage implicit multi-threading in ROOT: ROOT::EnableImplicitMT(numthreads), ROOT::DisableImplicitMT and ROOT::IsImplicitMTEnabled. They can be used to enable, disable and check the status of the global implicit multi-threading in ROOT, respectively.
* Even if the default reduce function specified in the invocation of the MapReduce method of TProcPool returns a pointer to a TObject, the return value of MapReduce is properly casted to the type returned by the map function.
* Add tutorial showing how to fill randomly histograms using the TProcPool class

## I/O Libraries
* Custom streamers need to #include TBuffer.h explicitly (see
[section Core Libraries](#core-libs))
* Check and flag short reads as errors in the xroot plugins. This fixes [ROOT-3341].


## TTree Libraries

* Update TChain::LoadTree so that the user call back routine is actually called for each input file even those containing TTree objects with no entries.
* Repair setting the branch address of a leaflist style branch taking directly the address of the struct.  (Note that leaflist is nonetheless still deprecated and declaring the struct to the interpreter and passing the object directly to create the branch is much better).
* Provide an implicitly parallel implementation of TTree::GetEntry. The approach is based on creating a task per top-level branch in order to do the reading, unzipping and deserialisation in parallel. In addition, a getter and a setter methods are provided to check the status and enable/disable implicit multi-threading for that tree (see Parallelisation section for more information about implicit multi-threading).
* Properly support std::cin (and other stream that can not be rewound) in TTree::ReadStream. This fixes [ROOT-7588].

## Histogram Libraries

* TH2Poly has a functional Merge method.

## Math Libraries


## RooFit Libraries


## 2D Graphics Libraries

* In `TColor::SetPalette`, make sure the high quality palettes are defined
  only ones taking care of transparency. Also `CreateGradientColorTable` has been
  simplified.
* New fast constructor for `TColor` avoiding to call `gROOT->GetColor()`. The
  normal constructor generated a big slow down when creating a Palette with
  `CreateGradientColorTable`.
* In `CreateGradientColorTable` we do not need anymore to compute the highest
  color index.
* In `TGraphPainter`, when graphs are painted with lines, they are split into
  chunks of length `fgMaxPointsPerLine`. This allows to paint line with an "infinite"
  number of points. In some case this "chunks painting" technic may create artefacts
  at the chunk's boundaries. For instance when zooming deeply in a PDF file. To avoid
  this effect it might be necessary to increase the chunks' size using the new function:
  `TGraphPainter::SetMaxPointsPerLine(20000)`.
* When using line styles different from 1 (continuous line), the behavior of TArrow
  was suboptimal. The problem was that the line style is also applied to the arrow
  head, which is usually not what one wants.
  The arrow tip is now drawn using a continuous line.
* It is now possible to select an histogram on a canvas by clicking on the vertical
  lines of the bins boundaries.
  This problem was reported [here](https://sft.its.cern.ch/jira/browse/ROOT-6649).
* When using time format in axis, `TGaxis::PaintAxis()` may in some cases call
  `strftime()` with invalid parameter causing a crash.
  This problem was reported [here](https://sft.its.cern.ch/jira/browse/ROOT-7689).

## 3D Graphics Libraries

* When painting a `TH3` as 3D boxes, `TMarker3DBox` ignored the max and min values
  specified by `SetMaximum()` and `SetMinimum()`.

## Geometry Libraries


## Database Libraries

* Fix `TPgSQLStatement::SetBinary` to actually handle binary data (previous limited to ascii).

## Networking Libraries


## GUI Libraries


## Montecarlo Libraries


## PROOF Libraries


## Language Bindings

### PyROOT
  * Added a new configuration option to disable processing of the rootlogon[.py|C] macro in addition
    ro the -n option in the command arguments. To disable processing the rootlogon do the following 
    before any other command that will trigger initialization:
    ```
    >>> import ROOT
    >>> ROOT.PyConfig.DisableRootLogon = True
    >>> ...
    ```

### Notebook integration

  * Refactoring of the Jupyter integration layer into the new package JupyROOT.
  * Added ROOT [Jupyter Kernel for ROOT](https://root.cern.ch/root-has-its-jupyter-kernel)
    * Magics are now invoked with standard syntax "%%", for example "%%cpp".
    * The methods "toCpp" and "toPython" have been removed.
  * Factorise output capturing and execution in an accelerator library and use ctypes to invoke functions.
  * When the ROOT kernel is used, the output is consumed progressively

## JavaScript ROOT


## Tutorials


## Class Reference Guide

## Build, Configuration and Testing Infrastructure
* Added new 'builtin_vc' option to bundle a version of Vc within ROOT. 
  The default is OFF, however if the Vc package is not found in the system the option is switched to
  ON if the option 'vc' option is ON.  



