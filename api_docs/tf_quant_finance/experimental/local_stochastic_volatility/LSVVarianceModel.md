<!--
This file is generated by a tool. Do not edit directly.
For open-source contributions the docs will be updated automatically.
-->

*Last updated: 2021-01-13.*

<div itemscope itemtype="http://developers.google.com/ReferenceObject">
<meta itemprop="name" content="tf_quant_finance.experimental.local_stochastic_volatility.LSVVarianceModel" />
<meta itemprop="path" content="Stable" />
<meta itemprop="property" content="__init__"/>
<meta itemprop="property" content="dim"/>
<meta itemprop="property" content="drift_fn"/>
<meta itemprop="property" content="dtype"/>
<meta itemprop="property" content="fd_solver_backward"/>
<meta itemprop="property" content="fd_solver_forward"/>
<meta itemprop="property" content="name"/>
<meta itemprop="property" content="sample_paths"/>
<meta itemprop="property" content="volatility_fn"/>
</div>

# tf_quant_finance.experimental.local_stochastic_volatility.LSVVarianceModel

<!-- Insert buttons and diff -->

<table class="tfo-notebook-buttons tfo-api" align="left">
</table>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/experimental/local_stochastic_volatility/lsv_variance_model.py">View source</a>



Implements Heston like variance process for the LSV models.

Inherits From: [`GenericItoProcess`](../../../tf_quant_finance/models/GenericItoProcess.md)

```python
tf_quant_finance.experimental.local_stochastic_volatility.LSVVarianceModel(
    k, m, alpha, dtype=None
)
```



<!-- Placeholder for "Used in" -->

Local stochastic volatility (LSV) models assume that the spot price of an
asset follows the following stochastic differential equation under the risk
neutral measure [1]:

```None
  dS / S =  (r - d) dt + sqrt(v) * L(t, S(t)) * dW_s
  dv = a(t, v) dt + b(t, v) dW_v
  E[dW_s * dW_v] = rho dt
```
where `r` and `d` denote the risk free interest rate and dividend yield
respectively. `S` is the spot price, `v` denotes the stochastic variance
and the function `L(t, S)` is the leverage function which is calibrated
using the volatility smile data. The functions `a(t, v)` and `b(t, v)` denote
the drift and volitility of the stochastic process for the variance and `rho`
denotes the instantaneous correlation between the spot and the variance
process. LSV models thus combine the local volatility dynamics with
stochastic volatility.

Using the relationship between the local volatility and the expectation of
future instantaneous variance, leverage function can be computed as follows
[2]:

```
sigma(T,K)^2 = L(T,K)^2 * E[v(T)|S(T)=K]
```
where the local volatility function `sigma(T,K)` can be computed using the
Dupire's formula.

The `LSVVarianceModel` class implements Heston like mean-reverting model
for the dynamics of the variance process. The variance model dynamics is
as follows:

```None
  dv = k * (m - v) dt + alpha * sqrt(v) * dW
```

#### References:
  [1]: Iain J. Clark. Foreign exchange option pricing - A Practitioner's
  guide. Chapter 5. 2011.
  [2]: I. Gyongy. Mimicking the one-dimensional marginal distributions of
  processes having an ito differential. Probability Theory and Related
  Fields, 71, 1986.

## Methods

<h3 id="dim"><code>dim</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
dim()
```

The dimension of the process.


<h3 id="drift_fn"><code>drift_fn</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
drift_fn()
```

Python callable calculating instantaneous drift.

The callable should accept two real `Tensor` arguments of the same dtype.
The first argument is the scalar time t, the second argument is the value of
Ito process X - `Tensor` of shape `batch_shape + [dim]`. The result is the
value of drift a(t, X). The return value of the callable is a real `Tensor`
of the same dtype as the input arguments and of shape `batch_shape + [dim]`.

#### Returns:

The instantaneous drift rate callable.


<h3 id="dtype"><code>dtype</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
dtype()
```

The data type of process realizations.


<h3 id="fd_solver_backward"><code>fd_solver_backward</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
fd_solver_backward(
    start_time, end_time, coord_grid, values_grid, discounting=None,
    one_step_fn=None, boundary_conditions=None, start_step_count=0, num_steps=None,
    time_step=None, values_transform_fn=None, dtype=None, name=None, **kwargs
)
```

Returns a solver for Feynman-Kac PDE associated to the process.

