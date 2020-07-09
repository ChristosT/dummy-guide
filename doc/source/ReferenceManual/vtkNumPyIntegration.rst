.. include:: ../macros.hrst
.. include:: ../abbreviations.hrst

.. _chapter:VTKNumPyIntegration:

Using NumPy for processing data
###############################


In  :numref:`chapter:PythonProgrammableFilter`, we looked at several recipes for
writing Python script for data processing that relied heavily on using NumPy for
accessing arrays and performing operations on them. In this chapter,
we take a closer look at the VTK-NumPy integration layer that makes it possible
to use VTK and NumPy together, despite significant differences in the data
representations between the two systems.

Teaser
======

Let's start with a teaser by creating a simple pipeline with  ``Sphere`` :index:`\ <Sphere>`\ 
connected to an  ``Elevation`` :index:`\ <Elevation>`\  filter, followed by the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\ .
Let's see how we would access the input data object in the  ``Script`` :index:`\ <Script>`\  for the
``Programmable Filter`` :index:`\ <Programmable Filter>`\ .

.. code-block:: python

  from paraview.vtk.numpy_interface import dataset_adapter as dsa
  from paraview.vtk.numpy_interface import algorithms as algs
  
  data = inputs[0]
  print(data.PointData.keys())
  print(data.PointData['Elevation'])

This example prints out the following in the output window.

.. code-block:: python

  ['Normals', 'Elevation']
  [ 0.67235619  0.32764378  0.72819519  0.7388373   0.70217478  0.62546903
    0.52391261  0.41762003  0.75839448  0.79325461  0.77003199  0.69332629
    0.57832992  0.44781935  0.72819519  0.7388373   0.70217478  0.62546903
    0.52391261  0.41762003  0.65528756  0.60746235  0.53835285  0.46164712
    0.39253765  0.34471244  0.58238     0.47608739  0.37453097  0.29782525
    0.2611627   0.27180481  0.55218065  0.42167011  0.30667371  0.22996798
    0.20674542  0.24160551  0.58238     0.47608739  0.37453097  0.29782525
    0.2611627   0.27180481  0.65528756  0.60746235  0.53835285  0.46164712
    0.39253765  0.34471244]

The importance lies in the last three lines. In particular, note how we used a different API to
access the  ``PointData`` :index:`\ <PointData>`\  and the  ``Elevation`` :index:`\ <Elevation>`\  array in the last two lines. Also note
that, when we printed the  ``Elevation`` :index:`\ <Elevation>`\  array, the output didn't look like one from a
``vtkDataArray``. In fact:

.. code-block:: python

  elevation = data.PointData['Elevation']
  print(type(elevation))
  
  import numpy
  print(isinstance(elevation, numpy.ndarray))

This will produce the following:

.. code-block:: python

  <class 'paraview.vtk.numpy_interface.dataset_adapter.VTKArray'>
  True

So, a VTK array is a NumPy array? What kind of trickery is this, you say? What
kind of magic makes the following possible?

.. code-block:: python

  data.PointData.append(elevation + 1, 'e plus one')
  print(algs.max(elevation))
  print(algs.max(data.PointData['e plus one']))
  print(data.VTKObject)

The output here is:

.. code-block:: python

  0.7932546138763428
  1.7932546138763428
  vtkPolyData (0x7fa20d011c60)
  ...
  Point Data:
  ...
  Number Of Arrays: 3
  Array 0 name = Normals
  Array 1 name = Elevation
  Array 2 name = e plus one

It is all in the  ``numpy_interface`` :index:`\ <numpy_interface>`\  module. It ties VTK datasets and data arrays to
NumPy arrays and introduces a number of algorithms that can work on these
objects. There is quite a bit to this module, and we will introduce it piece by
piece in the rest of this chapter.

Let's wrap up this section with one final teaser.

.. code-block:: python

  print(algs.gradient(data.PointData['Elevation']))

Output:

.. code-block:: python

  [[ 0.32640398  0.32640398  0.01982867]
   [ 0.32640402  0.32640402  0.01982871]
   ...
   [ 0.41252578  0.20134845  0.2212007 ]
   [ 0.41105482  0.21514832  0.0782456 ]]

Please note that this example is not very easily replicated by using pure NumPy.
The gradient function returns the gradient of an unstructured grid -- a concept
that does not exist in NumPy. However, the ease-of-use of NumPy is there.

Understanding the ``dataset_adapter`` module
============================================

