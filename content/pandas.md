Title: Speeding up rolling pandas
Date: 2018-11-16 
Category: Python
Tags: python, pandas
Authors: John Erik Sloper
Summary: Using numpy to speed up pandas rolling

Pandas is an exceedingly useful package for data analysis in python and is in general very performant. However there are some cases where improving performance can be of importance.

Below we look at using numpy to create a faster version of rolling windows.

Consider the following snippet.

	:::python
	import pandas as pd
	import numpy as np
	s = pd.Series(range(10**6))
	s.rolling(window=2).mean()

The rolling call will create windows of size 2 and then we calculate the mean of each:

	:::
	0              NaN
	1              0.5
	2              1.5
				...
	999998    999997.5
	999999    999998.5
	Length: 1000000, dtype: float64

However using stride_tricks in numpy we can create a function which iterates the values faster:

	:::python
	def rolling_window(a, window):
		shape = a.shape[:-1] + (a.shape[-1] - window + 1, window)
		strides = a.strides + (a.strides[-1],)
		return np.lib.stride_tricks.as_strided(a, shape=shape, strides=strides)

*Note*: There is a version of this function in scikit-image:

`from skimage.util.shape import view_as_windows`

We can use the functions as follows:
`np.mean(rolling_window(s,2), axis=1)`

This will return the same data as we calculated using the rolling() method from pandas (without the leading nan value)

### Measuring Performance

Using the %timeit tool (conveniently built into Ipython and therefore jupyter as well) we measure the performance of the two versions:

	:::python
	s = pd.Series(np.random.randint(10, size=10**6))
	%timeit s.rolling(window=2).mean()
	%timeit np.mean(rolling_window(s, 2), axis=1)

output:

	:::
	58.6 ms ± 1.42 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
	25.1 ms ± 1.24 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)

The numpy version is approximately twice as fast. For other sizes of arrays the performance will vary between 2-5x faster.

Let's check again, but with a different calculation:

	:::python
	s = pd.Series(np.random.randint(10, size=10**6))
	%timeit s.rolling(window=2).sum()
	%timeit np.sum(rolling_window(s, 2), axis=1)</pre>

output:

	:::
	52.5 ms ± 1.73 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
	14.9 ms ± 129 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)</pre>

That's it. We sacrifice a bit of readability for a significant speed up.

### Note
There are numerous ways to calculate means faster than the version above. If you are really looking into performance see the notebook in this gist: <a href="https://gist.github.com/johnsloper/e6783949bb4bc789a03a32e435593182">rolling.ipynb</a>