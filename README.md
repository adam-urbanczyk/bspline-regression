bspline-regression
==================

<p align="center">
  <img src="https://github.com/rstebbing/bspline-regression/raw/master/figures/README-0.png" alt="B-spline regression"/>
</p>

This repository provides the code necessary to fit uniform B-splines of arbitrary degree to 2D/3D point data.
For example, the noisy red points above are approximated by a uniform cubic B-spline (blue) with 14 control points (black).

The primary purpose for making this repository public is to provide a general B-spline solver to developers which is easy to use, has minimal dependencies, but is still performant.
The secondary purpose is to provide a starting point for people learning about B-splines, sparse non-linear least squares optimisation, and the Levenberg-Marquardt algorithm in particular.

Author: Richard Stebbing

License: MIT (refer to LICENSE)

Dependencies
------------
* Numpy
* Scipy
* Sympy
* matplotlib

Getting Started
---------------

To generate the above example problem and initialisation:
```
python generate_example.py 512 1 1e-1 3 14 Example_1.json --seed=0 --frequency=3.0
```
The arguments passed to [generate_example.py](generate_example.py) here are:

Value | Argument | Description
------------- | ------------- | -------------
`512` | `num_data_points` | The number of data points to generate.
`1` | `w` | The weight applied to each squared residual.
`1e-1` | `lambda_` | The regularisation weight.
`3` | `degree` | The degree of the uniform B-spline.
`14` | `num_control_points` | The number of control points.
`Example_1.json` | `output_path` | The output path.
`--seed` | `0` | The random number generator seed (optional).
`--frequency` | `3.0` | The frequency of the `sin` (optional).

Additionally:
* `w` can be a pair (2D) or triple (3D) so that `w[i]` is the weight applied for the squared residuals in the `i`th dimensions.
This is useful for time-series data where the uncertainty of the measurement (y-axis) is much greater than that of the reported time (x-axis).
* Increasing `lambda_` increases the "force" pulling adjacent B-spline control points together.

The output `Example_1.json` is a dictionary of the form:

Key | Value Type | Description
------------- | ------------- | -------------
`degree` | `int` | The degree of the uniform B-spline.
`num_control_points` | `num_control_points` | The number of control points.
`dim` | `int` | The dimension of the problem.
`is_closed` | `bool` | `True` if the B-spline is closed, `False` otherwise.
`Y` | `array_like, shape = (num_data_points, dim)` | The array of data points.
`w` | `array_like, shape = (num_data_points, dim)` | The weight of each residual on each dimension.
`lambda_` | `float` | The regularisation weight.
`X` | `array_like, shape = (num_control_points, dim)` | The array of B-spline control points.
`u` | `array_like, shape = (num_data_points, 1)` | The array of correspondences (*preimages*).

In summary, the first four keys define the *structure* of the B-spline, the next three specify the data points and problem configuration, and the final two define the *state*: `X` is the array of control points which defines the geometry of the B-spline and `u` is the array of *preimages*.
That is, `u[i]` gives the *coordinate* of the point on the B-spline that is closest to `Y[i]`.
In [generate_example.py](generate_example.py), `X` is initialised to a straight line and `u` is set approximately.

Visualising the initialisation is straightforward:
```
python visualise.py Example_1.json --empty
```
<p align="center">
  <img src="https://github.com/rstebbing/bspline-regression/raw/master/figures/README-1.png" alt="Initialisation"/>
</p>
where correspondences are shown in orange.
(`--empty` generates a plot *without* axis labels or a title.)

To solve for `X` and `u` and visualise the output:
```
python uniform_bspline_regression.py Example_1.json Example_1_Output.json
python visualise.py Example_1_Output.json --empty
```
<p align="center">
  <img src="https://github.com/rstebbing/bspline-regression/raw/master/figures/README-2.png" alt="Solution"/>
</p>

Alternatively, to save and visualise *all* optimisation steps:
```
python uniform_bspline_regression.py Example_1.json Example_1_Output --output-all
python visualise.py Example_1_Output Example_1_Output_Visualisation
```
Additional arguments to [visualise.py](visualise.py) and [uniform_bspline_regression.py](uniform_bspline_regression.py) can be found with `--help`.

### Additional Examples
- Fitting a uniform quadratic B-spline (5 control points) weighing errors along the x-axis more heavily than the y-axis:
```
python generate_example.py 256 "1e2,1" 1e-1 2 5 Example_2.json --seed=0 --frequency=0.75 --sigma=0.1 --alpha=-0.1
python uniform_bspline_regression.py Example_2.json Example_2_Output.json
python visualise.py Example_2_Output.json --empty
```

- Fitting a uniform quintic B-spline (9 control points) to 65536 data points:
```
python generate_example.py 65356 1 1e-1 5 9 Example_3.json --seed=0 --frequency=2 --alpha=0
python uniform_bspline_regression.py Example_3.json Example_3_Output.json
python visualise.py Example_3_Output.json -d u -d X --empty
```

- Fitting a quartic B-spline (10 control points) in 3D:
```
python generate_example.py 128 1 1e-1 4 10 Example_4.json --seed=0 --dim=3
python uniform_bspline_regression.py Example_4.json Example_4_Output.json
python visualise.py Example_4_Output.json --empty
```
<p align="center">
  <img src="https://github.com/rstebbing/bspline-regression/raw/master/figures/README-5.png" alt="Quartic B-spline"/>
</p>
