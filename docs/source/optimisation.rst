..    Copyright 2019 United Kingdom Research and Innovation
      Copyright 2019 The University of Manchester

      Licensed under the Apache License, Version 2.0 (the "License");
      you may not use this file except in compliance with the License.
      You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      See the License for the specific language governing permissions and
      limitations under the License.

     Authors:
     CIL Developers, listed at: https://github.com/TomographicImaging/CIL/blob/master/NOTICE.txt

Optimisation framework
**********************
This package allows rapid prototyping of optimisation-based reconstruction problems, i.e. defining and solving different optimization problems to enforce different properties on the reconstructed image.

Firstly, it provides an object-oriented framework for defining mathematical operators and functions as well a collection of useful example operators and functions. Both smooth and non-smooth functions can be used.

Further, it provides a number of high-level generic implementations of optimisation algorithms to solve generically formulated optimisation problems constructed from operator and function objects.

The fundamental components are:

+ :code:`Operator`: A class specifying a (currently linear) operator.
+ :code:`Function`: A class specifying mathematical functions such as a least squares data fidelity.
+ :code:`Algorithm`: Implementation of an iterative optimisation algorithm to solve a particular generic optimisation problem. Algorithms are iterable Python object which can be run in a for loop. Can be stopped and warm restarted.



Algorithms (Deterministic)
==========================

A number of generic algorithm implementations are provided including
Gradient Descent (GD), Conjugate Gradient Least Squares (CGLS),
Simultaneous Iterative Reconstruction Technique (SIRT), Primal Dual Hybrid
Gradient (PDHG), Primal dual three-operator (PD3O),  Iterative Shrinkage Thresholding Algorithm (ISTA),
and Fast Iterative Shrinkage Thresholding Algorithm (FISTA).

An algorithm is designed for a particular generic optimisation problem accepts and number of
instances of :code:`Function` derived classes and/or :code:`Operator` derived classes as input to
define a specific instance of the generic optimisation problem to be solved.
They are iterable objects which can be run in a for loop.
The user can provide a stopping criterion different than the default.

New algorithms can be easily created by extending the :code:`Algorithm` class.
The user is required to implement only 4 methods: set_up, __init__, update and update_objective.

+ :code:`set_up` and :code:`__init__` are used to configure the algorithm
+ :code:`update` is the actual iteration updating the solution
+ :code:`update_objective` defines how the objective is calculated.

For example, the implementation of the update of the Gradient Descent
algorithm to minimise a Function will only be:

.. code-block :: python

    def update(self):
        self.x += -self.rate * self.objective_function.gradient(self.x)
    def update_objective(self):
        self.loss.append(self.objective_function(self.x))

The :code:`Algorithm` provides the infrastructure to continue iteration, to access the values of the
objective function in subsequent iterations, the time for each iteration, and to provide a nice
print to screen of the status of the optimisation.

Base class
----------
.. autoclass:: cil.optimisation.algorithms.Algorithm
   :members:
   :inherited-members:

GD
--
.. autoclass:: cil.optimisation.algorithms.GD
   :members:
   :inherited-members: run, update_objective_interval

CGLS
----
.. autoclass:: cil.optimisation.algorithms.CGLS
   :members:
   :inherited-members: run, update_objective_interval

SIRT
----
.. autoclass:: cil.optimisation.algorithms.SIRT
   :members: update, update_objective
   :inherited-members: run, update_objective_interval

ISTA/PGD
--------
The Iterative Soft Thresholding Algorithm (ISTA) is also known as Proximal Gradient Descent (PGD). Note that in CIL, :ref:`PGD<ISTA>` is an alias of `ISTA`. 

.. _ISTA:
.. autoclass:: cil.optimisation.algorithms.ISTA
   :members:
   :inherited-members: run, update_objective_interval


FISTA
-----
The Fast Iterative Soft Thresholding Algorithm (FISTA). 