In this section, let's take a closer look at the  ``dataset_adapter`` :index:`\ <dataset_adapter>`\  module.
This module was designed to simplify accessing VTK datasets and arrays from
Python and to provide a NumPy-style interface.

Let's continue with the example from the previous section.
Remember, this script is being put in the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\ 's  ``Script`` :index:`\ <Script>`\ ,
connected to the  ``Sphere`` :index:`\ <Sphere>`\ , followed by the  ``Elevation`` :index:`\ <Elevation>`\  filter pipeline.

.. code-block:: python

  from vtk.numpy_interface import dataset_adapter as dsa
  ...
  print(data)
  print(isinstance(data, dsa.VTKObjectWrapper))

This will print:

.. code-block:: python

  <paraview.vtk.numpy_interface.dataset_adapter.PolyData object at 0x14b7caa50>
  True

We can access the underlying VTK object using the VTKObject member:

.. code-block:: python

  print(type(data.VTKObject))

which produces:

.. code-block:: python

  <type 'vtkCommonDataModelPython.vtkPolyData'>

What we get as the $inputs$ in the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\  is actually a Python
object that wraps the VTK data object itself. The  ``Programmable Filter`` :index:`\ <Programmable Filter>`\  does
this by manually calling the  ``WrapDataObject`` :index:`\ <WrapDataObject>`\  function from the
``vtk.numpy_interface.dataset_adapter`` :index:`\ <vtk.numpy_interface.dataset_adapter>`\   module on the VTK data object.
Note that the  ``WrapDataObject`` :index:`\ <WrapDataObject>`\  function will return an appropriate wrapper class
for all ``vtkDataSet`` subclasses, ``vtkTable``, and all ``vtkCompositeData`` subclasses.
Other ``vtkDataObject`` subclasses are not currently supported.

``VTKObjectWrapper`` :index:`\ <VTKObjectWrapper>`\  forwards VTK methods to its VTKObject so the VTK API can
be accessed directy as follows:

.. code-block:: python

  print(data.GetNumberOfCells())
  96L

However,  ``VTKObjectWrapper`` :index:`\ <VTKObjectWrapper>`\ s cannot be directly passed to VTK methods as an argument.

.. code-block:: python

  from paraview.vtk.vtkFiltersGeneral import vtkShrinkPolyData
  s = vtkShrinkPolyData()
  s.SetInputData(data)

This attempt to set the data results in an error message.

.. code-block:: python

  TypeError: SetInputData argument 1: method requires a VTK object

Instead, we must pass the VTK object to the VTK filter, like so:

.. code-block:: python

  s.SetInputData(data.VTKObject)

An important thing to note in the example above is how the Python class
``vtkShrinkPolyData`` was imported for use in the script. In VTK, classes are
organized into different groups of related functionality, and these groups can be
invididually imported as Python modules. To use a class, you first identify the module in
which it resides, which can be determined from the Doxygen documentation of the class
:cite:`VTKDoxygen`. Go to the Doxygen page for the class, find the path of the file from
which the documentation was generated at the bottom of the page, which has the form
``dox/<first directory>/<second directory>/<class name>``. The module is then derived as
``vtk<first directory><second directory>``. As an example, the documentation for
``vtkShrinkPolyData`` is generated from ``dox/Filters/General/vtkShrinkPolyData``,
hence its module is ``vtkFiltersGeneral``. Then, you can import the class with a
statement of the form.

Dataset attributes
^^^^^^^^^^^^^^^^^^

So far, we have a wrapper for VTK data objects that
partially behaves like a VTK data object. This gets a little bit more
interesting when we start looking at how to access the fields (arrays)
contained within this dataset.

For simplicity, we will embed the output generated by the script in the code
itself and use the ``>>>`` prefix to differentiate the code from the
output.

