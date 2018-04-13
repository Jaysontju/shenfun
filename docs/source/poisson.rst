.. Automatically generated Sphinx-extended reStructuredText file from DocOnce source
   (https://github.com/hplgit/doconce/)

.. Document title:

Demo - 1D Poisson equation
%%%%%%%%%%%%%%%%%%%%%%%%%%

:Authors: Mikael Mortensen (mikaem at math.uio.no)
:Date: Apr 13, 2018

*Summary.* This is a demonstration of how the Python module `shenfun <https://github.com/spectralDNS/shenfun>`__ can be used to solve the Poisson
equation with Dirichlet boundary conditions in one dimension. Spectral convergence, as shown in Figure :ref:`fig:ct0`, is demonstrated. 
The demo is implemented in
a single Python file `dirichlet_poisson1D.py <https://github.com/spectralDNS/shenfun/blob/master/demo/dirichlet_poisson1D.py>`__, and the numerical method is is described in more detail by J. Shen :cite:`shen1` and :cite:`shen95`.

.. _fig:ct0:

.. figure:: https://rawgit.com/spectralDNS/spectralutilities/master/figures/poisson1D_errornorm.png

   *Convergence of 1D Poisson solvers for both Legendre and Chebyshev modified basis function*

Model problem
=============
Poisson equation
----------------

The Poisson equation is given as

.. math::
   :label: eq:poisson

        
        \nabla^2 u(x) = f(x) \quad \text{for }\, x \in [-1, 1], 
        

.. math::
   :label: _auto1

          
        u(-1)=a, u(1)=b, \notag
        
        

where :math:`u(x)` is the solution, :math:`f(x)` is a function and :math:`a, b` are two possibly
non-zero constants. 

To solve Eq. :eq:`eq:poisson` with the Galerkin method we need smooth basis functions, :math:`v_k`, that live
in the Hilbert space :math:`H^1(x)` and that satisfy the given boundary conditions. And then we look for solutions
like

.. math::
   :label: eq:u

        
        u(x) = \sum_{k=0}^{N-1} \hat{u}_k v_k(x), 
        

where :math:`N` is the size of the discretized problem and the basis is :math:`V^N=\text{span}\{v_k\}_{k=0}^{N-1}`.
The basis functions can, for example,  be constructed from `Chebyshev <https://en.wikipedia.org/wiki/Chebyshev_polynomials>`__, :math:`T_k(x)`, or `Legendre <https://en.wikipedia.org/wiki/Legendre_polynomials>`__, :math:`L_k(x)`, functions 
and we use the common notation :math:`\phi_k(x)` to represent either one of them. It turns out that 
it is easiest to use basis functions with homogeneous Dirichlet boundary conditions

.. math::
   :label: _auto2

        
        v_k(x) = \phi_k(x) - \phi_{k+2}(x),
        
        

for :math:`k=0, 1, \ldots N-3`, and then the last two are added as two linear basis functions (that belong to the kernel of the Poisson equation)

.. math::
   :label: _auto3

        
        v_{N-2} = \frac{1}{2}(\phi_0 + \phi_1), 
        
        

.. math::
   :label: _auto4

          
        v_{N-1} = \frac{1}{2}(\phi_0 - \phi_1).
        
        

With these two final basis functions it is easy to see that the two last degrees
of freedom, :math:`\hat{u}_{N-2}` and :math:`\hat{u}_{N-1}`, now are given as

.. math::
   :label: eq:dirichleta

        
        u(-1) = \sum_{k=0}^{N-1} \hat{u}_k v_k(-1) = \hat{u}_{N-2} = a,
         
        

.. math::
   :label: eq:dirichletb

          
        u(+1) = \sum_{k=0}^{N-1} \hat{u}_k v_k(+1) = \hat{u}_{N-1} = b,
        
        

and, as such, we only have to solve for :math:`\{\hat{u}_k\}_{k=0}^{N-3}`, just like
for a problem with homogeneous boundary conditions.
We now formulate a variational problem using the Galerkin method: Find :math:`u \in V^N` such that

.. math::
   :label: eq:varform

        
        \int_{-1}^1 \nabla^2 u \, v \, w\, dx = \int_{-1}^1 f \, v\, w\, dx \quad \forall v \, \in \, V^N.  
        

The weighted integrals, weighted by :math:`w(x)`, are called inner products, and a common notation is

.. math::
   :label: _auto5

        
        \int_{-1}^1 u \, v \, w\, dx = \left( u, v\right)_w. 
        
        

The integral can either be computed exactly, or with quadrature. The advantage
of the latter is that it is generally faster, and that non-linear terms may be 
computed just as quickly as linear. For a linear problem, it does not make much
of a difference, if any at all. Approximating the integral with quadrature, we
obtain

.. math::
   :label: _auto6

        
        \int_{-1}^1 u \, v \, w\, dx \approx \left( u, v \right)_w^N,  
        
        

.. math::
   :label: _auto7

          
        \approx \sum_{j=0}^{N-1} u(x_j) v(x_j) w(x_j),
        
        

where :math:`w(x_j)` are quadrature weights. The quadrature points :math:`\{x_j\}_{j=0}^N` 
are specific to the chosen basis, and even within basis there are two different 
choices based on which quadrature rule is selected, either Gauss or Gauss-Lobatto.

Inserting for test and trialfunctions, we get the following bilinear form and
matrix :math:`A\in\mathbb{R}^{N-2\times N-2}` for the Laplacian (using the summation convention in step 2)

.. math::
        \begin{align*}
        \left( \nabla^2u, v \right)_w^N &= \left( \nabla^2\sum_{k=0}^{N-3}\hat{u}_k v_{k}, v_j \right)_w^N, \\ 
            &= \left(\nabla^2 v_{k}, v_j \right)_w^N \hat{u}_k, \\ 
            &= A_{jk} \hat{u}_k.
        \end{align*}

Note that the sum in :math:`A_{jk} \hat{u}_{k}` runs over :math:`k=0, 1, \ldots, N-3` since
the last two degrees of freedom already are known from Eq. :eq:`eq:dirichleta`
and :eq:`eq:dirichletb`, and the second derivatives of :math:`v_{N-2}` and :math:`v_{N-1}`
are zero.
The right hand side linear form and vector is computed as :math:`\tilde{f}_j = (f,
v_j)_w^N`, for :math:`j=0,1,\ldots, N-3`, where a tilde is used because this is not a complete transform of the function :math:`f`, but only an inner product. 

The linear system of equations to solve for the expansion coefficients of :math:`u(x)` is given as

.. math::
   :label: _auto8

        
        A_{jk} \hat{u}_k = \tilde{f}_j.
        
        

Now, when :math:`\hat{u}` is found by solving this linear system, it may be
transformed to real space :math:`u(x)` using :eq:`eq:u`, and here the contributions
from :math:`\hat{u}_{N-2}` and :math:`\hat{u}_{N-1}` must be accounted for. Note that the matrix
:math:`A_{jk}` (different for Legendre or Chebyshev) has a very special structure that
allows for a solution to be found very efficiently in order of :math:`\mathcal{O}(N)`
operations, see :cite:`shen1` and :cite:`shen95`. These solvers are implemented in
``shenfun`` for both bases.

Method of manufactured solutions
--------------------------------

In this demo we will use the method of manufactured
solutions to demonstrate spectral accuracy of the ``shenfun`` Dirichlet bases. To
this end we choose an analytical function that satisfies the given boundary
conditions:

.. math::
   :label: eq:u_e

        
        u_e(x) = \sin(k\pi x)(1-x^2) + a(1+x)/2 + b(1-x)/2, 
        

where :math:`k` is an integer and :math:`a` and :math:`b` are constants. Now, feeding :math:`u_e` through the Laplace operator, we see
that the last two linear terms disappear, whereas the first term results in
in

.. math::
   :label: _auto9

        
         \nabla^2 u_e(x) = \frac{d^2 u_e}{dx^2},  
        
        

.. math::
   :label: eq:solution

          
                          = -4k \pi x \cos(k\pi x) - 2\sin(k\pi x) - k^2 \pi^2 (1 -
        x^2) \sin(k \pi x).  
        

Now, setting :math:`f_e(x) = \nabla^2 u_e(x)` and solving for :math:`\nabla^2 u(x) = f_e(x)`, we can compare the numerical solution :math:`u(x)` with the analytical solution :math:`u_e(x)` and compute error norms.

Implementation
==============

Preamble
--------

We will solve the Poisson problem using the `shenfun <https://github.com/spectralDNS/shenfun>`__ Python module. The first thing needed
is then to import some of this module's functionality
plus some other helper modules, like `Numpy <https://numpy.org>`__ and `Sympy <https://sympy.org>`__:

.. code-block:: python

    from shenfun import inner, div, grad, TestFunction, TrialFunction, Function, \ 
        project, Dx, Array, chebyshev, legendre
    import numpy as np
    from sympy import symbols, cos, sin, exp, lambdify

We use ``Sympy`` for the manufactured solution and ``Numpy`` for testing.

Manufactured solution
---------------------

The exact solution :math:`u_e(x)` and the right hand side :math:`f_e(x)` are created using
``Sympy`` as follows 

.. code-block:: python

    a = -1
    b = 1
    k = 4
    x = symbols("x")
    ue = sin(k*np.pi*x)*(1-x**2) + a*(1 + x)/2. + b*(1 - x)/2.
    fe = ue.diff(x, 2)
    
    # Lambdify for faster evaluation
    ul = lambdify(x, ue, 'numpy')
    fl = lambdify(x, fe, 'numpy')

These solutions are now valid for a continuous domain. The next step is thus to discretize, using a discrete mesh :math:`\{x_j\}_{j=0}^{N-1}` and a finite number of basis functions. 

Note that it is not mandatory to use ``Sympy`` for the manufactured solution. Since the
solution is known :eq:`eq:solution`, we could just as well simply use ``Numpy``
to compute :math:`f_e` at :math:`\{x_j\}_{j=0}^{N-1}`. However, with ``Sympy`` it is much
easier to experiment and quickly change the solution.

Discretization
--------------

We create a basis with a given number of basis functions, and extract the computational mesh from the basis itself

.. code-block:: python

    N = 32
    SD = chebyshev.bases.ShenDirichletBasis(N, plan=True, bc=(a, b))
    #SD = legendre.bases.ShenDirichletBasis(N, plan=True, bc=(a, b))
    X = SD.mesh(N)

Note that we can either choose a Legendre or a Chebyshev basis. The keyword
``plan`` is used to tell the class :class:`shenfun.chebyshev.bases.ShenDirichletBasis` that it can 
go ahead and plan its transforms with `pyfftw <https://pyfftw.org>`__, because 
this basis will not be a part of a tensorproductspace, in which case the planning would need to wait.

Variational formulation
-----------------------

The variational problem :eq:`eq:varform` can be assembled using ``shenfun``'s
:class:`.TrialFunction`, :class:`.TestFunction` and :func:`.inner` functions.

.. code-block:: python

    u = TrialFunction(SD)
    v = TestFunction(SD)
    # Assemble left hand side matrix
    A = inner(v, div(grad(u)))
    # Assemble right hand side
    fj = fl(X)
    f_hat = Array(SD)
    f_hat = inner(v, fj, output_array=f_hat)

Solve linear equations
----------------------

Finally, solve linear equation system and transform solution from spectral :math:`\{\hat{u}_k\}_{k=0}^{N-1}` vector to the real space :math:`\{u(x_j)\}_{j=0}^N` 
and then check how the solution corresponds with the exact solution :math:`u_e`.

.. code-block:: python

    u_hat = A.solve(f_hat)
    uj = SD.backward(u_hat)
    ue = ul(X)
    print("Error=%2.16e" %(np.linalg.norm(uj-ue)))
    assert np.allclose(uj, ue)

Convergence test
----------------

A complete solver is given in Sec. :ref:`sec:complete`. This solver is created 
such that it takes in two commandline arguments and prints out the 
:math:`l_2`-errornorm of the solution in the end. We can use this to write a short 
script that performs a convergence test. The solver is run like

.. code-block:: text

    >>> python dirichlet_poisson1D.py 32 legendre
    Error=6.5955040031498912e-10

for a discretization of size :math:`N=32` and for the Legendre basis. Alternatively, change ``legendre`` to ``chebyshev`` for the Chebyshev basis.  

We set up the solver to run for a list of :math:`N=[12, 16, \ldots, 48]`, and collect
the errornorms in arrays to be plotted. Such a script can be easily created 
with the `subprocess <https://docs.python.org/3/library/subprocess.html>`__ module

.. code-block:: python

    import subprocess
    
    N = range(12, 50, 4)
    error = {}
    for basis in ('legendre', 'chebyshev'):
        error[basis] = []
        for i in range(len(N)):
            output = subprocess.check_output("python dirichlet_poisson1D.py {} {}".format(N[i], basis), shell=True)
            exec(output) # Error is printed as "Error=%2.16e"%(np.linalg.norm(uj-ua))
            error[basis].append(Error)

The error can be plotted using `matplotlib <https://matplotlib.org>`__, and the generated figure is shown in the summary's Fig. :ref:`fig:ct0`. The spectral convergence is evident and we can see that after :math:`N=40` roundoff errors dominate as the errornorm trails off around :math:`10^{-14}`.

.. code-block:: python

    import matplotlib.pyplot as plt
    plt.figure(figsize=(6, 4))
    for basis, col in zip(('legendre', 'chebyshev'), ('r', 'b')):
        plt.semilogy(N, error[basis], col, linewidth=2)
    plt.title('Convergence of Poisson solvers 1D')
    plt.xlabel('N')
    plt.ylabel('Error norm')
    plt.savefig('poisson1D_errornorm.png')
    plt.legend(('Legendre', 'Chebyshev'))
    plt.show()

.. FIGURE: [poisson1D_errornorm.png] Convergence test of Legendre and Chebyshev 1D Poisson solvers.

.. _sec:complete:

Complete solver
---------------
A complete solver, that can use either Legendre or Chebyshev bases, chosen as a
command-line argument, can be found `here <https://github.com/spectralDNS/shenfun/blob/master/demo/dirichlet_poisson1D.py>`__.

.. ======= Bibliography =======

.. % if FORMAT not in ("sphinx"):

.. [shen1]
   **J. Shen**. Efficient Spectral-Galerkin Method I. Direct Solvers of Second- and Fourth-Order Equations Using Legendre Polynomials,
   *SIAM Journal on Scientific Computing*,
   15(6),
   pp. 1489-1505,
   `doi: 10.1137/0915089 <http://dx.doi.org/10.1137/0915089>`__,
   1994,
   `https://doi.org/10.1137/0915089 <https://doi.org/10.1137/0915089>`_.

.. [shen95]
   **J. Shen**. Efficient Spectral-Galerkin Method II. Direct Solvers of Second- and Fourth-Order Equations Using Chebyshev Polynomials,
   *SIAM Journal on Scientific Computing*,
   16(1),
   pp. 74-87,
   `doi: 10.1137/0916006 <http://dx.doi.org/10.1137/0916006>`__,
   1995,
   `https://doi.org/10.1137/0916006 <https://doi.org/10.1137/0916006>`_.

.. % else:

.. .. bibliography:: ../papers.bib

.. :notcited:

.. % endif