This method applies a finite difference method to solve the final value
problem as it appears in the Feynman-Kac formula associated to this Ito
process. The Feynman-Kac PDE is closely related to the backward Kolomogorov
equation associated to the stochastic process and allows for the inclusion
of a discounting function.

For more details of the Feynman-Kac theorem see [1]. The PDE solved by this
method is:

```None
  V_t + Sum[mu_i(t, x) V_i, 1<=i<=n] +
    (1/2) Sum[ D_{ij} V_{ij}, 1 <= i,j <= n] - r(t, x) V = 0
```

In the above, `V_t` is the derivative of `V` with respect to `t`,
`V_i` is the partial derivative with respect to `x_i` and `V_{ij}` the
(mixed) partial derivative with respect to `x_i` and `x_j`. `mu_i` is the
drift of this process and `D_{ij}` are the components of the diffusion
tensor:

```None
  D_{ij}(t,x) = (Sigma(t,x) . Transpose[Sigma(t,x)])_{ij}
```

This method evolves a spatially discretized solution of the above PDE from
time `t0` to time `t1 < t0` (i.e. backwards in time).
The solution `V(t,x)` is assumed to be discretized on an `n`-dimensional
rectangular grid. A rectangular grid, G, in n-dimensions may be described
by specifying the coordinates of the points along each axis. For example,
a 2 x 4 grid in two dimensions can be specified by taking the cartesian
product of [1, 3] and [5, 6, 7, 8] to yield the grid points with
coordinates: `[(1, 5), (1, 6), (1, 7), (1, 8), (3, 5) ... (3, 8)]`.

This method allows batching of solutions. In this context, batching means
the ability to represent and evolve multiple independent functions `V`
(e.g. V1, V2 ...) simultaneously. A single discretized solution is specified
by stating its values at each grid point. This can be represented as a
`Tensor` of shape [d1, d2, ... dn] where di is the grid size along the `i`th
axis. A batch of such solutions is represented by a `Tensor` of shape:
[K, d1, d2, ... dn] where `K` is the batch size. This method only requires
that the input parameter `values_grid` be broadcastable with shape
[K, d1, ... dn].

The evolution of the solution from `t0` to `t1` is often done by
discretizing the differential equation to a difference equation along
the spatial and temporal axes. The temporal discretization is given by a
(sequence of) time steps [dt_1, dt_2, ... dt_k] such that the sum of the
time steps is equal to the total time step `t0 - t1`. If a uniform time
step is used, it may equivalently be specified by stating the number of
steps (n_steps) to take. This method provides both options via the
`time_step` and `num_steps` parameters. However, not all methods need
discretization along time direction (e.g. method of lines) so this argument
may not be applicable to some implementations.

The workhorse of this method is the `one_step_fn`. For the commonly used
methods, see functions in <a href="../../../tf_quant_finance/math/pde/steppers.md"><code>math.pde.steppers</code></a> module.

The mapping between the arguments of this method and the above
equation are described in the Args section below.

For a simple instructive example of implementation of this method, see
<a href="../../../tf_quant_finance/models/GenericItoProcess.md#fd_solver_backward"><code>models.GenericItoProcess.fd_solver_backward</code></a>.

TODO(b/142309558): Complete documentation.

#### Args:


* <b>`start_time`</b>: Real positive scalar `Tensor`. The start time of the grid.
  Corresponds to time `t0` above.
* <b>`end_time`</b>: Real scalar `Tensor` smaller than the `start_time` and greater
  than zero. The time to step back to. Corresponds to time `t1` above.
* <b>`coord_grid`</b>: List of `n` rank 1 real `Tensor`s. `n` is the dimension of the
  domain. The i-th `Tensor` has shape, `[d_i]` where `d_i` is the size of
  the grid along axis `i`. The coordinates of the grid points. Corresponds
  to the spatial grid `G` above.
* <b>`values_grid`</b>: Real `Tensor` containing the function values at time
  `start_time` which have to be stepped back to time `end_time`. The shape
  of the `Tensor` must broadcast with `[K, d_1, d_2, ..., d_n]`. The first
  axis of size `K` is the values batch dimension and allows multiple
  functions (with potentially different boundary/final conditions) to be
  stepped back simultaneously.
* <b>`discounting`</b>: Callable corresponding to `r(t,x)` above. If not supplied,
  zero discounting is assumed.
