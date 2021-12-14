# FastGPL

This is the code for the paper [[arXiv:2112.04122]](https://arxiv.org/abs/2112.04122)
>  FastGPL: a C++ library for fast evaluation of generalized polylogarithms
>
>  Yuxuan Wang, Li Lin Yang, Bin Zhou

If you are using `FastGPL`, please cite the above.

## Installation and usage

`FastGPL` can be obtained from GitHub using
```bash
git clone https://github.com/llyang/FastGPL.git
```
To compile and install the library, one needs to have CMake installed, and the following commands have to be executed:
```bash
cd FastGPL
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make
make install
```
where the prefix can be chosen by the user. The compilation procedure also builds a simple program in the `test` directory, which demonstrates the usage of the library.

To use `FastGPL`, one simply includes the header file `FastGPL.h`, and invokes the function `FastGPL::G` in the program. The syntax can be seen from the following code snippet:

```C++
    vector<complex<double>> a { {0.1, 0.2}, 0.4 };
    vector<int> s {1, -1};
    double x { 0.8 };
    cout << FastGPL::G(a, s, x) << '\n';
    cout << FastGPL::G(a, x) << '\n';
```

Here, `x` must be a non-negative real number. The reason is that this allows the program to easily decide which side of a branch cut to take according to the signs of the imaginary parts of the indices. In the `G(a, s, x)`  version of the function, `a` is a vector of complex indices, and `s` is a vector with elements being either `+1` or `-1`, which denote the signs of the imaginary parts of the corresponding indices. If one uses the `G(a, x)` version of the function, and if the imaginary part of a certain index is zero, the program will automatically choose `+1` as the sign. This convention is the same as `GiNaC`. Hence, the last two lines in the above code snippet correspond to `G(0.1+0.2i, 0.4-i0; 0.8)` and `G(0.1+0.2i, 0.4+i0; 0.8)`, respectively.

As a side effect, `FastGPL` also provides several functions for the evaluation of normal polylogarithms `Li<sub>n</sub>(z)` up to weight 8 and the Neilsen's generalized polylogarithm `S<sub>2,2</sub>(z)` for the complex argument `z`. Furthermore, as elliptic integrals often appear in loop integrals which cannot be represented purely with GPLs, we also provide two functions to evaluate the complete elliptic integrals of the first and the second kinds. The usage of these functions can be seen from the following code snippet:
```C++
    complex<double> z { 1.2, 0.5 };
    int s { 1 };
    cout << FastGPL::PolyLog(6, z, s);
    cout << FastGPL::S22(z, s);
    cout << FastGPL::EllipticK(z);
    cout << FastGPL::EllipticE(z);
```
Note that for `PolyLog` and `S22`, `s` will default to `-1` if it's omitted and if the imaginary part of `z` is zero. This convention is the same as `GiNaC` and `Mathematica`.