.. code-block:: python

  >>> print(data.PointData)
  <vtk.numpy_interface.dataset_adapter.DataSetAttributes at 0x110f5b750>
  
  >>> print(data.PointData.keys())
  ['Normals', 'Elevation']
  
  >>> print(data.CellData.keys())
  []
  
  >>> print(data.PointData['Elevation'])
  VTKArray([ 0.5       ,  0.        ,  0.45048442,  0.3117449 ,  0.11126047,
          0.        ,  0.        ,  0.        ,  0.45048442,  0.3117449 ,
          0.11126047,  0.        ,  0.        ,  0.        ,  0.45048442,
          ...,
          0.11126047,  0.        ,  0.        ,  0.        ,  0.45048442,
          0.3117449 ,  0.11126047,  0.        ,  0.        ,  0.        ], dtype=float32)
  
  >>> elevation = data.PointData['Elevation']
  
  >>> print(elevation[:5])
  VTKArray([0.5, 0., 0.45048442, 0.3117449, 0.11126047], dtype=float32)
  # Note that this works with composite datasets as well:
  
  >>> mb = vtk.vtkMultiBlockDataSet()
  >>> mb.SetNumberOfBlocks(2)
  >>> mb.SetBlock(0, data.VTKObject)
  >>> mb.SetBlock(1, data.VTKObject)
  >>> mbw = dsa.WrapDataObject(mb)
  >>> print(mbw.PointData)
  <vtk.numpy_interface.dataset_adapter.CompositeDataSetAttributes instance at 0x11109f758>
  
  >>> print(mbw.PointData.keys())
  ['Normals', 'Elevation']
  
  >>> print(mbw.PointData['Elevation'])
  <vtk.numpy_interface.dataset_adapter.VTKCompositeDataArray at 0x1110a32d0>

It is possible to access  ``PointData`` :index:`\ <PointData>`\ ,  ``CellData`` :index:`\ <CellData>`\ ,  ``FieldData`` :index:`\ <FieldData>`\ ,
``Points`` :index:`\ <Points>`\  (subclasses of ``vtkPointSet`` only), and  ``Polygons`` :index:`\ <Polygons>`\ 
(``vtkPolyData`` only) this way. We will continue to add accessors to more
types of arrays through this API.

Working with arrays
===================

For this section, let's change our test pipeline to consist of the  ``Wavelet`` :index:`\ <Wavelet>`\ 
source connected to the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\ .

In the  ``Script`` :index:`\ <Script>`\ , we access the ``RTData`` point data array as follows:

.. code-block:: python

  # Code for 'Script'
  from paraview.vtk.vtkFiltersGeneral import vtkDataSetTriangleFilter
  image = inputs[0]
  rtdata = image.PointData['RTData']
  
  # Let's transform this data as well, using another VTK filter.
  tets = vtkDataSetTriangleFilter()
  tets.SetInputDataObject(image.VTKObject)
  tets.Update()
  
  # Here, now we need to explicitly wrap the output dataset to get a
  # VTKObjectWrapper instance.
  ugrid = dsa.WrapDataObject(tets.GetOutput())
  rtdata2 = ugrid.PointData['RTData']

Here, we created two datasets: an image data (``vtkImageData``) and an unstructured
grid (``vtkUnstructuredGrid``). They essentially represent the same data but the
unstructured grid is created by tetrahedralizing the image data. So, we expect
the unstructured grid to have the same points but more cells (tetrahedra).

.. code-block:: python

  from paraview.vtk.vtkFiltersGeneral import vtkDataSetTriangleFilter

The array API
^^^^^^^^^^^^^

``numpy_interface`` :index:`\ <numpy_interface>`\  array objects behave very similar to NumPy arrays. In
fact, arrays from ``vtkDataSet`` subclasses are instances of VTKArray, which is a
subclass of  ``numpy.ndarray`` :index:`\ <numpy.ndarray>`\ . Arrays from ``vtkCompositeDataSet`` and subclasses are
not NumPy arrays, but they behave very similarly. We will outline the differences in a
separate section. Let's start with the basics. All of the following work as
expected.

As before, for simplicity, we will embed the output generated by the script in the code
itself and use the ``>>>`` prefix to differentiate the code from the
output.

.. code-block:: python

  >>> print(rtdata[0])
  60.763466
  
  >>> print(rtdata[-1])
  57.113735
  
  >>> print(repr(rtdata[0:10:3]))
  VTKArray([  60.76346588,   95.53707886,   94.97672272,  108.49817657], dtype=float32)
  
  >>> print(repr(rtdata + 1))
  VTKArray([ 61.76346588,  86.87795258,  73.80931091, ...,  68.51051331,
          44.34006882,  58.1137352 ], dtype=float32)
  
  >>> print(repr(rtdata < 70))
  VTKArray([ True , False, False, ...,  True,  True,  True])
  
  # We will cover algorithms later. This is to generate a vector field.
  >>> avector = algs.gradient(rtdata)
  
  # To demonstrate that avector is really a vector
  >>> print(algs.shape(rtdata))
  (9261,)
  
  >>> print(algs.shape(avector))
  (9261, 3)
  
  >>> print(repr(avector[:, 0]))
  VTKArray([ 25.69367027,   6.59600449,   5.38400745, ...,  -6.58120966,
          -5.77147198,  13.19447994])
  