.. _FISTA:
.. autoclass:: cil.optimisation.algorithms.FISTA
   :members:
   :inherited-members: run, update_objective_interval

APGD
-----
The Accelerated Proximal Gradient Descent Algorithm (APGD). This is an extension of the PGD/ISTA algorithm allowing you to either use a constant momemtum or a momentum that is updated at each iteration. 

.. autoclass:: cil.optimisation.algorithms.APGD
   :members:
   :inherited-members: run, update_objective_interval

Current options for are based on the scalar momentum, with base class:

.. autoclass:: cil.optimisation.algorithms.APGD.ScalarMomentumCoefficient 
   :members:

Implemented examples are: 

.. autoclass:: cil.optimisation.algorithms.APGD.ConstantMomentum
   :members:


.. autoclass:: cil.optimisation.algorithms.APGD.NesterovMomentum
   :members:


PDHG
----
.. autoclass:: cil.optimisation.algorithms.PDHG
   :members: update, set_step_sizes, update_step_sizes, update_objective
   :member-order: bysource
   :inherited-members: run, update_objective_interval

LADMM
-----
.. autoclass:: cil.optimisation.algorithms.LADMM
   :members:
   :inherited-members: run, update_objective_interval

PD3O
----
.. autoclass:: cil.optimisation.algorithms.PD3O
   :members:
   :inherited-members: run, update_objective_interval


Algorithms (Stochastic)
========================

Consider optimisation problems that take the form of a separable sum: 

.. math:: \min_{x} f(x)+g(x) = \min_{x} \sum_{i=0}^{n-1} f_{i}(x) + g(x) = \min_{x} (f_{0}(x) + f_{1}(x) + ... + f_{n-1}(x))+g(x)

where :math:`n` is the number of functions.  Where there is a large number of :math:`f_i` or their gradients are expensive to calculate, stochastic optimisation methods could prove more efficient.   
There is a growing range of Stochastic optimisation algorithms available with potential benefits of faster convergence in number of iterations or in computational cost. 
This is an area of continued development for CIL and, depending on  the properties of the :math:`f_i` and the regulariser :math:`g`, there is a range of different options for the user. 



SPDHG
-----
Stochastic Primal Dual Hybrid Gradient (SPDHG) is a stochastic version of PDHG and deals with optimisation problems of the form: 
    
    .. math::
    
      \min_{x} f(Kx) + g(x) = \min_{x} \sum f_i(K_i x) + g(x)

