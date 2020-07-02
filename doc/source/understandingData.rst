\input{VTKDataModel}

Getting data information in ``paraview``
=====

In the visualization pipeline (Section :ref:`sec:BasicsOfVisualization`),
sources, readers, and filters are all producing data. In a VTK-based pipeline,
this data is one of the types discussed. Thus, when you create a source or open
a data file in |paraview| and hit  ``Apply`` :index:`\ <Apply>`\ , data is being produced.
The  ``Information`` :index:`\ <Information>`\  panel and the  ``Statistics Inspector`` :index:`\ <Statistics Inspector>`\  panel can be used
to inspect the characteristics of the data produced by any pipeline module.

The ``Information`` panel
^^^^^
 :index:`\ <Information Panel>`\ 

The  ``Information`` :index:`\ <Information>`\  panel provides summary information about the data produced by
the active source. By default, this panel is tucked under a tab below the
 ``Properties`` :index:`\ <Properties>`\  panel. You can toggle its visibility using
:guilabel:`View > Information`.

.. figure:: images/InformationPanel.png
    :name: fig:InformationPanel
    :width: 50%
    :align: center

    The  ``Information`` :index:`\ <Information>`\  panel in |paraview| showing data summaries for the active source.

The  ``Information`` :index:`\ <Information>`\  panel shows the data information for the active source. Thus,
similar to the  ``Properties`` :index:`\ <Properties>`\  panel, it changes when the active source is
changed (e.g., by changing the selection in the  ``Pipeline Browser`` :index:`\ <Pipeline Browser>`\ ). One way
to think of this panel is as a panel showing a summary for the data *currently*
produced by the active source. Remember that a newly-created pipeline
module does not produce any data until you hit  ``Apply`` :index:`\ <Apply>`\ . Thus, valid
information for a newly-created source will be shown on this panel only after
that  ``Apply`` :index:`\ <Apply>`\ . Similarly, if you change properties on the source and hit
 ``Apply`` :index:`\ <Apply>`\ , this panel will
reflect any changes in data characteristics. Additionally, for temporal pipelines, this panel shows
information for the current timestep alone (except as noted). Thus, as you step
through timesteps in a temporal dataset, the information displayed
will potentially change, and the panel will reflect those changes.

\begin{didyouknow}
Any text on this panel is *copy*-able. For example, if want to copy the
number of points value to use it as a property value on the  ``Properties`` :index:`\ <Properties>`\ 
panel, simply double-click on the number or click-and-drag to select the
number and use the common keyboard shortcut :kbd:`\ctrl+C` (or
:kbd:`\cmdmac+C`) to copy that value to the clipboard. Now, you can paste it
in an input widget in |paraview| or any other application, such as an
editor, by using :kbd:`\ctrl+V` (or :kbd:`\cmdmac+V`) or the application-specific
shortcut for pasting text from the clipboard. The same is true for numbers shown in
lists, such as the  ``Data Ranges`` :index:`\ <Data Ranges>`\ .
\end{didyouknow}

The panel itself is comprised of several groups of information. Groups may be
hidden based on the type of pipeline module or the type of data being produced.

The file  ``Properties`` :index:`\ <Properties>`\  group is shown for readers with information about the
file that is opened. For a temporal file series, as you step through each
time step, the file name is updated to point to the name of the file in the
series that corresponds to the current time step.

The  ``Statistics`` :index:`\ <Statistics>`\  group provides a summary of the dataset produced including its
type, its number of cells and points (or rows and columns in cases of Tabular
  datasets), and an estimate of the memory used by the dataset. This number only
includes the memory space needed to save the data arrays for the dataset. It does not include
the memory space used by the data structures themselves and, hence, must only be treated as an
estimate.

The  ``Data Arrays`` :index:`\ <Data Arrays>`\  group lists all of the available point, cells, and field arrays,
as well as their types and ranges for the current time step. The  ``Current data time`` :index:`\ <Current data time>`\ 
field shows the time value for the current timestep as a reference. As with
other places in |paraview|, the icons \icon{Images/pqCellData16.png},
\icon{Images/pqNodalData16.png}, and \icon{Images/pqGlobalData16.png} are used
to indicate cell, point, and field data arrays. Since data arrays can have multiple
components, the range for each component of the data array is shown.

 ``Bounds`` :index:`\ <Bounds>`\  shows the spatial bounds of the datasets in 3D Cartesian space. This
will be unavailable for non-geometric datasets such as tables.

For reader modules, the  ``Time`` :index:`\ <Time>`\  group shows the available time steps and corresponding
time values provided by the file.

For structured datasets such as uniform rectilinear grids or curvilinear grids,
the  ``Extents`` :index:`\ <Extents>`\  group is shown that displays the structured extents and dimensions
of the datasets.

All of the summary information discussed so far provides a synopsis of the entire dataset
produced by the pipeline module, including across all ranks (which will become
clearer once we look at using |ParaView| for parallel data processing). In cases
of composite datasets, such as mutliblock datasets or AMR datasets, recall that
these are datasets that are comprised of other datasets. In such cases, these
are summaries over all the blocks in the composite dataset. Every so often,
you will notice that the  ``Data Arrays`` :index:`\ <Data Arrays>`\  table lists an array with the suffix
 ``(partial)`` :index:`\ <(partial)>`\  (Figure :ref:`fig:InformationPanelPartialArrays`).