A few things to note in this example:

* Single component arrays always have the following shape: (n-tuples,) and
  not (n-tuples, 1)
* Multiple component arrays have the following shape: (n-tuples, n-components)
* Tensor arrays have the following shape: (n-tuples, 3, 3)
* The above holds even for images and other structured data. All arrays
  have one dimension (1 component arrays), two dimensions (multi-component arrays), or
  three dimensions (tensor arrays).

One more cool thing: It is possible to use boolean arrays to index arrays. Thus,
the following works very nicely:

.. code-block:: python

  >>> print(repr(rtdata[rtdata < 70]))
  VTKArray([ 60.76346588,  66.75043488,  69.19681549,  50.62128448,
          64.8801651 ,  57.72655106,  49.75050354,  65.05570221,
          57.38450241,  69.51113129,  64.24596405,  67.54656982,
          ...,
          61.18143463,  66.61872864,  55.39360428,  67.51051331,
          43.34006882,  57.1137352 ], dtype=float32)
  
  >>> print(repr(avector[avector[:,0] > 10]))
  VTKArray([[ 25.69367027,   9.01253319,   7.51076698],
         [ 13.1944809 ,   9.01253128,   7.51076508],
         [ 25.98717642,  -4.49800825,   7.80427408],
         ...,
         [ 12.9009738 , -16.86548471,  -7.80427504],
         [ 25.69366837,  -3.48665428,  -7.51076889],
         [ 13.19447994,  -3.48665524,  -7.51076794]])

Algorithms
^^^^^^^^^^

You can do a lot simply using the array API. However, things get much more
interesting when we start using the  ``numpy_interface.algorithms`` :index:`\ <numpy_interface.algorithms>`\  module. We
introduced it briefly in the previous examples. We will expand on it a bit more
here. For a full list of algorithms, use  ``help(algs)`` :index:`\ <help(algs)>`\ . Here are some
self-explanatory examples:

.. code-block:: python

  >>> import paraview.vtk.numpy_interface.algorithms as algs
  >>> print(repr(algs.sin(rtdata)))
  VTKArray([-0.87873501, -0.86987603, -0.52497   , ..., -0.99943125,
         -0.59898132,  0.53547275], dtype=float32)
  
  >>> print(repr(algs.min(rtdata)))
  VTKArray(37.35310363769531)
  
  >>> print(repr(algs.max(avector)))
  VTKArray(34.781060218811035)
  
  >>> print(repr(algs.max(avector, axis=0)))
  VTKArray([ 34.78106022,  29.01940918,  18.34743023])
  
  >>> print(repr(algs.max(avector, axis=1)))
  VTKArray([ 25.69367027,   9.30603981,   9.88350773, ...,  -4.35762835,
          -3.78016186,  13.19447994])

If you haven't used the axis argument before, it is pretty easy. When you don't
pass an axis value, the function is applied to all values of an array without
any consideration for dimensionality. When  ``axis=0`` :index:`\ <axis=0>`\ , the function will be applied
to each component of the array independently. When  ``axis=1`` :index:`\ <axis=1>`\ , the function will be
applied to each tuple independently. Experiment if this is not clear to you.
Functions that work this way include  ``sum`` :index:`\ <sum>`\ ,  ``min`` :index:`\ <min>`\ ,  ``max`` :index:`\ <max>`\ ,  ``std`` :index:`\ <std>`\ , and  ``var`` :index:`\ <var>`\ .

Another interesting and useful function is where the indices of an
array are returned where a particular condition occurs.

.. code-block:: python

  >>> print(repr(algs.where(rtdata < 40)))
  (array([ 420, 9240]),)
  # For vectors, this will also return the component index if an axis is not
  # defined.
  
  >>> print(repr(algs.where(avector < -29.7)))
  (VTKArray([4357, 4797, 4798, 4799, 5239]), VTKArray([1, 1, 1, 1, 1]))

So far, all of the functions that we discussed are directly provided by NumPy.
Many of the NumPy ufuncs are included in the algorithms module. They all work
with single arrays and composite data arrays.
Algorithms also provide some functions that behave somewhat differently than
their NumPy counterparts. These include cross, dot, inverse, determinant,
eigenvalue, eigenvector, etc. All of these functions are applied to each
tuple rather than to a whole array/matrix. For example:

