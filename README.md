# handyG



[![pipeline status](https://gitlab.psi.ch/mcmule/handyG/badges/master/pipeline.svg)](https://gitlab.psi.ch/mcmule/handyG/commits/master)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

This is the code for the paper [[arXiv:1909.01656]](https://arxiv.org/abs/1909.01656)
>  handyG - rapid numerical evaluation of generalised polylogarithms
>
>  L. Naterop, A. Signer, Y. Ulrich

If you are using `handyG` (under the conditionals of GPL v3), please cite the
above.


## Obtaining the code
The code can be downloaded from this page in compressed form or cloned
using the `git` command
```bash
git clone https://gitlab.psi.ch/mcmule/handyG.git
```
This will download `handyG` into a subfolder called `handyg`. Within
this folder
```bash
git pull
```
can be used to update `handyG`.

## Installation
`handyG` should run on a variety of systems though this can obviously
not be guaranteed. The code follows the conventional installation
scheme[^1]
```bash
./configure  # Look for compilers and make a guess at
             # necessary flags
make all     # Compiles the library
make check   # Performs a variety of checks (optional)
make install # Installs library into prefix (optional)
```

`handyG` has a Mathematica interface (activate with `--with-mcc`) and
a GiNaC interface (activate with `--with-ginac`) that can be activated
by supplying the necessary flags to `./configure`. The latter is only
used for testing purposes and is not actually required for running.
Another important flag is `--quad` which enables quadruple precision
in Fortran. Note that this will slow down `handyG`, so that it should
only be used if double-precision is indeed not enough.

The compilation process creates the following results

 * `libhandyg.a`  the `handyG` library
 * `handyg.mod`   the module files for Fortran 90
 * `geval`        a binary file for quick-and-dirty evaluation
 * `handyG`       the Mathematica interface


[^1]: Desipite the name, `./configure` has nothing to do with autotools

## Usage in Fortran
`handyG` is written with Fortran in mind. We provide a module
`handyg.mod` containing the following objects

* `prec`

    the working precision as a Fortran `kind`. This is read-only,
    the code needs to be reconfigured for a change to take effect.
    Note that this does not necessarily increase the result's
    precision without also changing the next options.

* `set_options`

    a subroutine to set runtime parameters of `handyG`. `set_options`
    takes the following arguments

   * `real(kind=prec) :: MPLdel = 1e-15`

      difference between two successive terms at which the series
      expansion (6) is truncated.

   * `real(kind=prec) :: Lidel = 1e-15`

      difference $`|\delta_i|`$ between two successive terms at which the
      series expansion for

      ```math
        {\rm Li}_n(x) = \sum_{i=1}^\infty  \frac{x^i}{i^n}
                      = \sum_{i=1}^\infty \delta_i
      ```
      is truncated.


   * `real(kind=prec) :: hCircle = 1.1`

      the size of the H&ouml;lder circle $`\lambda`$ (see Section
      4.4).

* `inum`

    a datatype to handle $`{\rm i0}^\pm`$-prescription (see Section
    3.4).

* `clearcache`

    `handyG` caches a certain number of classical polylogarithms (see
    Section 3.5). This resets the cache (in a Monte Carlo
    this should be called at every phase space point).

* `G`

    the main interface for generalised polylogarithms.

The following code calculates five GPLs (see paper for details)
```fortran
PROGRAM gtest
  use handyG
  complex(kind=prec) :: res(5), x, weights(4)
  call clearcache

  x = 0.3 ! the parameter

  ! flat form with integers
  res(1) = G((/ 1, 2, 1 /))

  ! very flat form for real numbers using F2003 arrays
  res(2) = G([ 1., 0., 0.5, real(x)])
  ! this is equivalent to the flat expression
  res(2) = G([ 1., 0., 0.5 ], real(x))
  ! or in condesed form
  res(2) = G((/1, 2/), (/ 1., 0.5 /), real(x))

  ! flat form with complex arguments
  weights = [(1.,0.), (0.,0.), (0.5,0.), (!.,1.) ]
  res(3) = G(weights, x)

  ! flat form with explicit i0-prescription
  res(4) = G([inum(1.,+1),inum(0,+1),inum(5,+1)], &
                    inum(1/x,di0))
  res(5) = G([inum(1.,-1),inum(0,+1),inum(5,+1)],&
                    inum(1/x,di0))
  ! this is equivalent to
  res(5) = G((/1,2/),[inum(1.,-1),inum(5,+1)], &
                    inum(1/x,+1))

  do i =1,5
    write(*,900) i, real(res(i)), aimag(res(i))
  enddo
900 FORMAT("res(",I1,") = ",F9.6,"+",F9.6,"i")
END PROGRAM gtest
```

The easiest way to compile code is with `pkg-config`. Assuming `handyG` has
been installed with `make install`, the example program `example.f90` can be
compiled as (assuming you are using GFortran)

```bash
$ gfortran -o example example.f90 \
       `pkg-config --cflags --libs handyg`
$ ./example
res(1) = -0.822467+ 0.000000i
res(2) =  0.128388+ 0.000000i
res(3) = -0.003748+ 0.003980i
res(4) = -0.961279+-0.662888i
res(5) = -0.961279+ 0.662888i
```
If `pkg-config` is not avaible and/or for non-standard installations it might
be neccessary to specify the search paths
```bash
$ gfortran -o example example.f90 \
>     -I/absolute/path/to/handyg -fdefault-real-8 \
>     -L/absolute/path/to/handyg -lhandyg
```

## Usage in Mathematica
We have interfaced our code to Mathematica using Wolfram's MathLink interface.
Below we show how to calculate the functions above in Mathematica, again,
assuming that the code was installed with `make install`
```mathematica
In[1]:= Install["handyg"];
handyG by L. Naterop, Y. Ulrich, A. Signer

In[2]:= x=0.3;

In[3]:= res[1] = G[1,2,1]

Out[3]= -0.822467

In[4]:= res[2] = G[1,0,1/2,x]

Out[4]= 0.128388

In[5]:= res[3] = G[1,0,1/2,1+I,x]

Out[5]= -0.00374796 + 0.00398002 I

In[6]:= res[4] = G[SubPlus[1],5,1/x]

Out[6]= -1.12732 - 0.701026 I

In[7]:= res[5] = G[SubMinus[1],5,1/x]

Out[7]= -1.12732 + 0.701026 I
```
Using `SubPlus` and `SubMinus` the side of the branch cut can be specified. In
Mathematica, this can be entered using `ctrl _` followed by `+` or `-`.  When
using `handyG` in Mathematica, keep in mind that it uses Fortran which means
that computations are performed with fixed precision.