* <b>`one_step_fn`</b>: The transition kernel. A callable that consumes the following
  arguments by keyword:
    1. 'time': Current time
    2. 'next_time': The next time to step to. For the backwards in time
      evolution, this time will be smaller than the current time.
    3. 'coord_grid': The coordinate grid.
    4. 'values_grid': The values grid.
    5. 'boundary_conditions': The boundary conditions.
    6. 'quadratic_coeff': A callable returning the quadratic coefficients
      of the PDE (i.e. `(1/2)D_{ij}(t, x)` above). The callable accepts
      the time and  coordinate grid as keyword arguments and returns a
      `Tensor` with shape that broadcasts with `[dim, dim]`.
    7. 'linear_coeff': A callable returning the linear coefficients of the
      PDE (i.e. `mu_i(t, x)` above). Accepts time and coordinate grid as
      keyword arguments and returns a `Tensor` with shape that broadcasts
      with `[dim]`.
    8. 'constant_coeff': A callable returning the coefficient of the
      linear homogenous term (i.e. `r(t,x)` above). Same spec as above.
      The `one_step_fn` callable returns a 2-tuple containing the next
      coordinate grid, next values grid.
* <b>`boundary_conditions`</b>: A list of size `dim` containing boundary conditions.
  The i'th element of the list is a 2-tuple containing the lower and upper
  boundary condition for the boundary along the i`th axis.
* <b>`start_step_count`</b>: Scalar integer `Tensor`. Initial value for the number of
  time steps performed.
  Default value: 0 (i.e. no previous steps performed).
* <b>`num_steps`</b>: Positive int scalar `Tensor`. The number of time steps to take
  when moving from `start_time` to `end_time`. Either this argument or the
  `time_step` argument must be supplied (but not both). If num steps is
  `k>=1`, uniform time steps of size `(t0 - t1)/k` are taken to evolve the
  solution from `t0` to `t1`. Corresponds to the `n_steps` parameter
  above.
* <b>`time_step`</b>: The time step to take. Either this argument or the `num_steps`
  argument must be supplied (but not both). The type of this argument may
  be one of the following (in order of generality): (a) None in which case
    `num_steps` must be supplied. (b) A positive real scalar `Tensor`. The
    maximum time step to take. If the value of this argument is `dt`, then
    the total number of steps taken is N = (t0 - t1) / dt rounded up to
    the nearest integer. The first N-1 steps are of size dt and the last
    step is of size `t0 - t1 - (N-1) * dt`. (c) A callable accepting the
    current time and returning the size of the step to take. The input and
    the output are real scalar `Tensor`s.
* <b>`values_transform_fn`</b>: An optional callable applied to transform the
  solution values at each time step. The callable is invoked after the
  time step has been performed. The callable should accept the time of the
  grid, the coordinate grid and the values grid and should return the
  values grid. All input arguments to be passed by keyword.
* <b>`dtype`</b>: The dtype to use.
* <b>`name`</b>: The name to give to the ops.
  Default value: None which means `solve_backward` is used.
* <b>`**kwargs`</b>: Additional keyword args:
  (1) pde_solver_fn: Function to solve the PDE that accepts all the above
    arguments by name and returns the same tuple object as required below.
    Defaults to `tff.math.pde.fd_solvers.solve_backward`.


#### Returns:

A tuple object containing at least the following attributes:
  final_values_grid: A `Tensor` of same shape and dtype as `values_grid`.
    Contains the final state of the values grid at time `end_time`.
  final_coord_grid: A list of `Tensor`s of the same specification as
    the input `coord_grid`. Final state of the coordinate grid at time
    `end_time`.
  step_count: The total step count (i.e. the sum of the `start_step_count`
    and the number of steps performed in this call.).
  final_time: The final time at which the evolution stopped. This value
    is given by `max(min(end_time, start_time), 0)`.


<h3 id="fd_solver_forward"><code>fd_solver_forward</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
fd_solver_forward(
    start_time, end_time, coord_grid, values_grid, one_step_fn=None,
    boundary_conditions=None, start_step_count=0, num_steps=None, time_step=None,
    values_transform_fn=None, dtype=None, name=None, **kwargs
)
```

Returns a solver for the Fokker Plank equation of this process.

The Fokker Plank equation (also known as the Kolmogorov Forward equation)
associated to this Ito process is given by:

```None
  V_t + Sum[(mu_i(t, x) V)_i, 1<=i<=n]
    - (1/2) Sum[ (D_{ij} V)_{ij}, 1 <= i,j <= n] = 0
```