Such arrays are referred to as *partial arrays*. Partial arrays
is a term used to refer to arrays that are present on some non-composite
blocks or leaf nodes in a composite dataset, but not all. The  ``(partial)`` :index:`\ <(partial)>`\ 
suffix to indicate partial arrays :index:`\ <partial arrays>`\  is also used by |paraview| in other
places in the UI.

.. figure:: images/InformationPanelPartialArrays.png
    :name: fig:InformationPanelPartialArrays
    :width: 80%
    :align: center

    The  ``Data Arrays`` :index:`\ <Data Arrays>`\  section on  ``Information`` :index:`\ <Information>`\  panel showing *partial* arrays. Partial arrays are arrays that present on certain blocks in a composite dataset, but not all.

While summaries over all of the datasets in the composite dataset are useful,
you may also want to look at the data information for individual blocks.
To do so, you can use the  ``Data Hierarchy`` :index:`\ <Data Hierarchy>`\  group, which appears when
summarizing composite datasets. The  ``Data Hierarchy`` :index:`\ <Data Hierarchy>`\  widget shows the
structure or hierarchy of the composite dataset
(Figure :ref:`fig:InformationPanelDataHierarchy`). The  ``Information`` :index:`\ <Information>`\  panel
switches to showing the summaries for the selected sub-tree.
By default, the root element will be selected. You can now select any block in
the hierarchy to view the summary limited to just that sub-tree.

.. figure:: images/InformationPanelDataHierarchy.png
    :name: fig:InformationPanelDataHierarchy
    :width: 80%
    :align: center

    The  ``Data Hierarchy`` :index:`\ <Data Hierarchy>`\  section on the  ``Information`` :index:`\ <Information>`\  panel showing
    the composite data hierarchy. Selecting a particular block or subtree in this
    widget will result in the reset of the  ``Information`` :index:`\ <Information>`\  panel showing
    summaries for that block or subtree alone.

\begin{didyouknow}
Memory information shown on the  ``Information`` :index:`\ <Information>`\  panel and the  ``Statistics
Inspector`` :index:`\ <Statistics
Inspector>`\  should only be used as an approximate reference and does not translate
to how much memory the data produced by a particular pipeline module takes. This is due
to the following factors:

\begin{compactitem}
\item The size does not include the amount of memory needed to build the
data structures to store the data arrays. While, in most cases, this is negligible
compared to that of the data arrays, it can be nontrivial, especially when dealing
with deeply-nested composite datasets.
\item Several filters such as  ``Calculator`` :index:`\ <Calculator>`\  and  ``Shrink`` :index:`\ <Shrink>`\  simply pass input
data arrays through, so there's no extra space needed for those data arrays that
are shared with the input. The memory size numbers shown, however, do not take
this into consideration.
\end{compactitem}

If you need an overview of how much physical memory is being used by |ParaView| in
its current state, you can use the  ``Memory Inspector`` :index:`\ <Memory Inspector>`\ 
(Section :ref:`chapter:MemoryInspector`).
\end{didyouknow}


The ``Statistics Inspector`` panel
^^^^^
 :index:`\ <Statistics Inspector>`\ 

The  ``Information`` :index:`\ <Information>`\  panel shows data information for the active source. If you
need a quick summary of the data produced by all the pipeline modules, you can
use the  ``Statistics  Inspector`` :index:`\ <Statistics  Inspector>`\  panel. It's accessible from
:guilabel:`Views > Statistics Inspector`.

.. figure:: images/StatisticsInspector.png
    :name: fig:StatisticsInspector
    :width: 80%
    :align: center

    The  ``Statistics Inspector`` :index:`\ <Statistics Inspector>`\  panel in |paraview| showing summaries for all pipeline modules.


All of the information on this panel is also presented on the  ``Information`` :index:`\ <Information>`\ 
panel, except  ``Geometry Size`` :index:`\ <Geometry Size>`\ . This corresponds to how much memory is
needed for the transformed dataset used for rendering in the active view. For example, to
render a 3D dataset as a surface in the 3D view, |ParaView| must extract the
surface mesh as a polydata.  ``Geometry Size`` :index:`\ <Geometry Size>`\  represents the memory needed for
this polydata with the same memory-size-related caveats as with the
 ``Information`` :index:`\ <Information>`\  panel.

Getting data information in ``pvpython``
=====
.. _sec:DataInformationInPython:

When scripting with |ParaView|, you will often find yourself needing information
about the data. While |paraview| sets up filter properties and color
tables automatically using the information from the data, you
must do that explicitly when scripting.

In |pvpython|, for any pipeline module (sources, readers, or
filters), you can use the following ways to get information about the data
produced.

