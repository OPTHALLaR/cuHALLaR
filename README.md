## **[cuHALLaR](https://github.com/OPTHALLaR/cuHALLaR)**

cuHALLaR is an enhanced GPU-based first-order method solver written in Julia for low-rank semi-definite programming problems (SDPs). It is developed based on methodologies described in this [paper](https://optimization-online.org/wp-content/uploads/2025/05/SDP_GPU_May19_Revised.pdf). It supports SDPA files and introduces a new file format that allows low-rank factorizations of the objective and constraints.

#### Problem Statement:

cuHALLaR solves the primal–dual pair of semidefinite programs (SDPs)

$$
P_{*} := \min_{X} \{ C \bullet X \ : \ A(X) = b,\ X \in \Delta^n \}
$$

and

$$
D_{*} := \max_{p \in \mathbb{R}^m,\ \theta \in \mathbb{R}} \{ -b^{\top}p - \theta \mid S = C + \theta I \succeq 0,\ \theta \ge 0 \}.
$$

where $b \in \mathbb{R}^m$, $C \in \mathbb{S}^n$, $A : \mathbb{S}^n \to \mathbb{R}^m$ is a linear map, and $\Delta^n$ is the spectraplex

$$
\Delta^n := \{ X \in \mathbb{S}^n : \mathrm{Tr}(X) \le \tau,\ X \succeq 0 \}.
$$

**Features of the problem:**

- linear, trace-constrained objective
- affine constraints
- positive-semidefinite variables
- objective and constraints may be represented as low-rank factorizations

#### Current release:

cuHALLaR is currently under active development. A pre-compiled binary that processes SDPA (.dat-s) format files along with Hybrid Sparse Low-Rank (HSLR) formatted files is available. Users testing the solver on Linux can download the release from [the release site](https://github.com/OPTHALLaR/cuHALLaR).

Then, uncompress the .tar.xz by running

```
tar -xvf cuHALLaR.tar.xz
```

#### Getting started:

By running

```sh
./bin/cuHALLaR -i <path_to_file> -c <path_to_options> -p <path_to_primal_solution_file> -d <path_to_dual_solution_file> [other options]
```

we can solve SDPs represented in standard SDPA format (the user must provide a trace_bound parameter in the settings) and files in the HSLR format (see the user manual).

If everything goes well, we would see logs like below:

```
---------- Basic Settings ------------------
input_path = examples/hybrid_HSLR_format_v2.HSLR
output_path = out.txt
config_file =
...
---------- Intermediate Settings ------------------
scale_A = 1.0
scale_C = 1.0
beta0 = 10.0
...
---------- Advanced Settings ------------------
maxiter_fista = 10000
mu_fista = 0.75
chi_fista = 0.0001
...

Reading SDPA file: examples/mc_3.dat-s
Problem dimensions:
  - Matrix size: 3000 x 3000
  - Number of constraints: 216172
  - Number of blocks: 1
  - Trace bound: 51601.0

Solving SDP problem with GPU acceleration...

##########################################################################
  #  rank        gap     feas    pval    dval    pnlty   steps
   0    1         -       2.9e-03    9.690e-06    NaN    1.0e+01 AE
   1    1     NaN    2.9e-03    8.201e-06    2.500e-03    1.0e+01  AE
   2    1     NaN    2.9e-03    6.903e-06    6.250e-03    1.0e+01  AE
...
  42    3     8.8e-06    1.3e-08    8.357e-02    8.357e-02    6.6e+03
Final Results
Primal Obj             = 0.08356806847402057
Dual Obj               = 0.08356659006982121
PD Gap                 = 8.844561680506419e-6
Primal infeasibility     = 1.3353696237066644e-8

#ADAP FISTA Calls: 44
#ACG Iterations: 262
#FW Calls: 2
Primal val unscaled = 4312.195901327936
Run time = 2.718115 seconds
Writing output
Output written to out.txt
```

_Note:_ Even if the Julia application is precompiled, there will still be some dynamic compilation time at runtime due to Julia's Just-In-Time (JIT) compilation mechanism. (up to several seconds per run)

#### Environment Requirements:

cuHALLaR was mainly developed using Red Hat Enterprise Linux 9.5, however it is expected to run with a minimum system requirement of Ubuntu 20.04. We have tested the following configurations, all of which successfully run the software:

- H200 + Red Hat Enterprise Linux 9.5 (“Plow”)
- H100 + Red Hat Enterprise Linux 9.5 (“Plow”)
- A100 + Red Hat Enterprise Linux 9.5 (“Plow”)
- RTX3080 + Ubuntu 22.04 (“Jammy Jellyfish”)

#### Output Format

cuHallar outputs the primal and dual solution as csv files. Option `-p` provides a path for the output primal solution, and `-d` a path to the dual solution. The primal solution output is the lowrank factor $Y \in \mathbb{R}^{n \times r}$, such that $X = YY^\top$. The dual solution file contains one line where the first element is the dual of the trace constraint, while the remaining elements are the dual vector $p \in \mathbb{R}^m$.

#### Settings

cuHALLaR provides users with customizable parameters to fine-tune the solving process according to specific problem requirements (if needed). Below is a detailed description of each parameter:

| **Option**                   | **Default Value**  | **Description**                                                                                                                                         |
| ---------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Input / Output**           |                    |                                                                                                                                                         |
| `-i`                         | none (required)    | Path to the input file in HSLR format.                                                                                                                  |
| `-p`                         | `"primal_out.txt"` | Path for the output file containing the primal solution.                                                                                                |
| `-d`                         | `"dual_out.txt"`   | Path for the output file containing the dual solution.                                                                                                  |
| `-c`                         | `""`               | Path to a configuration file to load options.                                                                                                           |
| `--run_tests`                | `false (flag)`     | Run test routine with example instances.                                                                                                                |
| **FISTA Parameters**         |                    |                                                                                                                                                         |
| `--maxiter_fista`            | `1e4`              | Maximum number of ADAP-FISTA iterations.                                                                                                                |
| `--mu_fista`                 | `0.5`              | FISTA parameter μ.                                                                                                                                      |
| `--chi_fista`                | `1e-4`             | FISTA parameter χ.                                                                                                                                      |
| `--L0_fista`                 | `1.0`              | Initial Lipschitz constant for ADAP-FISTA.                                                                                                              |
| `--L_inc_fista`              | `2.0`              | Lipschitz constant increment factor.                                                                                                                    |
| `--sigma_fista`              | `0.3`              | FISTA parameter σ.                                                                                                                                      |
| `--err_tol_fista`            | `1e-8`             | Error tolerance for ADAP-FISTA.                                                                                                                         |
| **AIPP Parameters**          |                    |                                                                                                                                                         |
| `--maxiter_aipp`             | `5`                | Maximum number of AIPP iterations.                                                                                                                      |
| `--lam0_aipp`                | `0.1`              | AIPP initial parameter λ₀.                                                                                                                              |
| **Hybrid Low-Rank / Hallar** |                    |                                                                                                                                                         |
| `--maxiter_hlr`              | `10`               | Maximum iterations for the hybrid low-rank method.                                                                                                      |
| `--maxiter_hallar`           | `1e4`              | Maximum number of outer \ourmethod iterations.                                                                                                          |
| **Stopping Criteria**        |                    |                                                                                                                                                         |
| `--eps_pfeas`                | `1e-5`             | Primal feasibility tolerance (ε_feas).                                                                                                                  |
| `--eps_gap`                  | `1e-5`             | Relative duality gap tolerance (ε_gap).                                                                                                                 |
| **Penalty Parameters**       |                    |                                                                                                                                                         |
| `--beta0`                    | `10.0`             | Initial penalty parameter β₀.                                                                                                                           |
| `--beta_inc`                 | `1.1`              | Increment factor for β.                                                                                                                                 |
| `--beta_min`                 | `10.0`             | Minimum value for β.                                                                                                                                    |
| `--beta_max`                 | `1e11`             | Maximum value for β.                                                                                                                                    |
| **Scaling**                  |                    |                                                                                                                                                         |
| `--scale_A`                  | `1.0`              | Scaling factor for constraint matrices.                                                                                                                 |
| `--scale_C`                  | `1.0`              | Scaling factor for the cost matrix.                                                                                                                     |
| **Miscellaneous**            |                    |                                                                                                                                                         |
| `--verbosity`                | `1`                | Verbosity level (0: silent, 1: summary steps, 2: detailed, 3: debug).                                                                                   |
| `--time_limit`               | `3600.0`           | Time limit in seconds.                                                                                                                                  |
| `--trace_bound`              | `1.0`              | τ value for the trace constraint. Used when passing a sparse SDPA format file. When using HSLR format, the trace bound is passed inside the input file. |

For example, to set **`time_limit`** to `300.0` and solve a problem, we can execute

```
./bin/cuHALLaR -i <path_to_file> --time_limit 300.0
```

Alternatively, the user may build a configuration file with the options (see `options.cfg` file in the examples directory) and pass to cuHALLaR with `-c <path_to_file>`.

#### Developing Team

cuHALLaR is developed by

- Jacob M. Aguirre: [aguirre@gatech.edu](mailto:aguirre@gatech.edu)
- Diego Cifuentes: [dfc3@gatech.edu](mailto:dfc3@gatech.edu)
- Vincent Guigues
- Renato D.C. Monteiro
- Victor Hugo Nascimento: [nascimento.victor.1@fgv.edu.br](mailto:nascimento.victor.1@fgv.edu.br)
- Arnesh Sujanani: [a3sujana@uwaterloo.edu](mailto:a3sujana@uwaterloo.edu)

#### Reference

- Monteiro, Renato DC, Arnesh Sujanani, and Diego Cifuentes. "A low-rank augmented Lagrangian method for large-scale semidefinite programming based on a hybrid convex-nonconvex approach." arXiv preprint arXiv:2401.12490 (2024).

- Aguirre, Jacob M., et al. "cuHALLaR: A GPU Accelerated Low-Rank Augmented Lagrangian Method for Large-Scale Semidefinite Programming." arXiv preprint arXiv:2505.13719 (2025).

```
@article{monteiro2024low,
  title={A low-rank augmented Lagrangian method for large-scale semidefinite programming based on a hybrid convex-nonconvex approach},
  author={Monteiro, Renato DC and Sujanani, Arnesh and Cifuentes, Diego},
  journal={arXiv preprint arXiv:2401.12490},
  year={2024}
}

@article{aguirre2025cuhallar,
  title={cuHALLaR: A GPU Accelerated Low-Rank Augmented Lagrangian Method for Large-Scale Semidefinite Programming},
  author={Aguirre, Jacob M and Cifuentes, Diego and Guigues, Vincent and Monteiro, Renato DC and Nascimento, Victor Hugo and Sujanani, Arnesh},
  journal={arXiv preprint arXiv:2505.13719},
  year={2025}
}
```

**cuHALLaR is free for academic use. When utilizing cuHALLaR in published works, please cite the source above.**