.. code-block:: python

  >>> amatrix = algs.gradient(avector)
  >>> print(repr(algs.determinant(amatrix)))
  VTKArray([-1221.2732624 ,  -648.48272183,    -3.55133937, ...,    28.2577152 ,
          -629.28507693, -1205.81370163])

Note that everything above only leveraged per-tuple information and did not rely
on the mesh. One of VTK's biggest strengths is that its data model supports a
large variety of meshes, while its algorithms work generically on all of these mesh
types. The algorithms module exposes some of this functionality. Other functions
can be easily implemented by leveraging existing VTK filters. We used gradient
before to generate a vector and a matrix. Here it is again:

.. code-block:: python

  >>> avector = algs.gradient(rtdata)
  >>> amatrix = algs.gradient(avector)

Functions like this require access to the dataset containing the array and the
associated mesh. This is one of the reasons why we use a subclass of  ``ndarray`` :index:`\ <ndarray>`\  in
dataset_adapter:

.. code-block:: python

  >>> print(repr(rtdata.DataSet))
  <paraview.vtk.numpy_interface.dataset_adapter.DataSet at 0x11b61e9d0>

Each array points to the dataset containing it. Functions such as gradient use
the mesh and the array together. NumPy provides a gradient function too, you say.
What is so exciting about yours? Well, this:

.. code-block:: python

  >>> print(repr(algs.gradient(rtdata2)))
  VTKArray([[ 25.46767712,   8.78654003,   7.28477383],
         [  6.02292252,   8.99845123,   7.49668884],
         [  5.23528767,   9.80230141,   8.3005352 ],
         ...,
         [ -6.43249083,  -4.27642155,  -8.30053616],
         [ -5.19838905,  -3.47257614,  -7.49668884],
         [ 13.42047501,  -3.26066017,  -7.28477287]])
  >>> print(rtdata2.DataSet.GetClassName())
  vtkUnstructuredGrid

Gradient and algorithms that require access to a mesh work whether that mesh is
a uniform grid, a curvilinear grid, or an unstructured grid thanks to VTK's
data model. Take a look at various functions in the algorithms module to see all
the cool things that can be accomplished using it. In the remaining sections, we
demonstrate how specific problems can be solved using these modules.

Handling composite datasets
===========================

In this section, we take a closer look at composite datasets. For this example,
our pipeline is  ``Sphere`` :index:`\ <Sphere>`\  source, and  ``Cone`` :index:`\ <Cone>`\  source is set as two
inputs to the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\ .

We can create a multiblock dataset in the  ``Programmable Filter`` :index:`\ <Programmable Filter>`\ 's  ``Script`` :index:`\ <Script>`\ 
as follows:

.. code-block:: python

  # Let's assume inputs[0] is the output from Sphere and
  # inputs[1] is the output from Cone.
  mb = vtk.vtkMultiBlockDataSet()
  mb.SetBlock(0, inputs[0].VTKObject)
  mb.SetBlock(1, inputs[1].VTKObject)

Many of VTK's algorithms work with composite datasets without any change. For example:

.. code-block:: python

  e = vtk.vtkElevationFilter()
  e.SetInputData(mb)
  e.Update()
  
  mbe = e.GetOutputDataObject(0)
  print(mbe.GetClassName())

This will output ``vtkMultiBlockDataSet``.

Now that we have a composite dataset with a scalar, we can use  ``numpy_interface`` :index:`\ <numpy_interface>`\ .
As before, for simplicity, we will embed the output generated by the script in the code
itself and use the ``>>>`` prefix to differentiate the code from the
output.

.. code-block:: python

  >>> from paraview.vtk.numpy_interface import dataset_adapter as dsa
  >>> mbw = dsa.WrapDataObject(mbe)
  >>> print(repr(mbw.PointData.keys()))
  ['Normals', 'Elevation']
  >>> elev = mbw.PointData['Elevation']
  >>> print(repr(elev))
  <paraview.vtk.numpy_interface.dataset_adapter.VTKCompositeDataArray at 0x1189ee410>

Note that the array type is different than we have previously seen
(``VTKArray``). However, it still works the same way.

.. code-block:: python

  >>> from paraview.vtk.numpy_interface import algorithms as algs
  >>> print(algs.max(elev))
  0.5
  >>> print(algs.max(elev + 1))
  1.5

You can individually access the arrays of each block as follows.