with the initial value condition $$V(0, x) = u(x)$$.

This method evolves a spatially discretized solution of the above PDE from
time `t0` to time `t1 > t0` (i.e. forwards in time).
The solution `V(t,x)` is assumed to be discretized on an `n`-dimensional
rectangular grid. A rectangular grid, G, in n-dimensions may be described
by specifying the coordinates of the points along each axis. For example,
a 2 x 4 grid in two dimensions can be specified by taking the cartesian
product of [1, 3] and [5, 6, 7, 8] to yield the grid points with
coordinates: `[(1, 5), (1, 6), (1, 7), (1, 8), (3, 5) ... (3, 8)]`.

Batching of solutions is supported. In this context, batching means
the ability to represent and evolve multiple independent functions `V`
(e.g. V1, V2 ...) simultaneously. A single discretized solution is specified
by stating its values at each grid point. This can be represented as a
`Tensor` of shape [d1, d2, ... dn] where di is the grid size along the `i`th
axis. A batch of such solutions is represented by a `Tensor` of shape:
[K, d1, d2, ... dn] where `K` is the batch size. This method only requires
that the input parameter `values_grid` be broadcastable with shape
[K, d1, ... dn].

The evolution of the solution from `t0` to `t1` is often done by
discretizing the differential equation to a difference equation along
the spatial and temporal axes. The temporal discretization is given by a
(sequence of) time steps [dt_1, dt_2, ... dt_k] such that the sum of the
time steps is equal to the total time step `t1 - t0`. If a uniform time
step is used, it may equivalently be specified by stating the number of
steps (n_steps) to take. This method provides both options via the
`time_step` and `num_steps` parameters. However, not all methods need
discretization along time direction (e.g. method of lines) so this argument
may not be applicable to some implementations.

The workhorse of this method is the `one_step_fn`. For the commonly used
methods, see functions in <a href="../../../tf_quant_finance/math/pde/steppers.md"><code>math.pde.steppers</code></a> module.

The mapping between the arguments of this method and the above
equation are described in the Args section below.

For a simple instructive example of implementation of this method, see
<a href="../../../tf_quant_finance/models/GenericItoProcess.md#fd_solver_forward"><code>models.GenericItoProcess.fd_solver_forward</code></a>.

TODO(b/142309558): Complete documentation.

#### Args:


* <b>`start_time`</b>: Real positive scalar `Tensor`. The start time of the grid.
  Corresponds to time `t0` above.
* <b>`end_time`</b>: Real scalar `Tensor` smaller than the `start_time` and greater
  than zero. The time to step back to. Corresponds to time `t1` above.
* <b>`coord_grid`</b>: List of `n` rank 1 real `Tensor`s. `n` is the dimension of the
  domain. The i-th `Tensor` has shape, `[d_i]` where `d_i` is the size of
  the grid along axis `i`. The coordinates of the grid points. Corresponds
  to the spatial grid `G` above.
* <b>`values_grid`</b>: Real `Tensor` containing the function values at time
  `start_time` which have to be stepped back to time `end_time`. The shape
  of the `Tensor` must broadcast with `[K, d_1, d_2, ..., d_n]`. The first
  axis of size `K` is the values batch dimension and allows multiple
  functions (with potentially different boundary/final conditions) to be
  stepped back simultaneously.
* <b>`one_step_fn`</b>: The transition kernel. A callable that consumes the following
  arguments by keyword:
    1. 'time': Current time
    2. 'next_time': The next time to step to. For the backwards in time
      evolution, this time will be smaller than the current time.
    3. 'coord_grid': The coordinate grid.
    4. 'values_grid': The values grid.
    5. 'quadratic_coeff': A callable returning the quadratic coefficients
      of the PDE (i.e. `(1/2)D_{ij}(t, x)` above). The callable accepts
      the time and  coordinate grid as keyword arguments and returns a
      `Tensor` with shape that broadcasts with `[dim, dim]`.
    6. 'linear_coeff': A callable returning the linear coefficients of the
      PDE (i.e. `mu_i(t, x)` above). Accepts time and coordinate grid as
      keyword arguments and returns a `Tensor` with shape that broadcasts
      with `[dim]`.
    7. 'constant_coeff': A callable returning the coefficient of the
      linear homogenous term (i.e. `r(t,x)` above). Same spec as above.
      The `one_step_fn` callable returns a 2-tuple containing the next
      coordinate grid, next values grid.
