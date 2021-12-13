# FastGPL

## Installation and usage

\name{} can be obtained from GitHub using
\begin{lstlisting}
git clone https://github.com/llyang/FastGPL.git
\end{lstlisting}
To compile and install the library, one needs to have CMake installed, and the following commands have to be executed:
\begin{lstlisting}
cd FastGPL
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make
make install
\end{lstlisting}
where the prefix can be chosen by the user. The compilation procedure also builds a simple program in the \texttt{test} directory, which demonstrates the usage of the library.

To use \name{}, one simply includes the header file \texttt{FastGPL.h}, and invokes the function \texttt{FastGPL::G} in the program. The syntax can be seen from the following code snippet:
\begin{lstlisting}
    vector<complex<double>> a { {0.1, 0.2}, 0.4 };
    vector<int> s {1, -1};
    double x { 0.8 };
    cout << FastGPL::G(a, s, x) << '\n';
    cout << FastGPL::G(a, x) << '\n';
\end{lstlisting}
Here, \texttt{x} must be a non-negative real number. The reason is given in Section~\ref{sec:notation}. In the \texttt{G(a, s, x)} version of the function, \texttt{a} is a vector of complex indices, and \texttt{s} is a vector with elements being either $\mathtt{+1}$ or $\mathtt{-1}$, which denote the signs of the imaginary parts of the corresponding indices. If one uses the $\texttt{G(a, x)}$ version of the function, and if the imaginary part of a certain index is zero, the program will automatically choose $\mathtt{+1}$ as the sign. This convention is the same as $\texttt{GiNaC}$. Hence, the last two lines in the above code snippet correspond to $\mathtt{G(0.1+0.2i, 0.4-i0; 0.8)}$ and $\mathtt{G(0.1+0.2i, 0.4+i0; 0.8)}$, respectively.

As a side effect, \name{} also provides several functions for the evaluation of normal polylogarithms $\mathrm{Li}_n(z)$ up to weight 8 and the Neilsen's generalized polylogarithm $\mathrm{S}_{2,2}(z)$ for the complex argument $z$. Furthermore, as elliptic integrals often appear in loop integrals which cannot be represented purely with GPLs, we also provide two functions to evaluate the complete elliptic integrals of the first and the second kinds. The usage of these functions can be seen from the following code snippet:
\begin{lstlisting}    
    complex<double> z { 1.2, 0.5 };
    int s { 1 };
    cout << FastGPL::PolyLog(6, z, s);
    cout << FastGPL::S22(z, s);
    cout << FastGPL::EllipticK(z);
    cout << FastGPL::EllipticE(z);
\end{lstlisting}
Note that for \texttt{PolyLog} and \texttt{S22}, \texttt{s} will default to $\mathtt{-1}$ if it's omitted and if the imaginary part of $\mathtt{z}$ is zero. This convention is the same as \texttt{GiNaC} and \texttt{Mathematica}.