where :math:`f_i` and the regulariser :math:`g` need only be proper, convex and lower semi-continuous ( i.e. do not need to be differentiable). 
Each iteration considers just one index of the sum, potentially reducing computational cost. For more examples see our [user notebooks]( https://github.com/vais-ral/CIL-Demos/blob/master/Tomography/Simulated/Single%20Channel/PDHG_vs_SPDHG.py).


.. autoclass:: cil.optimisation.algorithms.SPDHG
   :members: update, set_step_sizes, set_step_sizes_from_ratio, update_objective
   :inherited-members: run, update_objective_interval


Approximate gradient methods
----------------------------------

Alternatively, consider that, in addition to the functions :math:`f_i` and the regulariser :math:`g` being proper, convex and lower semi-continuous, the :math:`f_i` are differentiable. In this case we consider stochastic methods that replace a gradient calculation in a deterministic algorithm with a, potentially cheaper to calculate, approximate gradient. 
For example, when :math:`g(x)=0`, the standard Gradient Descent algorithm utilises iterations of the form

   .. math::
      x_{k+1}=x_k-\alpha \nabla f(x_k) =x_k-\alpha \sum_{i=0}^{n-1}\nabla f_i(x_k).
:math:`\nabla f(x_k)=\sum_{i=0}^{n-1}\nabla f_i(x_k)` with :math:`n \nabla f_i(x_k)`, for an index :math:`i` which changes each iteration, leads to the well known stochastic gradient descent algorithm. 



Replacing, :math:`\nabla f(x_k)=\sum_{i=0}^{n-1}\nabla f_i(x_k)` with :math:`n \nabla f_i(x_k)`, for an index :math:`i` which changes each iteration, leads to the well known stochastic gradient descent algorithm. 

In addition, if :math:`g(x)\neq 0` and has a calculable proximal ( need not be differentiable) one can consider ISTA iterations: 

   .. math::
         x_{k+1}=prox_{\alpha g}(x_k-\alpha \nabla f(x_k) )=prox_{\alpha g}(x_k-\alpha \sum_{i=0}^{n-1}\nabla f_i(x_k))

and again replacing :math:`\nabla f(x_k)=\sum_{i=0}^{n-1}\nabla f_i(x_k)` with an approximate gradient. 

In a similar way, plugging approximate gradient calculations into deterministic algorithms can lead to a range of stochastic algorithms. In the following table, the left hand column has the approximate gradient function subclass, :ref:`Approximate Gradient base class` the header row has one of CIL's deterministic optimisation algorithm and the body of the table has the resulting stochastic algorithm.

+----------------+-------+------------+----------------+
|                | GD    | ISTA       | FISTA          |
+----------------+-------+------------+----------------+
| SGFunction     | SGD   | Prox-SGD   | Acc-Prox-SGD   |
+----------------+-------+------------+----------------+
| SAGFunction\  | SAG   | Prox-SAG   | Acc-Prox-SAG   |
+----------------+-------+------------+----------------+
| SAGAFunction\ | SAGA  | Prox-SAGA  | Acc-Prox-SAGA  |
+----------------+-------+------------+----------------+
| SVRGFunction\ | SVRG  | Prox-SVRG  | Acc-Prox-SVRG  |
+----------------+-------+------------+----------------+
| LSVRGFunction\| LSVRG | Prox-LSVRG | Acc-Prox-LSVRG |
+----------------+-------+------------+----------------+

\*In development 

The stochastic gradient functions can be found listed under functions in the documentation. 

Stochastic Gradient Descent Example
----------------------------------
The below is an example of Stochastic Gradient Descent built of the SGFunction and Gradient Descent algorithm:

.. code-block :: python

   from cil.optimisation.utilities import Sampler
   from cil.optimisation.algorithms import GD 
   from cil.optimisation.functions import LeastSquares, SGFunction
   from cil.utilities import dataexample
   from cil.plugins.astra.operators import ProjectionOperator
   
   # get the data  
   data = dataexample.SIMULATED_PARALLEL_BEAM_DATA.get()
   data.reorder('astra')
   data = data.get_slice(vertical='centre')

   # create the geometries 
   ag = data.geometry 
   ig = ag.get_ImageGeometry()

   # partition the data and build the projectors
   n_subsets = 10 
   partitioned_data = data.partition(n_subsets, 'sequential')
   A_partitioned = ProjectionOperator(ig, partitioned_data.geometry, device = "cpu")

   # create the list of functions for the stochastic sum 
   list_of_functions = [LeastSquares(Ai, b=bi) for Ai,bi in zip(A_partitioned, partitioned_data)]

   #define the sampler and the stochastic gradient function 
   sampler = Sampler.staggered(len(list_of_functions), stride=2)
   f = SGFunction(list_of_functions, sampler=sampler)  
   
   #set up and run the gradient descent algorithm 
   alg = GD(initial=ig.allocate(0), objective_function=f, step_size=1/f.L)
   alg.run(300)


Note
----
 All the approximate gradients written in CIL are of a similar order of magnitude to the full gradient calculation. For example, in the :code:`SGFunction` we approximate the full gradient by :math:`n\nabla f_i` for an index :math:`i` given by the sampler. 
 The multiplication by :math:`n` is a choice to more easily allow comparisons between stochastic and non-stochastic methods and between stochastic methods with varying numbers of subsets.
 The multiplication ensures that the (SAGA, SGD, and SVRG  and LSVRG) approximate gradients are an unbiased estimator of the full gradient ie :math:`\mathbb{E}\left[\tilde\nabla f(x)\right] =\nabla f(x)`.
  This has an implication when choosing step sizes. For example, a suitable step size for GD with a SGFunction could be 
  :math:`\propto 1/(L_{max}*n)`, where :math:`L_{max}` is the largest Lipschitz constant of the list of functions in the SGFunction and the additional factor of  :math:`n` reflects this multiplication by  :math:`n` in the approximate gradient. 

  
Memory requirements
-------------------
Note that the approximate gradient methods have different memory requirements:
+ The `SGFunction` has the same requirements as a `SumFunction`, so no increased memory usage
+ `SAGFunction` and `SAGAFunction` both store `n+3` times the image size in memory to store the last calculated gradient for each function in the sum and for intermediary calculations. 
+ `SVRGFunction` and `LSVRGFunction` with the default `store_gradients = False` store 4 times the image size in memory, including the "snapshot" point and gradient. If `store_gradients = True`, some computational effort is saved, at the expensive of stored memory `n+4` times the image size.  


Operators
=========
The two most important methods are :code:`direct` and :code:`adjoint`
methods that describe the result of applying the operator, and its
adjoint respectively, onto a compatible :code:`DataContainer` input.
The output is another :code:`DataContainer` object or subclass
hereof. An important special case is to represent the tomographic
forward and backprojection operations.


Operator base classes
---------------------

All operators extend the :code:`Operator` class. A special class is the :code:`LinearOperator`
which represents an operator for which the :code:`adjoint` operation is defined.
A :code:`ScaledOperator` represents the multiplication of any operator with a scalar.

.. autoclass:: cil.optimisation.operators.Operator
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.operators.LinearOperator
   :members:


.. autoclass:: cil.optimisation.operators.ScaledOperator
   :members:


.. autoclass:: cil.optimisation.operators.CompositionOperator
   :members:


.. autoclass:: cil.optimisation.operators.DiagonalOperator
   :members:


.. autoclass:: cil.optimisation.operators.ChannelwiseOperator
   :members:


.. autoclass:: cil.optimisation.operators.SumOperator
   :members:


Trivial operators
-----------------

Trivial operators are the following.

.. autoclass:: cil.optimisation.operators.IdentityOperator
   :members:


.. autoclass:: cil.optimisation.operators.ZeroOperator
   :members:


.. autoclass:: cil.optimisation.operators.MatrixOperator
   :members:


.. autoclass:: cil.optimisation.operators.MaskOperator
   :members:


.. autoclass:: cil.optimisation.operators.ProjectionMap
   :members:

GradientOperator
-----------------

.. autoclass:: cil.optimisation.operators.GradientOperator
   :members:


.. autoclass:: cil.optimisation.operators.FiniteDifferenceOperator
   :members:

.. autoclass:: cil.optimisation.operators.SparseFiniteDifferenceOperator
   :members:

.. autoclass:: cil.optimisation.operators.SymmetrisedGradientOperator
   :members:


WaveletOperator
---------------
We utilise PyWavelets (https://pywavelets.readthedocs.io/en/latest/index.html) to build wavelet operators in CIL:

.. autoclass:: cil.optimisation.operators.WaveletOperator
   :members:




Functions
=========

A :code:`Function` represents a mathematical function of one or more inputs
and is intended to accept :code:`DataContainers` as input as well as any
additional parameters.

Fixed parameters can be passed in during the creation of the function object.
The methods of the function reflect the properties of it, for example, if the function
represented is differentiable the function should contain a method :code:`gradient`
which should return the gradient of the function evaluated at an input point.
If the function is not differentiable but allows a simple proximal operator,
the method :code:`proximal` should return the proximal operator evaluated at an
input point. The function value is evaluated by calling the function itself,
e.g. :code:`f(x)` for a :code:`Function f` and input point :code:`x`.


Base classes
------------

.. autoclass:: cil.optimisation.functions.Function
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.SumFunction
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.ScaledFunction
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.SumScalarFunction
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.TranslateFunction
   :members:
   :inherited-members:

Simple functions
----------------
.. autoclass:: cil.optimisation.functions.ConstantFunction
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.ZeroFunction
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.Rosenbrock
   :members:
   :inherited-members:

Composition of operator and a function
--------------------------------------

This class allows the user to write a function which does the following:

.. math::

  F ( x ) = G ( Ax )

where :math:`A` is an operator. For instance the least squares function can
be expressed as

.. math::

  F(x) = || Ax - b ||^2_2 \qquad \text{where} \qquad G(y) = || y - b ||^2_2

.. code-block :: python

  F1 = Norm2Sq(A, b)
  # or equivalently
  F2 = OperatorCompositionFunction(L2NormSquared(b=b), A)


.. autoclass:: cil.optimisation.functions.OperatorCompositionFunction
  :members:
  :inherited-members:

Indicator box
-------------

.. autoclass:: cil.optimisation.functions.IndicatorBox
   :members:
   :inherited-members:


KullbackLeibler
---------------

.. autoclass:: cil.optimisation.functions.KullbackLeibler
   :members:
   :inherited-members:

L1 Norm
-------

.. autoclass:: cil.optimisation.functions.L1Norm
   :members:
   :inherited-members:

L2 Norm Squared
-----------------------
.. l2norm:

.. autoclass:: cil.optimisation.functions.L2NormSquared
   :members:
   :inherited-members:

.. autoclass:: cil.optimisation.functions.WeightedL2NormSquared
   :members:
   :inherited-members:


Least Squares
-------------

.. autoclass:: cil.optimisation.functions.LeastSquares
   :members:
   :inherited-members:


L1 Sparsity
----------
.. autoclass:: cil.optimisation.functions.L1Sparsity
   :members:
   :inherited-members:


Mixed L21 norm
--------------

.. autoclass:: cil.optimisation.functions.MixedL21Norm
   :members:
   :inherited-members:

Smooth Mixed L21 norm
---------------------

.. autoclass:: cil.optimisation.functions.SmoothMixedL21Norm
   :members:
   :inherited-members:

Mixed L11 norm
---------------------

.. autoclass:: cil.optimisation.functions.MixedL11Norm
   :members:
   :inherited-members:

Total variation
---------------

.. autoclass:: cil.optimisation.functions.TotalVariation
   :members:
   :inherited-members:

Function of Absolute Value 
--------------------------

.. autoclass:: cil.optimisation.functions.FunctionOfAbs
   :members:
   :inherited-members:


Approximate Gradient base class 
--------------------------------

.. autoclass:: cil.optimisation.functions.ApproximateGradientSumFunction 
   :members:
   :inherited-members:
   

Stochastic Gradient function 
-----------------------------

.. autoclass:: cil.optimisation.functions.SGFunction 
   :members:
   :inherited-members:

SAG function
-------------

.. autoclass:: cil.optimisation.functions.SAGFunction 
   :members:
   :inherited-members:

SAGA function
--------------

.. autoclass:: cil.optimisation.functions.SAGAFunction 
   :members:
   :inherited-members:



Stochastic Variance Reduced Gradient Function 
----------------------------------------------
.. autoclass:: cil.optimisation.functions.SVRGFunction 
   :members:
   :inherited-members:


Loopless Stochastic Variance Reduced Gradient Function 
----------------------------------------------
.. autoclass:: cil.optimisation.functions.LSVRGFunction 
   :members:
   :inherited-members:



Utilities
=========

Contains utilities for the CIL optimisation framework.

Samplers
--------

Here, we define samplers that select from a list of indices {0, 1, …, N-1} either randomly or by some deterministic pattern.
The :code:`cil.optimisation.utilities.sampler` class defines a function :code:`next()` which gives the next sample. It also has utility to :code:`get_samples` to access which samples have or will be drawn.

For ease of use we provide the following static methods in `cil.optimisation.utilities.sampler` that allow you to configure your sampler object rather than initialising the classes directly:

.. automethod:: cil.optimisation.utilities.Sampler.from_function

.. automethod:: cil.optimisation.utilities.Sampler.sequential

.. automethod:: cil.optimisation.utilities.Sampler.staggered

.. automethod:: cil.optimisation.utilities.Sampler.herman_meyer

.. automethod:: cil.optimisation.utilities.Sampler.random_with_replacement

.. automethod:: cil.optimisation.utilities.Sampler.random_without_replacement


They will all instantiate a Sampler defined in the following class:

.. autoclass:: cil.optimisation.utilities.Sampler
   

The random samplers are instantiated from a random sampling class which is a child class of  `cil.optimisation.utilities.sampler` and provides options for sampling with and without replacement:

.. autoclass:: cil.optimisation.utilities.SamplerRandom
   

Callbacks
---------

A list of :code:`Callback` s to be executed each iteration can be passed to `Algorithm`'s :code:`run` method.

.. code-block :: python

   from cil.optimisation.utilities.callbacks import LogfileCallback
   ...
   algorithm.run(..., callbacks=[LogfileCallback("log.txt")])

.. autoclass:: cil.optimisation.utilities.callbacks.Callback
   :members:

Built-in callbacks include:

.. autoclass:: cil.optimisation.utilities.callbacks.ProgressCallback
   :members:

.. autoclass:: cil.optimisation.utilities.callbacks.TextProgressCallback
   :members:

.. autoclass:: cil.optimisation.utilities.callbacks.LogfileCallback
   :members:

.. autoclass:: cil.optimisation.utilities.callbacks.EarlyStoppingObjectiveValue
   :members:

.. autoclass:: cil.optimisation.utilities.callbacks.CGLSEarlyStopping
   :members:

Users can also write custom callbacks.

Below is an example of a custom callback implementing early stopping.
In each iteration of the :code:`TestAlgo`, the objective :math:`x` is reduced by :math:`5`. The :code:`EarlyStopping` callback terminates the algorithm when :math:`x \le -15`. The algorithm thus terminates after :math:`3` iterations.

.. code:: python

   from cil.optimisation.algorithms import Algorithm
   from cil.optimisation.utilities import callbacks

   class TestAlgo(Algorithm):
       def __init__(self, *args, **kwargs):
           self.x = 0
           super().__init__(*args, **kwargs)
           self.configured = True

       def update(self):
           self.x -= 5

       def update_objective(self):
           self.loss.append(2 ** self.x)

   class EarlyStopping(callbacks.Callback):
       def __call__(self, algorithm: Algorithm):
           if algorithm.x <= -15:  # arbitrary stopping criterion
               raise StopIteration

   algo = TestAlgo()
   algo.run(20, callbacks=[callbacks.ProgressCallback(), EarlyStopping()])


.. code:: raw

   Output:
    15%|███                 | 3/20 [00:00<00:00, 11770.73it/s, objective=3.05e-5]


Step size methods 
------------------
A step size method is a class which acts on an algorithm and can be passed to  `cil.optimisation.algorithm.GD`, `cil.optimisation.algorithm.ISTA`  `cil.optimisation.algorithm.FISTA` and it's method `get_step_size` is called after the calculation of the gradient before the gradient descent step is taken. It outputs a float value to be used as the step-size. 

Currently in CIL we have a base class:

.. autoclass:: cil.optimisation.utilities.StepSizeMethods.StepSizeRule
   :members:

We also have a number of example classes:

.. autoclass:: cil.optimisation.utilities.StepSizeMethods.ConstantStepSize
   :members:

.. autoclass:: cil.optimisation.utilities.StepSizeMethods.ArmijoStepSizeRule
   :members:

.. autoclass:: cil.optimisation.utilities.StepSizeMethods.BarzilaiBorweinStepSizeRule
   :members:



Preconditioners
----------------
A preconditioner is a class which acts on an algorithm and can be passed to  `cil.optimisation.algorithm.GD`, `cil.optimisation.algorithm.ISTA` or `cil.optimisation.algorithm.FISTA` and it's method `apply` is called after the calculation of the gradient before the gradient descent step is taken. It modifies and returns a passed `gradient`. 

Currently in CIL we have a base class:

.. autoclass:: cil.optimisation.utilities.preconditioner.Preconditioner
   :members:

We also have a number of already provided pre-conditioners

.. autoclass:: cil.optimisation.utilities.preconditioner.Sensitivity
   :members:

.. autoclass:: cil.optimisation.utilities.preconditioner.AdaptiveSensitivity
   :members:

Block Framework
***************

To be able to express more advanced optimisation problems we developed the
`Block Framework`_, which provides a generic strategy to treat variational
problems in the following form:

.. math::
    \min \text{Regulariser} + \text{Fidelity}

The block framework consists of:

+ `BlockDataContainer`_
+ `BlockFunction`_
+ `BlockOperator`_




The block framework allows writing more advanced optimisation problems. Consider the typical
`Tikhonov regularisation <https://en.wikipedia.org/wiki/Tikhonov_regularization>`_:

.. math::

  \underset{u}{\mathrm{argmin}}\begin{Vmatrix}A u - b \end{Vmatrix}^2_2 + \alpha^2\|Lu\|^2_2

where,

* :math:`A` is the projection operator
* :math:`b` is the acquired data
* :math:`u` is the unknown image to be solved for
* :math:`\alpha` is the regularisation parameter
* :math:`L` is a regularisation operator

The first term measures the fidelity of the solution to the data. The second term measures the
fidelity to the prior knowledge we have imposed on the system, operator :math:`L`.

This can be re-written equivalently in the block matrix form:

.. math::
  \underset{u}{\mathrm{argmin}}\begin{Vmatrix}\binom{A}{\alpha L} u - \binom{b}{0}\end{Vmatrix}^2_2

With the definitions:

* :math:`\tilde{A} = \binom{A}{\alpha L}`
* :math:`\tilde{b} = \binom{b}{0}`

this can now be recognised as a least squares problem which can be solved by any algorithm in the :code:`cil.optimisation`
which can solve least squares problem, e.g. CGLS.

.. math::

  \underset{u}{\mathrm{argmin}}\begin{Vmatrix}\tilde{A} u - \tilde{b}\end{Vmatrix}^2_2

To be able to express our optimisation problems in the matrix form above, we developed the so-called,
Block Framework comprising 4 main actors: :code:`BlockGeometry`, :code:`BlockDataContainer`,
:code:`BlockFunction` and :code:`BlockOperator`.



BlockDataContainer
==================

`BlockDataContainer`_ holds `DataContainer`_ as column vector. It is possible to
do basic algebra between `BlockDataContainer`_ s and with numbers, list or numpy arrays.

.. math::

  x = [x_{1}, x_{2} ]\in (X_{1}\times X_{2})

  y = [y_{1}, y_{2}, y_{3} ]\in(Y_{1}\times Y_{2} \times Y_{3})


.. autoclass:: cil.framework.BlockDataContainer
   :members:
   :special-members:


Block Function
==============

`BlockFunction`_ acts on `BlockDataContainer`_ as a separable sum function:

      .. math::

          f = [f_1,...,f_n] \newline

          f([x_1,...,x_n]) = f_1(x_1) +  .... + f_n(x_n)


.. math::

  Y = \begin{bmatrix}
  y_{1}\\
  y_{2}\\
  y_{3}\\
  \end{bmatrix}, \quad  F  = [ f_{1}, f_{2}, f_{3} ]

  F(Y) : = f_{1}(y_{1}) + f_{2}(y_{2}) + f_{3}(y_{3})


.. autoclass:: cil.optimisation.functions.BlockFunction
   :members:
   :special-members:

Block Operator
==============

`BlockOperator`_ represent a block matrix with operators

.. math::
  K = \begin{bmatrix}
      A_{1} & A_{2} \\
      A_{3} & A_{4} \\
      A_{5} & A_{6}
 \end{bmatrix}_{(3,2)} *  \quad \underbrace{\begin{bmatrix}
 x_{1} \\
 x_{2}
 \end{bmatrix}_{(2,1)}}_{\textbf{x}} =  \begin{bmatrix}
 A_{1}x_{1}  + A_{2}x_{2}\\
 A_{3}x_{1}  + A_{4}x_{2}\\
 A_{5}x_{1}  + A_{6}x_{2}\\
 \end{bmatrix}_{(3,1)} =  \begin{bmatrix}
 y_{1}\\
 y_{2}\\
 y_{3}
 \end{bmatrix}_{(3,1)} = \textbf{y}

Column: Share the same domains :math:`X_{1}, X_{2}`

Rows: Share the same ranges :math:`Y_{1}, Y_{2}, Y_{3}`

.. math::
 K : (X_{1}\times X_{2}) \rightarrow (Y_{1}\times Y_{2} \times Y_{3})

:math:`A_{1}, A_{3}, A_{5}`: share the same domain :math:`X_{1}` and
:math:`A_{2}, A_{4}, A_{6}`: share the same domain :math:`X_{2}`

.. math::

 A_{1}: X_{1} \rightarrow Y_{1} \\
 A_{3}: X_{1} \rightarrow Y_{2} \\
 A_{5}: X_{1} \rightarrow Y_{3} \\
 A_{2}: X_{2} \rightarrow Y_{1} \\
 A_{4}: X_{2} \rightarrow Y_{2} \\
 A_{6}: X_{2} \rightarrow Y_{3}

For instance with these ingredients one may write the following objective
function,

.. math::
   \alpha ||\nabla u||_{2,1} + ||u - g||_2^2

where :math:`g` represent the measured values, :math:`u` the solution
:math:`\nabla` is the gradient operator, :math:`|| ~~ ||_{2,1}` is a norm for
the output of the gradient operator and :math:`|| x-g ||^2_2` is
least squares fidelity function as

.. math::
   K = \begin{bmatrix}
           \nabla \\
           \mathbb{1}
         \end{bmatrix}

 F(x) = \Big[ \alpha \lVert ~x~ \rVert_{2,1} ~~ , ~~ || x - g||_2^2 \Big]

 w = [ u ]

Then we have rewritten the problem as

.. math::
  F(Kw) =   \alpha \left\lVert \nabla u \right\rVert_{2,1} + ||u-g||^2_2

Which in Python would be like

.. code-block:: python

   op1 = GradientOperator(ig, correlation=GradientOperator.CORRELATION_SPACE)
   op2 = IdentityOperator(ig, ag)

   # Create BlockOperator
   K = BlockOperator(op1, op2, shape=(2,1) )

   # Create functions
   F = BlockFunction(alpha * MixedL21Norm(), 0.5 * L2NormSquared(b=noisy_data))


.. autoclass:: cil.optimisation.operators.BlockOperator
   :members:
   :special-members:


:ref:`Return Home <mastertoc>`

.. _BlockDataContainer: #blockdatacontainer
.. _DataContainer: ../framework/#datacontainer
.. _BlockFunction: #block-function
.. _BlockOperator: #block-operator




References
----------

.. bibliography::