* <b>`boundary_conditions`</b>: A list of size `dim` containing boundary conditions.
  The i'th element of the list is a 2-tuple containing the lower and upper
  boundary condition for the boundary along the i`th axis.
* <b>`start_step_count`</b>: Scalar integer `Tensor`. Initial value for the number of
  time steps performed.
  Default value: 0 (i.e. no previous steps performed).
* <b>`num_steps`</b>: Positive int scalar `Tensor`. The number of time steps to take
  when moving from `start_time` to `end_time`. Either this argument or the
  `time_step` argument must be supplied (but not both). If num steps is
  `k>=1`, uniform time steps of size `(t0 - t1)/k` are taken to evolve the
  solution from `t0` to `t1`. Corresponds to the `n_steps` parameter
  above.
* <b>`time_step`</b>: The time step to take. Either this argument or the `num_steps`
  argument must be supplied (but not both). The type of this argument may
  be one of the following (in order of generality): (a) None in which case
    `num_steps` must be supplied. (b) A positive real scalar `Tensor`. The
    maximum time step to take. If the value of this argument is `dt`, then
    the total number of steps taken is N = (t1 - t0) / dt rounded up to
    the nearest integer. The first N-1 steps are of size dt and the last
    step is of size `t1 - t0 - (N-1) * dt`. (c) A callable accepting the
    current time and returning the size of the step to take. The input and
    the output are real scalar `Tensor`s.
* <b>`values_transform_fn`</b>: An optional callable applied to transform the
  solution values at each time step. The callable is invoked after the
  time step has been performed. The callable should accept the time of the
  grid, the coordinate grid and the values grid and should return the
  values grid. All input arguments to be passed by keyword.
* <b>`dtype`</b>: The dtype to use.
* <b>`name`</b>: The name to give to the ops.
  Default value: None which means `solve_forward` is used.
* <b>`**kwargs`</b>: Additional keyword args:
  (1) pde_solver_fn: Function to solve the PDE that accepts all the above
    arguments by name and returns the same tuple object as required below.
    Defaults to `tff.math.pde.fd_solvers.solve_forward`.


#### Returns:

A tuple object containing at least the following attributes:
  final_values_grid: A `Tensor` of same shape and dtype as `values_grid`.
    Contains the final state of the values grid at time `end_time`.
  final_coord_grid: A list of `Tensor`s of the same specification as
    the input `coord_grid`. Final state of the coordinate grid at time
    `end_time`.
  step_count: The total step count (i.e. the sum of the `start_step_count`
    and the number of steps performed in this call.).
  final_time: The final time at which the evolution stopped. This value
    is given by `max(min(end_time, start_time), 0)`.


<h3 id="name"><code>name</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
name()
```

The name to give to ops created by this class.


