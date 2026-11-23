# GROMACS Command Generator

**A free web tool that turns a few choices into a complete GROMACS setup: every `gmx` command and every `.mdp` file, in running order.**

Live tool: https://gmx-gen.celes26protein-lab.workers.dev

No installation and no sign-up. Open the page, choose your settings, press **Generate**, and copy or download the result.

---

## What it does

Setting up a molecular dynamics (MD) simulation with GROMACS means running a long chain of commands and keeping several `.mdp` files consistent: the same temperature, the same cutoff and the same pressure coupling in every stage. This tool builds that chain for you from one set of choices, so the stages always match.

It produces:

- the commands for each step, ready to paste into a terminal
- the `.mdp` files for ions, energy minimization, NVT, NPT and production MD
- a `commands.txt` file with everything in order
- a resume command for production MD, in case the run is interrupted

## The workflow it generates

| # | Step | Main GROMACS tools |
|---|------|--------------------|
| 1 | Topology | `pdb2gmx` |
| 2 | Box | `editconf` |
| 3 | Solvation | `solvate` |
| 4 | Ionization (neutralize and add salt) | `grompp`, `genion` |
| 5 | Energy minimization | `grompp`, `mdrun` |
| 6 | NVT equilibration | `grompp`, `mdrun` |
| 7 | NPT equilibration | `grompp`, `mdrun` |
| 8 | Production MD | `grompp`, `mdrun` |
| 9 | Resume after interruption (optional) | `mdrun -cpi` |

The equilibration and production steps pass the checkpoint (`.cpt`) of the previous stage, so velocities carry over.

## How to use it

1. Open the tool and choose your settings (see below).
2. Press **Generate**.
3. Save the `.mdp` files next to your structure file. Each file has **Copy** and **Download** buttons.
4. Copy the commands, or press **Download commands.txt**, and run them in order in a terminal where GROMACS is installed.

You need your own structure file (for example `protein.pdb`) in the folder where you run the commands. The tool never sees this file.

## Settings you can choose

**Topology and system**
- Input structure file name
- Force field: `amber99sb-ildn`, `amber99sb`, `charmm27`, `oplsaa`, `gromos54a7`
- Water model: `tip3p`, `spce`, `tip4p`, `tip5p`, `spc`
- Box type (cubic, dodecahedron, octahedron) and distance from the solute to the box edge
- Cation (Na+, K+ or Mg2+) and salt concentration; the anion is Cl-, and the system is neutralized

**Simulation protocol**
- Temperature, applied consistently to every stage
- Cutoff radius (you type the value; the right choice depends on your force field)
- Energy minimization: force tolerance and maximum steps
- NVT and NPT length
- Position restraints on the protein during equilibration (adds `-DPOSRES` and `-r`)
- Pressure coupling: `C-rescale` (GROMACS 2021 or later), or `Berendsen` / `Parrinello-Rahman` for older versions
- Production MD: length, how often to save a frame, and initial velocities (continue from NPT, or generate new velocities with a chosen seed, useful for independent replicas)
- `-maxwarn 1` for `grompp`, as an opt-in checkbox

**Advanced `.mdp` settings**
- Temperature coupling groups (`Protein / Non-Protein` or `System`), EM integrator, time step, `tau_t`, `tau_p`, compressibility, `nstlist`

**mdrun options**
- Hardware: CPU, CPU with a chosen thread setting, or GPU (Auto by default, or explicit offload of non-bonded, PME and bonded work)
- Optional `-v` (progress), `-maxh` (stop safely after a number of hours) and `-cpt` (checkpoint interval); periodic checkpoints are switched on by default for GPU runs

## Default values

If you press **Generate** without changing anything, you get a standard protein-in-water protocol: `amber99sb-ildn` with `tip3p`, a cubic box with 1.0 nm to the edge, 0.15 mol/L NaCl, 300 K, a 1.0 nm cutoff, 100 ps each of NVT and NPT with position restraints, then 5 ns of production MD with a frame every 10 ps. The time step is 2 fs with hydrogen-bond constraints, PME electrostatics, the V-rescale thermostat and `C-rescale` pressure coupling.

## Requirements

- GROMACS installed on your own computer or cluster. The commands use the `gmx` form (GROMACS 2018 and later).
- `C-rescale` needs GROMACS 2021 or later. Choose Berendsen or Parrinello-Rahman for older versions.
- The chosen force field must be installed in your GROMACS.

## Important notes

- **Check before you run.** The output is a starting point, not a substitute for your own judgment. Read the commands and `.mdp` files, and confirm that the force field, water model, cutoff and other parameters suit your system and your research question.
- **Designed for a protein in water.** The default temperature coupling groups assume a protein is present. Systems with membranes, ligands, nucleic acids or unusual components usually need extra steps and different groups; the `System` option helps with simple cases only.
- **Ion names** (`NA`, `K`, `MG`, `CL`) follow the GROMACS force-field files. If `genion` fails, check the ion names in your force field.
- **`ions.mdp`** is a general starting file for placing ions; review it for your system.
- **GPU options** depend on your GROMACS build and settings. If `mdrun` reports an error, start with Auto.
- **`-maxwarn 1`** hides one `grompp` warning. Use it only if you understand the warning.

## Privacy

When you press **Generate**, the options you selected (for example the input file name, temperature and run length) are sent to the tool's server, which builds the text and sends it back. Structure files and other data from your computer are never uploaded. Nothing is shared with the GROMACS project.

## Disclaimer

This is an independent tool. It is not affiliated with, endorsed by or supported by the GROMACS project or its developers. GROMACS is the work of its own authors; please cite it in your publications according to its documentation. The tool is provided as is, without any warranty, and you are responsible for the simulations you run with its output.

## License

This project is licensed under the Business Source License 1.1: free to use for personal, academic, and evaluation purposes. The app is provided as-is; see the live app for full terms.

For any other use, please get in touch (see below).

## Feedback

Questions, bug reports, suggestions and requests for other uses are welcome: https://github.com/celes26protein-lab