.. code-block:: python

  >>> print(repr(elev.Arrays[0]))
  VTKArray([ 0.5       ,  0.        ,  0.45048442,  0.3117449 ,  0.11126047,
          0.        ,  0.        ,  0.        ,  0.45048442,  0.3117449 ,
          0.11126047,  0.        ,  0.        ,  0.        ,  0.45048442,
          0.3117449 ,  0.11126047,  0.        ,  0.        ,  0.        ,
          0.45048442,  0.3117449 ,  0.11126047,  0.        ,  0.        ,
          0.        ,  0.45048442,  0.3117449 ,  0.11126047,  0.        ,
          0.        ,  0.        ,  0.45048442,  0.3117449 ,  0.11126047,
          0.        ,  0.        ,  0.        ,  0.45048442,  0.3117449 ,
          0.11126047,  0.        ,  0.        ,  0.        ,  0.45048442,
          0.3117449 ,  0.11126047,  0.        ,  0.        ,  0.        ], dtype=float32)

Note that indexing is slightly different.

.. code-block:: python

  >>> print(elev[0:3])
  [VTKArray([ 0.5,  0.,  0.45048442], dtype=float32),
   VTKArray([ 0.,  0.,  0.43301269], dtype=float32)]

The return value is a composite array consisting of two VTKArrays. The  ``[]`` :index:`\ <[]>`\  operator
simply returned the first four values of each array. In general, all indexing
operations apply to each VTKArray in the composite array collection. It is similar
for algorithms, where:

.. code-block:: python

  >>> print(algs.where(elev < 0.5))
  [(array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16, 17,
         18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34,
         35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49]),),
         (array([0, 1, 2, 3, 4, 5, 6]),)]

Now, let's look at the other array called Normals.


.. code-block:: python

  >>> normals = mbw.PointData['Normals']
  >>> print(repr(normals.Arrays[0]))
  VTKArray([[  0.00000000e+00,   0.00000000e+00,   1.00000000e+00],
         [  0.00000000e+00,   0.00000000e+00,  -1.00000000e+00],
         [  4.33883727e-01,   0.00000000e+00,   9.00968850e-01],
         [  7.81831503e-01,   0.00000000e+00,   6.23489797e-01],
         [  9.74927902e-01,   0.00000000e+00,   2.22520933e-01],
         ...
         [  6.89378142e-01,  -6.89378142e-01,   2.22520933e-01],
         [  6.89378142e-01,  -6.89378142e-01,  -2.22520933e-01],
         [  5.52838326e-01,  -5.52838326e-01,  -6.23489797e-01],
         [  3.06802124e-01,  -3.06802124e-01,  -9.00968850e-01]], dtype=float32)
  >>> print(repr(normals.Arrays[1]))
  <paraview.vtk.numpy_interface.dataset_adapter.VTKNoneArray at 0x1189e7790>


Notice how the second array is a  ``VTKNoneArray`` :index:`\ <VTKNoneArray>`\ . This is because ``vtkConeSource``
does not produce normals. Where an array does not exist, we use a VTKNoneArray
as placeholder. This allows us to maintain a one-to-one mapping between datasets
of a composite dataset and the arrays in the VTKCompositeDataArray. It also
allows us to keep algorithms working in parallel without a lot of specialized
code.

Where many of the algorithms apply independently to each array in a collection,
some algorithms are global. Take  ``min`` :index:`\ <min>`\  and  ``max`` :index:`\ <max>`\ , for example, as we demonstrated above.
It is sometimes useful to get per-block answers. For this, you can use
``_per_block`` algorithms.

.. code-block:: python

  >>> print(algs.max_per_block(elev))
  [0.5, 0.4330127]

These work very nicely together with other operations. For example, here is how
we can normalize the elevation values in each block.

.. code-block:: python

  >>> _min = algs.min_per_block(elev)
  >>> _max = algs.max_per_block(elev)
  >>> _norm = (elev - _min) / (_max - _min)
  >>> print(algs.min(_norm))
  0.0
  >>> print(algs.max(_norm))
  1.0

Once you grasp these features, you should be able to use composite arrays very
similarly to single arrays.

A final note on composite datasets: The composite data wrapper provided by
``numpy_interface.dataset_adapter`` :index:`\ <numpy_interface.dataset_adapter>`\  offers a few convenience functions to traverse
composite datasets. Here is a simple example:

.. code-block:: python

  >>> for ds in mbw:
  >>>    print(type(ds))
  <class 'paraview.vtk.numpy_interface.dataset_adapter.PolyData'>
  <class 'paraview.vtk.numpy_interface.dataset_adapter.PolyData'>