\begin{python}
>>>>> from paraview.simple import *
>>>>> reader = OpenDataFile(".../ParaViewData/Data/can.ex2")
>>
>># We need to update the pipeline. Otherwise, all of the data
>># information we get will be from before the file is actually
>># read and, hence, will be empty.
>>>>> UpdatePipeline()
>>
>>>>> dataInfo = reader.GetDataInformation()
>>
>># To get the number of cells or points in the dataset:
>>>>> dataInfo.GetNumberOfPoints()
>>10088
>>>>> dataInfo.GetNumberOfCells()
>>7152
>>
>># You can always nest the call, e.g.:
>>>>> reader.GetDataInformation().GetNumberOfPoints()
>>10088
>>>>> reader.GetDataInformation().GetNumberOfCells()
>>7152
>>
>># Use source.PointData or source.CellData to get information about
>># point data arrays and cell data arrays, respectively.
>>
>># Let's print the available point data arrays.
>>>>> reader.PointData[:]
>>[Array: ACCL, Array: DISPL, Array: GlobalNodeId, Array: PedigreeNodeId, Array: VEL]
>>
>># Similarly, for cell data arrays:
>>>>> reader.CellData[:]
>>[Array: EQPS, Array: GlobalElementId, Array: ObjectId, Array: PedigreeElementId]
\end{python}

 ``PointData`` :index:`\ <PointData>`\  (and  ``CellData`` :index:`\ <CellData>`\ ) is a map or dictionary where the keys are the
names of the arrays, and the values are objects that provide more information
about each of the arrays. In the rest of this section, anything we demonstrate on
 ``PointData`` :index:`\ <PointData>`\  is also applicable to  ``CellData`` :index:`\ <CellData>`\ .

\begin{python}
# Let's get the number of available point arrays.
>>> len(reader.PointData)
5

# Print the names for all available point arrays.
>>> reader.PointData.keys()
['ACCL', 'DISPL', 'GlobalNodeId', 'PedigreeNodeId', 'VEL']

>>> reader.PointData.values()
[Array: ACCL, Array: DISPL, Array: GlobalNodeId, Array: PedigreeNodeId, Array: VEL]

# To test if a particular array is present:
>>> reader.PointData.has_key("ACCL")
True

>>> reader.PointData.has_key("--non-existent-array--")
False

\end{python}

From  ``PointData`` :index:`\ <PointData>`\  (or  ``CellData`` :index:`\ <CellData>`\ ), you can get access to an object
that provides information for each of the arrays. This object gives us
methods to get data ranges, component counts, tuple counts, etc.

\begin{python}
# Let's get information about 'ACCL' array.
>>> arrayInfo = reader.PointData["ACCL"]
>>> arrayInfo.GetName()
'ACCL'

# To get the number of components in each tuple and the number
# of tuples in the data array:
>>> arrayInfo.GetNumberOfTuples()
10088
>>> arrayInfo.GetNumberOfComponents()
3

# Alternative way for doing the same:
>>> reader.PointData["ACCL"].GetNumberOfTuples()
10088
>>> reader.PointData["ACCL"].GetNumberOfComponents()
3

# To get the range for a particular component, e.g. component 0:
>>> reader.PointData["ACCL"].GetRange(0)
(-4.965284006175352e-07, 3.212448973499704e-07)

# To get the range for the magnitude in cases of multi-component arrays
# use -1 as the component number.
>>> reader.PointData["ACCL"].GetRange(-1)
(0.0, 1.3329898584157294e-05)

# To determine the data data type for this array:
>>> from paraview import vtk
>>> reader.PointData["ACCL"].GetDataType() == vtk.VTK_DOUBLE
True
# The paraview.vtk module provides access to these constants such as
# VTK_DOUBLE, VTK_FLOAT, VTK_INT, etc.

# Likewise, to test the dataset type, itself:
>>> reader.GetDataInformation().GetDataSetType() == \
                               vtk.VTK_MULTIBLOCK_DATA_SET
True
\end{python}

Here's a sample script to iterate over all point data arrays and print their
magnitude ranges:

\begin{python}
>>> def print_point_data_ranges(source):
...    """Prints array ranges for all point arrays"""
...    for arrayInfo in source.PointData:
...        # get the array's name
...        name = arrayInfo.GetName()
...        # get magnitude range
...        range = arrayInfo.GetRange(-1)
...        print "%s = [%.3f, %.3f]" % (name, range[0], range[1])

# Let's call this function on our reader.
>>> print_point_data_ranges(reader)
ACCL = [0.000, 0.000]
DISPL = [0.000, 0.000]
GlobalNodeId = [1.000, 10088.000]
PedigreeNodeId = [1.000, 10088.000]
VEL = [0.000, 5000.000]
\end{python}

\begin{didyouknow}
The example scripts in this section all demonstrated how to obtain information about the data such
as the number of points and cells, data bounds, and array ranges. However, what they
do not show is how to access the raw data itself. To see how to obtain the full data,
please see Section :ref:`sec:Fetch`.
\end{didyouknow}

\begin{TODO}
\begin{itemize}
\item Explain what is involved when collecting data information so user
understand the costs involved. Then we could add a didyouknow blurb about how
that differs in symmetric batch -- or something.
\end{itemize}
\end{TODO}
