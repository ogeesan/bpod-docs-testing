---
icon: material/function-variant
---

# Function Reference

This section contains documentation of Bpod's functions. Bpod's functions, stored in Bpod_Gen2/Functions/, are added onto MATLAB's [Path](https://mathworks.com/help/matlab/matlab_env/what-is-the-matlab-search-path.html) at startup when `Bpod` is used.

The sidebar section on the left contains sections for functions, grouped into categories.

Because all of the functions in the Functions/ folder are added onto the Path, Bpod always has access to functions like ['TrialTypeOutcomePlot()'](general-plugins.md#trialtypeoutcomeplot) regardless of which protocol is active. Protocol specific functions should be placed within the protocol folder.