<h3 id="sample_paths"><code>sample_paths</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
sample_paths(
    times, num_samples=1, initial_state=None, random_type=None, seed=None,
    swap_memory=True, name=None, time_step=None, num_time_steps=None, skip=0,
    precompute_normal_draws=True, times_grid=None, normal_draws=None,
    watch_params=None, validate_args=False
)
```

Returns a sample of paths from the process using Euler sampling.

The default implementation uses the Euler scheme. However, for particular
types of Ito processes more efficient schemes can be used.

#### Args:


* <b>`times`</b>: Rank 1 `Tensor` of increasing positive real values. The times at
  which the path points are to be evaluated.
* <b>`num_samples`</b>: Positive scalar `int`. The number of paths to draw.
  Default value: 1.
* <b>`initial_state`</b>: `Tensor` of shape `[dim]`. The initial state of the
  process.
  Default value: None which maps to a zero initial state.
* <b>`random_type`</b>: Enum value of `RandomType`. The type of (quasi)-random number
  generator to use to generate the paths.
  Default value: None which maps to the standard pseudo-random numbers.
* <b>`seed`</b>: Seed for the random number generator. The seed is
  only relevant if `random_type` is one of
  `[STATELESS, PSEUDO, HALTON_RANDOMIZED, PSEUDO_ANTITHETIC,
    STATELESS_ANTITHETIC]`. For `PSEUDO`, `PSEUDO_ANTITHETIC` and
  `HALTON_RANDOMIZED` the seed should be an Python integer. For
  `STATELESS` and  `STATELESS_ANTITHETIC `must be supplied as an integer
  `Tensor` of shape `[2]`.
  Default value: `None` which means no seed is set.
* <b>`swap_memory`</b>: A Python bool. Whether GPU-CPU memory swap is enabled for
  this op. See an equivalent flag in `tf.while_loop` documentation for
  more details. Useful when computing a gradient of the op since
  `tf.while_loop` is used to propagate stochastic process in time.
  Default value: True.
* <b>`name`</b>: Python string. The name to give this op.
  Default value: `None` which maps to `sample_paths` is used.
* <b>`time_step`</b>: An optional scalar real `Tensor` - maximal distance between
  points in the time grid.
  Either this or `num_time_steps` should be supplied.
  Default value: `None`.
* <b>`num_time_steps`</b>: An optional Scalar integer `Tensor` - a total number of
  time steps performed by the algorithm. The maximal distance betwen
  points in grid is bounded by
  `times[-1] / (num_time_steps - times.shape[0])`.
  Either this or `time_step` should be supplied.
  Default value: `None`.
* <b>`skip`</b>: `int32` 0-d `Tensor`. The number of initial points of the Sobol or
  Halton sequence to skip. Used only when `random_type` is 'SOBOL',
  'HALTON', or 'HALTON_RANDOMIZED', otherwise ignored.
  Default value: `0`.
* <b>`precompute_normal_draws`</b>: Python bool. Indicates whether the noise
  increments in Euler scheme are precomputed upfront (see
  <a href="../../../tf_quant_finance/models/euler_sampling/sample.md"><code>models.euler_sampling.sample</code></a>). For `HALTON` and `SOBOL` random types
  the increments are always precomputed. While the resulting graph
  consumes more memory, the performance gains might be significant.
  Default value: `True`.
* <b>`times_grid`</b>: An optional rank 1 `Tensor` representing time discretization
  grid. If `times` are not on the grid, then the nearest points from the
  grid are used.
  Default value: `None`, which means that times grid is computed using
  `time_step` and `num_time_steps`.
* <b>`normal_draws`</b>: A `Tensor` of shape `[num_samples, num_time_points, dim]`
  and the same `dtype` as `times`. Represents random normal draws to
  compute increments `N(0, t_{n+1}) - N(0, t_n)`. When supplied,
  `num_sample`, `time_step` and `num_time_steps` arguments are ignored and
  the first dimensions of `normal_draws` are used instead.
* <b>`watch_params`</b>: An optional list of zero-dimensional `Tensor`s of the same
  `dtype` as `initial_state`. If provided, specifies `Tensor`s with
  respect to which the differentiation of the sampling function will
  happen. A more efficient algorithm is used when `watch_params` are
  specified. Note the the function becomes differentiable onlhy wrt to
  these `Tensor`s and the `initial_state`. The gradient wrt any other
  `Tensor` is set to be zero.
* <b>`validate_args`</b>: Python `bool`. When `True` and `normal_draws` are supplied,
  checks that `tf.shape(normal_draws)[1]` is equal to `num_time_steps`
  that is either supplied as an argument or computed from `time_step`.
  When `False` invalid dimension may silently render incorrect outputs.
  Default value: `False`.


#### Returns:

A real `Tensor` of shape `[num_samples, k, n]` where `k` is the size of the
`times`, and `n` is the dimension of the process.



#### Raises:


* <b>`ValueError`</b>:   (a) When `times_grid` is not supplied, and neither `num_time_steps` nor
    `time_step` are supplied or if both are supplied.
  (b) If `normal_draws` is supplied and `dim` is mismatched.
* <b>`tf.errors.InvalidArgumentError`</b>: If `normal_draws` is supplied and
  `num_time_steps` is mismatched.

<h3 id="volatility_fn"><code>volatility_fn</code></h3>

<a target="_blank" href="https://github.com/google/tf-quant-finance/blob/master/tf_quant_finance/models/generic_ito_process.py">View source</a>

```python
volatility_fn()
```

Python callable calculating the instantaneous volatility.

The callable should accept two real `Tensor` arguments of the same dtype and
shape `times_shape`. The first argument is the scalar time t, the second
argument is the value of Ito process X - `Tensor` of shape `batch_shape +
[dim]`. The result is value of volatility `S_ij`(t, X). The return value of
the callable is a real `Tensor` of the same dtype as the input arguments and
of shape `batch_shape + [dim, dim]`.

#### Returns:

The instantaneous volatility callable.



